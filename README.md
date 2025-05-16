
echo "Replacing google() and replacing mavenCentral() in node_modules..."

FIND_DIR="../node_modules"
REPO_URL="https://my.rpoxy"
REPO_USERNAME_VAR="MY_REPO_USERNAME"
REPO_PASSWORD_VAR="MY_REPO_PASSWORD"

REPLACEMENT_GROOVY="maven {
    url \"$REPO_URL\"
    credentials {
        username = System.getenv(\"$REPO_USERNAME_VAR\") ?: \"defaultUser\"
        password = System.getenv(\"$REPO_PASSWORD_VAR\") ?: \"defaultPass\"
    }
}"

REPLACEMENT_KTS="maven {
    url = uri(\"$REPO_URL\")
    credentials {
        username = System.getenv(\"$REPO_USERNAME_VAR\") ?: \"defaultUser\"
        password = System.getenv(\"$REPO_PASSWORD_VAR\") ?: \"defaultPass\"
    }
}"

find "$FIND_DIR" -type f \( -name "build.gradle" -o -name "build.gradle.kts" -o -name "settings.gradle" -o -name "settings.gradle.kts" \) | while read -r file; do
    echo "Processing $file"
    tmp_file="${file}.tmp"

    if [[ "$file" == *.kts ]]; then
        awk -v replacement="$REPLACEMENT_KTS" '
            /^[[:space:]]*mavenCentral\(\)[[:space:]]*$/ { print replacement; next }
            /^[[:space:]]*google\(\)[[:space:]]*$/ { next }
            { print }
        ' "$file" > "$tmp_file"
    else
        awk -v replacement="$REPLACEMENT_GROOVY" '
            /^[[:space:]]*mavenCentral\(\)[[:space:]]*$/ { print replacement; next }
            /^[[:space:]]*google\(\)[[:space:]]*$/ { next }
            { print }
        ' "$file" > "$tmp_file"
    fi

    mv "$tmp_file" "$file" || true
done

echo "Done."
