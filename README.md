const fs = require("fs");
const path = require("path");
const os = require("os");

const inputPath = path.resolve(__dirname, "../android/build.gradle");
const outputPath = path.join(os.homedir(), ".gradle/init.d/ext-init.gradle");

if (!fs.existsSync(inputPath)) {
  console.error("❌ Nie znaleziono pliku android/build.gradle");
  process.exit(1);
}

const file = fs.readFileSync(inputPath, "utf8");

// Znajdź blok ext { ... } wewnątrz buildscript { ... }
const extMatch = file.match(/buildscript\s*{[\s\S]*?ext\s*{([\s\S]*?)^\s*}\s*}/m);
if (!extMatch) {
  console.warn("⚠️  Nie znaleziono bloku buildscript.ext");
  process.exit(1);
}

const extBlock = extMatch[1];
const pairs = [...extBlock.matchAll(/^\s*(\w+)\s*=\s*(.+?)\s*$/gm)];

if (pairs.length === 0) {
  console.warn("⚠️  Nie znaleziono żadnych zmiennych w ext");
  process.exit(1);
}

const output = [
  "// AUTOMATYCZNIE WYGENEROWANY PLIK — NIE EDYTUJ RĘCZNIE",
  "allprojects {"
];

for (const [, key, value] of pairs) {
  output.push(`  ext.${key} = ${value}`);
}

output.push("}");

fs.mkdirSync(path.dirname(outputPath), { recursive: true });
fs.writeFileSync(outputPath, output.join("\n") + "\n");

console.log(`✅ Wygenerowano: ${outputPath}`);
