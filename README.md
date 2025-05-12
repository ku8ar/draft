#!/bin/bash

# Wejdź do katalogu .maven-proxy-cache
cd .maven-proxy-cache || { echo "Nie znaleziono katalogu .maven-proxy-cache"; exit 1; }

# Wypisz co robimy
echo "Generowanie brakujących plików .sha1 dla artefaktów (.jar, .aar, .pom, .module)..."

# Przejdź po wszystkich plikach .jar, .aar, .pom, .module
find . -type f \( -name "*.jar" -o -name "*.aar" -o -name "*.pom" -o -name "*.module" \) | while read -r file; do
    sha1file="${file}.sha1"

    if [ -f "$sha1file" ]; then
        echo "[OK] $sha1file istnieje"
    else
        echo "[GEN] Generuję $sha1file"
        sha1sum "$file" | awk '{print $1}' > "$sha1file"
    fi
done

echo "Zakończono generowanie checksum SHA1."
