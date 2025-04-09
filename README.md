import { AppRegistry, Text } from 'react-native';
import React from 'react';

// 1. Federation manifest
global.__REMOTE_FEDERATION_MANIFEST__ = {
  remotes: {
    app1: {
      url: 'http://localhost:9001',
    },
  },
};

// 2. createURLResolver
global.createURLResolver = function () {
  return (remoteName, modulePath) => {
    const base = global.__REMOTE_FEDERATION_MANIFEST__?.remotes?.[remoteName]?.url;
    if (!base) throw new Error(`Remote "${remoteName}" not found in manifest`);
    return `${base}/${modulePath}`;
  };
};

// 3. Minimal RN component bez JSX
const App = () =>
  React.createElement(
    Text,
    {
      style: {
        fontSize: 24,
        marginTop: 100,
        textAlign: 'center',
      },
    },
    'test'
  );

// 4. Federation expose
global.__federation_exposes__ = {
  './App': App,
};

// 5. Repack runtime mocks
global.__webpack_require__ = global.__webpack_require__ || function () {};
global.__federation_shared__ = global.__federation_shared__ || {};

// 6. Optional for native test rendering
AppRegistry.registerComponent('RemoteApp', () => App);

console.log('✅ container.bundle loaded');
