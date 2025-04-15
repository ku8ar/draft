import http.server
import urllib.request
import urllib.error
import socketserver
from pathlib import Path

PORT = 9292
SCRIPT_DIR = Path(__file__).resolve().parent
CACHE_DIR = SCRIPT_DIR / ".maven-proxy-cache"

REPOSITORIES = [
    'https://repo1.maven.org/maven2/',
    'https://backup.repo.local/repo/'
]

# Use system proxy if available
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

        # Serve from cache if available
        if cache_file.exists():
            self.send_response(200)
            self.send_header("Content-Type", "application/octet-stream")
            self.send_header("Content-Length", str(cache_file.stat().st_size))
            self.end_headers()

            if method == 'GET':
                with cache_file.open('rb') as f:
                    self.wfile.write(f.read())
                print(f"[CACHE] GET {cache_file}")
            else:
                print(f"[CACHE] HEAD {cache_file}")
            return

        # Try downloading from remote repositories
        for base_url in REPOSITORIES:
            full_url = base_url + path
            req = urllib.request.Request(full_url, method=method)

            try:
                with opener.open(req) as res:
                    status = res.getcode()
                    self.send_response(status)
                    for key, value in res.getheaders():
                        self.send_header(key, value)
                    self.end_headers()

                    if method == 'GET':
                        body = res.read()
                        self.wfile.write(body)

                        # Save to cache
                        cache_file.parent.mkdir(parents=True, exist_ok=True)
                        with cache_file.open('wb') as f:
                            f.write(body)
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
        print(f"Maven proxy with disk cache running at http://localhost:{PORT}/")
        print(f"Cache directory: {CACHE_DIR}")
        httpd.serve_forever()
