class FetchXHR {
  readyState = 0
  status = 0
  responseText = ''
  responseURL = ''
  onreadystatechange: (() => void) | null = null

  private method = 'GET'
  private url = ''
  private headers: Record<string, string> = {}
  private body: any = null

  open(method: string, url: string) {
    this.method = method
    this.url = url
    this.readyState = 1
    this.trigger()
  }

  setRequestHeader(header: string, value: string) {
    this.headers[header] = value
  }

  send(body?: any) {
    this.body = body

    fetch(this.url, {
      method: this.method,
      headers: this.headers,
      body: body
    })
      .then((res) => {
        this.status = res.status
        this.responseURL = res.url
        this.readyState = 4
        return res.text()
      })
      .then((text) => {
        this.responseText = text
        this.trigger()
      })
      .catch(() => {
        this.status = 0
        this.readyState = 4
        this.trigger()
      })
  }

  private trigger() {
    this.onreadystatechange?.()
  }
}
