remotes: {
  SomeApp: `promise new Promise(function(resolve, reject) {
    fetch("http://localhost:20113/index.bundle")
      .then(function(res) { return res.text(); })
      .then(function(jsCode) {
        new Function(jsCode)();
        resolve({
          get: globalThis.SomeApp.get,
          init: globalThis.SomeApp.init,
        });
      })
      .catch(reject);
  })`
}
