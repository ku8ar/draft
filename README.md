class ProxyHandler(http.server.BaseHTTPRequestHandler):
    def do_GET(self):
        self.handle_method('GET')

    def do_HEAD(self):
        self.handle_method('HEAD')

    def handle_method(self, method):
        path = self.path.lstrip('/')

        for base_url in REPOSITORIES:
            full_url = base_url + path
            req = urllib.request.Request(full_url, method=method)

            try:
                with opener.open(req) as res:
                    self.send_response(res.getcode())
                    for key, value in res.getheaders():
                        self.send_header(key, value)
                    self.end_headers()
                    if method == 'GET':
                        self.wfile.write(res.read())
                    print(f"[{res.getcode()}] {method} {full_url}")
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
