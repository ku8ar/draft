import http.server
import urllib.request
import urllib.error
import socketserver
from pathlib import Path
from datetime import datetime, timezone
import email.utils  # for RFC 2822 date parsing/formatting

PORT = 9292
SCRIPT_DIR = Path(__file__).resolve().parent
CACHE_DIR = SCRIPT_DIR / ".maven-proxy-cache"

REPOSITORIES = [
    'https://repo1.maven.org/maven2/',
    'https://backup.repo.local/repo/'
]

proxy_handler = urllib.request.ProxyHandler()
opener = urllib.request.build_opener(proxy_handler)

class ProxyHandler(http.server.BaseHTTPRequestHandler):
    def do_GET(self):
        self.handle_method('GET')

    def do_HEAD(self):
        self.handle_method('HEAD')

    def handle_method(self, method):
        path = self.path.lstrip('/')
        cache_file = CACHE_DIR / Path(path)
        headers_file = cache_file.with_suffix(cache_file.suffix + ".headers")

        # --- Check cache ---
        if cache_file.exists():
            # Handle If-Modified-Since for HEAD or GET
            if_modified_since = self.headers.get("If-Modified-Since")
            if if_modified_since:
                file_mtime = datetime.fromtimestamp(cache_file.stat().st_mtime, tz=timezone.utc)
                ims_dt = email.utils.parsedate_to_datetime(if_modified_since)

                if file_mtime <= ims_dt:
                    self.send_response(304)
                    self.end_headers()
                    print(f"[CACHE] {method} 304 {cache_file}")
                    return

            # Serve cached file
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
                print(f"[CACHE] GET {cache_file}")
            else:
                print(f"[CACHE] HEAD {cache_file}")
            return

        # --- Try remote repos ---
        for base_url in REPOSITORIES:
            full_url = base_url + path
            req = urllib.request.Request(full_url, method=method)

            try:
                with opener.open(req) as res:
                    status = res.getcode()
                    self.send_response(status)

                    headers = []
                    for key, value in res.getheaders():
                        self.send_header(key, value)
                        headers.append(f"{key}: {value}")
                    self.end_headers()

                    if method == 'GET':
                        body = res.read()
                        self.wfile.write(body)

                        # Cache body
                        cache_file.parent.mkdir(parents=True, exist_ok=True)
                        with cache_file.open('wb') as f:
                            f.write(body)

                        # Cache headers (content-type etc.)
                        with headers_file.open('w') as f:
                            f.write('\n'.join(headers))

                        # Set file modification time from Date header
                        date_header = res.headers.get("Date")
                        if date_header:
                            try:
                                dt = email.utils.parsedate_to_datetime(date_header)
                                ts = dt.timestamp()
                                os.utime(cache_file, (ts, ts))
                            except Exception:
                                pass

                        print(f"[{status}] Cached {full_url} -> {cache_file}")
                    else:
                        print(f"[{status}] HEAD {full_url}")
                    return
            except urllib.error.HTTPError as e:
                print(f"[{e.code}] {method} {full_url}")
                if e.code in (401, 403, 404):
                    continue
                else:
                    break
            except Exception as e:
                print(f"[ERR] {method} {full_url}: {e}")
                continue

        self.send_response(404)
        self.end_headers()
        if method == 'GET':
            self.wfile.write(b'Artifact not found in any repository.')
        print(f"[404] Not found: {method} {self.path}")

if __name__ == '__main__':
    with socketserver.ThreadingTCPServer(("", PORT), ProxyHandler) as httpd:
        print(f"Maven proxy with cache, ETag-like HEAD, and content-type headers at http://localhost:{PORT}/")
        print(f"Cache directory: {CACHE_DIR}")
        httpd.serve_forever()
