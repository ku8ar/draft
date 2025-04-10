// scripts/build-remote.js
import { bundle } from '@callstack/repack';
import { fileURLToPath } from 'node:url';
import path from 'node:path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

bundle({
  platform: 'ios',
  mode: 'production',
  entry: path.resolve(__dirname, '../index.js'),
  output: path.resolve(__dirname, '../dist/ios'),
}).catch(err => {
  console.error('[Repack] Build failed:', err);
  process.exit(1);
});
