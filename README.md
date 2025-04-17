remotes: {
  ChildApp: `promise new Promise(async function(resolve, reject) {
    try {
      const res = await fetch("http://localhost:8000/index.bundle");
      const jsCode = await res.text();

      const remoteInit = new Function(jsCode);

      await __webpack_init_sharing__('default'); // 🔑 kluczowy krok

      remoteInit(); // teraz wykonaj remote

      await globalThis.ChildApp.init(__webpack_share_scopes__.default);

      resolve({
        get: globalThis.ChildApp.get,
        init: globalThis.ChildApp.init,
      });
    } catch (err) {
      console.error("Failed to load ChildApp:", err);
      reject(err);
    }
  })`
}
