export class MinimalEventTarget {
  private listeners: Record<string, ((event: any) => void)[]> = {}

  addEventListener(type: string, callback: (event?: any) => void) {
    if (!this.listeners[type]) this.listeners[type] = []
    this.listeners[type].push(callback)
  }

  removeEventListener(type: string, callback: (event?: any) => void) {
    this.listeners[type] = (this.listeners[type] || []).filter(fn => fn !== callback)
  }

  dispatchEvent(type: string, event: any = {}) {
    this.listeners[type]?.forEach(fn => fn.call(this, event))
  }
}
