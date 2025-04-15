import http.server
import urllib.request
import urllib.error
import socketserver

PORT = 9292

REPOSITORIES = [
    'https://repo1.maven.org/maven2/',
    'https://backup.repo.local/repo/',
    'https://another.repo.org/artifactory/'
]

# Use system proxy (macOS compatible)
proxy_handler = urllib.request.ProxyHandler()
opener = urllib.request.build_opener(proxy_handler)

class ProxyHandler(http.server.BaseHTTPRequestHandler):
    def do_GET(self):
        path = self.path.lstrip('/')

        for base_url in REPOSITORIES:
            full_url = base_url + path
            req = urllib.request.Request(full_url)

            try:
                with opener.open(req) as res:
                    self.send_response(res.getcode())
                    for key, value in res.getheaders():
                        self.send_header(key, value)
                    self.end_headers()
                    self.wfile.write(res.read())
                    print(f"[{res.getcode()}] {full_url}")
                    return
            except urllib.error.HTTPError as e:
                print(f"[{e.code}] {full_url}")
                if e.code in (401, 403, 404):
                    continue
                else:
                    break
            except Exception as e:
                print(f"[ERR] {full_url}: {e}")
                continue

        self.send_response(404)
        self.end_headers()
        self.wfile.write(b'Artifact not found in any repository.')
        print(f"[404] Not found: {self.path}")

if __name__ == '__main__':
    with socketserver.ThreadingTCPServer(("", PORT), ProxyHandler) as httpd:
        print(f"Maven proxy server running at http://localhost:{PORT}/")
        httpd.serve_forever()
