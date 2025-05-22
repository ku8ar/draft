type Headers = Record<string, string>

export class MonkeyXMLHttpRequest {
  private method = 'GET'
  private url = ''
  private headers: Headers = {}
  private body: any = null
  private controller: AbortController | null = null

  readyState = 0
  status = 0
  responseText = ''
  onreadystatechange: (() => void) | null = null
  onerror: (() => void) | null = null
  onabort: (() => void) | null = null

  open(method: string, url: string) {
    this.method = method.toUpperCase()
    this.url = url
    this.readyState = 1
    this._emitChange()
  }

  setRequestHeader(key: string, value: string) {
    this.headers[key] = value
  }

  send(body?: any) {
    this.body = body
    this.controller = new AbortController()
    this.readyState = 2
    this._emitChange()

    // użyj Twojego fetcha — np. DBSFetch.fetch()
    DBSFetch.fetch(this.url, {
      method: this.method,
      headers: this.headers,
      body: this.body,
      signal: this.controller.signal
    })
      .then((res) => {
        this.status = res.status
        return res.text()
      })
      .then((text) => {
        this.responseText = text
        this.readyState = 4
        this._emitChange()
      })
      .catch((e) => {
        if (e.name === 'AbortError') {
          this.readyState = 0
          this.onabort?.()
        } else {
          this.readyState = 4
          this.onerror?.()
        }
        this._emitChange()
      })
  }

  abort() {
    this.controller?.abort()
  }

  // ⬇️ prywatna pomocnicza metoda do odpalania zdarzenia
  private _emitChange() {
    this.onreadystatechange?.call(this)
  }
}
