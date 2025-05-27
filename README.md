const fs = require('fs');
const path = require('path');

const ROOT = path.resolve(__dirname, '..');
const PKG_PATH = path.join(ROOT, 'package.json');
const NODE_MODULES = path.join(ROOT, 'node_modules');

const pkg = JSON.parse(fs.readFileSync(PKG_PATH, 'utf-8'));
const dependencies = Object.keys(pkg.dependencies || {});
const appName = pkg.name;

function updateGradleInDependency(dep) {
  const androidDir = path.join(NODE_MODULES, dep, 'android');
  const gradleFile = path.join(androidDir, 'settings.gradle');
  const localProps = path.join(androidDir, 'local.properties');

  if (!fs.existsSync(gradleFile)) return;

  let lines = fs.readFileSync(gradleFile, 'utf-8').split('\n');

  // Dodaj rootProject.name jeśli nie istnieje
  const hasProjectName = lines.some(line => line.includes('rootProject.name'));
  if (!hasProjectName) {
    lines.unshift(`rootProject.name = '${appName}'`);
  }

  // Usuń linie zaczynające się od org.gradle.jvmargs
  lines = lines.filter(line => !line.trim().startsWith('org.gradle.jvmargs'));

  fs.writeFileSync(gradleFile, lines.join('\n'), 'utf-8');
  console.log(`✔ Zaktualizowano: ${dep}/android/settings.gradle`);

  // Usuń local.properties jeśli istnieje
  if (fs.existsSync(localProps)) {
    fs.unlinkSync(localProps);
    console.log(`🗑 Usunięto: ${dep}/android/local.properties`);
  }
}

dependencies.forEach(updateGradleInDependency);
