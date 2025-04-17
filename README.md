remotes: {
  SomeApp: `promise new Promise(function(resolve, reject) {
    fetch("http://localhost:20113/index.bundle")
      .then(function(res) { return res.text(); })
      .then(function(jsCode) {
        new Function(jsCode)();

        if (!globalThis.SomeApp) {
          throw new Error("globalThis.SomeApp is not defined after eval");
        }

        globalThis.SomeApp.init(__webpack_share_scopes__.default).then(() => {
          resolve({
            get: globalThis.SomeApp.get,
            init: globalThis.SomeApp.init,
          });
        }).catch(reject);
      })
      .catch(reject);
  })`
}
