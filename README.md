      remotes: {
        ChildApp: `promise new Promise(function(resolve, reject) {
          fetch("http://localhost:8000/remoteEntry.bundle")
            .then(function(res) { return res.text(); })
            .then(function(code) {
              const remoteInit = new Function(code);
              remoteInit();

              // Nie używamy __webpack_init_sharing__ w RN — to API z Webpack Web
              if (!globalThis.ChildApp) {
                throw new Error("ChildApp global not found");
              }

              // Niektóre bundlery wymagają initowania ręcznie
              if (globalThis.ChildApp.init) {
                globalThis.ChildApp.init(global.__webpack_share_scopes__.default || {});
              }

              resolve({
                get: globalThis.ChildApp.get,
                init: globalThis.ChildApp.init,
              });
            })
            .catch(reject);
        })`
      },
