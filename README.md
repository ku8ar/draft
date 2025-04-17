remotes: {
  SomeApp: `promise new Promise(async (resolve, reject) => {
    try {
      const bundleUrl = "http://localhost:20113/index.bundle";

      const response = await fetch(bundleUrl);
      const jsCode = await response.text();

      // wykonujemy kod remote (rejestruje globalThis.SomeApp)
      new Function(jsCode)();

      if (!globalThis.SomeApp) {
        throw new Error("SomeApp was not registered on globalThis");
      }

      // budujemy proxy dla Module Federation
      resolve({
        get: globalThis.SomeApp.get,
        init: globalThis.SomeApp.init,
      });
    } catch (err) {
      console.error("Failed to load remote SomeApp:", err);
      reject(err);
    }
  })`
}
