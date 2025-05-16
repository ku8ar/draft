#!/usr/bin/env node

const fs = require('fs');
const path = require('path');

const FIND_DIR = path.resolve(__dirname, '../node_modules');
const REPO_URL = 'https://my.rpoxy';
const REPO_USERNAME_VAR = 'MY_REPO_USERNAME';
const REPO_PASSWORD_VAR = 'MY_REPO_PASSWORD';

const TEMPLATES = {
    groovy: `maven {
    url "${REPO_URL}"
    credentials {
        username = System.getenv("${REPO_USERNAME_VAR}") ?: "defaultUser"
        password = System.getenv("${REPO_PASSWORD_VAR}") ?: "defaultPass"
    }
}`,
    kts: `maven {
    url = uri("${REPO_URL}")
    credentials {
        username = System.getenv("${REPO_USERNAME_VAR}") ?: "defaultUser"
        password = System.getenv("${REPO_PASSWORD_VAR}") ?: "defaultPass"
    }
}`
};

function processFile(filePath) {
    const isKts = filePath.endsWith('.kts');
    const content = fs.readFileSync(filePath, 'utf8');
    const replaced = content
        .replace(/^[ \t]*google\(\)[ \t]*\n?/gm, '')
        .replace(/^[ \t]*mavenCentral\(\)[ \t]*\n?/gm, (isKts ? TEMPLATES.kts : TEMPLATES.groovy) + '\n');
    fs.writeFileSync(filePath, replaced, 'utf8');
    console.log(`✔ Processed ${filePath}`);
}

function main() {
    const exts = ['build.gradle', 'build.gradle.kts', 'settings.gradle', 'settings.gradle.kts'];
    const findFiles = (dir) => {
        return fs.readdirSync(dir, { withFileTypes: true }).flatMap(entry => {
            const fullPath = path.join(dir, entry.name);
            return entry.isDirectory() ? findFiles(fullPath) : exts.some(ext => entry.name.endsWith(ext)) ? [fullPath] : [];
        });
    };
    findFiles(FIND_DIR).forEach(processFile);
}

main();
