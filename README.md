remotes: {
  SomeApp: `promise new Promise(async (resolve, reject) => {
    try {
      const res = await fetch("http://localhost:20113/index.bundle");
      const jsCode = await res.text();
      new Function(jsCode)();
      resolve({
        get: globalThis.SomeApp.get,
        init: globalThis.SomeApp.init,
      });
    } catch (err) {
      reject(err);
    }
  })`
}
