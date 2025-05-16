echo "Safe replacing google() and mavenCentral() in node_modules..."

FIND_DIR="../node_modules"
REPLACEMENT='maven {
    url "https://my.rpoxy"
    credentials {
        username = System.getenv("MY_REPO_USERNAME") ?: "defaultUser"
        password = System.getenv("MY_REPO_PASSWORD") ?: "defaultPass"
    }
}'

find "$FIND_DIR" -type f -name "build.gradle" | while read -r file; do
    echo "Processing $file"
    tmp_file="${file}.tmp"
    
    # Usuń google(), zamień mavenCentral()
    sed "s|mavenCentral()|$REPLACEMENT|g" "$file" | sed '/^[[:space:]]*google()[[:space:]]*$/d' > "$tmp_file"
    
    mv "$tmp_file" "$file"
done

echo "Done."
