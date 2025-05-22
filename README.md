private _listeners: Record<string, Function[]> = {}

addEventListener(type: string, callback: (...args: any[]) => void) {
  if (!this._listeners[type]) {
    this._listeners[type] = []
  }
  this._listeners[type].push(callback)
}

removeEventListener(type: string, callback: (...args: any[]) => void) {
  if (!this._listeners[type]) return
  this._listeners[type] = this._listeners[type].filter(fn => fn !== callback)
}

dispatchEvent(type: string, event: any = {}) {
  this._listeners[type]?.forEach(fn => fn.call(this, event))
}
