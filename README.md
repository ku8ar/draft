import http.server
import urllib.request
import urllib.error
import socketserver
from pathlib import Path
from datetime import datetime, timezone
import email.utils
import os

PORT = 9292
SCRIPT_DIR = Path(__file__).resolve().parent
CACHE_DIR = SCRIPT_DIR / ".maven-proxy-cache"

REPOSITORIES = [
    'https://maven.google.com/',
    'https://repo1.maven.org/maven2/',
    'https://plugins.gradle.org/m2/'
]

proxy_handler = urllib.request.ProxyHandler()
opener = urllib.request.build_opener(proxy_handler)

class ProxyHandler(http.server.BaseHTTPRequestHandler):
    def do_GET(self):
        self.handle_method('GET')

    def do_HEAD(self):
        self.handle_method('HEAD')

    def try_fetch(self, path, method):
        cache_file = CACHE_DIR / Path(path)
        headers_file = cache_file.with_suffix(cache_file.suffix + ".headers")

        # Serve from cache
        if cache_file.exists():
            if_modified_since = self.headers.get("If-Modified-Since")
            if if_modified_since:
                file_mtime = datetime.fromtimestamp(cache_file.stat().st_mtime, tz=timezone.utc)
                ims_dt = email.utils.parsedate_to_datetime(if_modified_since)
                if file_mtime <= ims_dt:
                    self.send_response(304)
                    self.end_headers()
                    return True

            self.send_response(200)
            content_type = "application/octet-stream"
            if headers_file.exists():
                for line in headers_file.read_text().splitlines():
                    key, _, value = line.partition(": ")
                    if key.lower() == "content-type":
                        content_type = value.strip()
            self.send_header("Content-Type", content_type)
            self.send_header("Content-Length", str(cache_file.stat().st_size))
            self.send_header("Last-Modified", self.date_time_string(cache_file.stat().st_mtime))
            self.end_headers()

            if method == 'GET':
                with cache_file.open('rb') as f:
                    self.wfile.write(f.read())
            print(f"[CACHE] {method} {path}")
            return True

        # Try all repositories
        for base_url in REPOSITORIES:
            full_url = base_url + path
            req = urllib.request.Request(full_url, method=method)
            try:
                with opener.open(req) as res:
                    status = res.getcode()
                    self.send_response(status)
                    headers = res.getheaders()
                    for key, value in headers:
                        self.send_header(key, value)
                    self.end_headers()

                    if method == 'GET':
                        body = res.read()
                        self.wfile.write(body)

                        cache_file.parent.mkdir(parents=True, exist_ok=True)
                        with cache_file.open('wb') as f:
                            f.write(body)
                        with headers_file.open('w') as f:
                            f.write('\n'.join(f"{k}: {v}" for k, v in headers))

                        date_header = res.headers.get("Date")
                        if date_header:
                            try:
                                dt = email.utils.parsedate_to_datetime(date_header)
                                ts = dt.timestamp()
                                os.utime(cache_file, (ts, ts))
                            except Exception:
                                pass

                    print(f"[{status}] {method} {full_url}")
                    return True
            except urllib.error.HTTPError as e:
                print(f"[{e.code}] {method} {full_url}")
                if 400 <= e.code < 600:
                    continue
            except Exception as e:
                print(f"[ERR] {method} {full_url}: {e}")
                continue
        return False

    def try_plugin_marker_fallback(self, path, method):
        parts = path.strip("/").split("/")
        if len(parts) < 4 or parts[-1].endswith(".sha1"):
            return False
        group_parts = parts[:-3]
        artifact = parts[-3]
        version = parts[-2]
        filename = parts[-1]
        group_id = ".".join(group_parts)
        marker_artifact = f"{group_id}.{artifact}.gradle.plugin"
        new_parts = group_parts + [artifact, marker_artifact, version, f"{marker_artifact}-{version}" + filename[filename.find("."):]]
        new_path = "/".join(new_parts)
        print(f"[FALLBACK] Trying plugin marker path: {new_path}")
        return self.try_fetch(new_path, method)

    def handle_method(self, method):
        path = self.path.lstrip('/')
        if self.try_fetch(path, method):
            return
        if self.try_plugin_marker_fallback(path, method):
            return
        self.send_response(404)
        self.end_headers()
        if method == 'GET':
            self.wfile.write(b'Artifact not found in any repository.')
        print(f"[404] {method} {self.path}")

if __name__ == '__main__':
    with socketserver.ThreadingTCPServer(("", PORT), ProxyHandler) as httpd:
        print(f"Maven proxy with disk cache and plugin marker fallback running at http://localhost:{PORT}/")
        print(f"Cache directory: {CACHE_DIR}")
        httpd.serve_forever()
