import { DBSFetch } from './DBSFetch' // <-- zaimportuj swój moduł

type Headers = Record<string, string>

export class MonkeyXMLHttpRequest {
  private method = 'GET'
  private url = ''
  private headers: Headers = {}
  private body: any = null
  private aborted = false

  private _listeners: Record<string, Function[]> = {}

  readyState = 0
  status = 0
  responseText = ''

  onreadystatechange: (() => void) | null = null
  onerror: (() => void) | null = null
  onabort: (() => void) | null = null
  onload: (() => void) | null = null
  onloadend: (() => void) | null = null

  open(method: string, url: string) {
    this.method = method.toUpperCase()
    this.url = url
    this.readyState = 1
    this._emit('readystatechange')
  }

  setRequestHeader(key: string, value: string) {
    this.headers[key] = value
  }

  send(body?: any) {
    this.body = body
    this.readyState = 2
    this._emit('readystatechange')
    this._emit('loadstart')

    DBSFetch.fetch(this.url, {
      method: this.method,
      headers: this.headers,
      body: this.body
    })
      .then((res) => {
        if (this.aborted) return

        this.status = res.status
        this.responseText = res.body
        this.readyState = 4
        this._emit('readystatechange')
        this._emit('load')
        this._emit('loadend')
      })
      .catch((e) => {
        if (this.aborted) return
        this.readyState = 4
        this._emit('error')
        this._emit('loadend')
      })
  }

  abort() {
    this.aborted = true
    this.readyState = 0
    this._emit('abort')
    this._emit('loadend')
  }

  addEventListener(type: string, callback: (...args: any[]) => void) {
    if (!this._listeners[type]) this._listeners[type] = []
    this._listeners[type].push(callback)
  }

  removeEventListener(type: string, callback: (...args: any[]) => void) {
    this._listeners[type] = (this._listeners[type] || []).filter(fn => fn !== callback)
  }

  dispatchEvent(type: string, event: any = {}) {
    this._emit(type, event)
  }

  private _emit(type: string, event: any = {}) {
    const handler = (this as any)[`on${type}`]
    if (typeof handler === 'function') {
      handler.call(this, event)
    }
    this._listeners[type]?.forEach(fn => fn.call(this, event))
  }
}
