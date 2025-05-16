function processFile(filePath) {
    const isKts = filePath.endsWith('.kts');
    const lines = fs.readFileSync(filePath, 'utf8').split(/\r?\n/);

    const newLines = lines.flatMap(line => {
        const trimmed = line.trim();
        if (trimmed === 'google()') return [];
        if (trimmed === 'mavenCentral()') return [isKts ? TEMPLATES.kts : TEMPLATES.groovy];
        return [line];
    });

    fs.writeFileSync(filePath, newLines.join('\n'), 'utf8');
    console.log(`✔ Processed ${filePath}`);
}
