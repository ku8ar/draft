set +e  # Don't crash the script on errors

echo "Safe replacing google() and mavenCentral() in node_modules..."

FIND_DIR="../node_modules"

# Centralized config
REPO_URL="https://my.rpoxy"
REPO_USERNAME_VAR="MY_REPO_USERNAME"
REPO_PASSWORD_VAR="MY_REPO_PASSWORD"

# Replacement for Groovy DSL (build.gradle, settings.gradle)
REPLACEMENT_GROOVY="maven {
    url \"$REPO_URL\"
    credentials {
        username = System.getenv(\"$REPO_USERNAME_VAR\") ?: \"defaultUser\"
        password = System.getenv(\"$REPO_PASSWORD_VAR\") ?: \"defaultPass\"
    }
}"

# Replacement for Kotlin DSL (build.gradle.kts, settings.gradle.kts)
REPLACEMENT_KTS="maven {
    url = uri(\"$REPO_URL\")
    credentials {
        username = System.getenv(\"$REPO_USERNAME_VAR\") ?: \"defaultUser\"
        password = System.getenv(\"$REPO_PASSWORD_VAR\") ?: \"defaultPass\"
    }
}"

# Obsługujemy build.gradle, build.gradle.kts, settings.gradle, settings.gradle.kts
find "$FIND_DIR" -type f \( -name "build.gradle" -o -name "build.gradle.kts" -o -name "settings.gradle" -o -name "settings.gradle.kts" \) | while read -r file; do
    echo "Processing $file"
    tmp_file="${file}.tmp"

    if [[ "$file" == *.kts ]]; then
        # Kotlin DSL
        sed "s|mavenCentral()|$REPLACEMENT_KTS|g" "$file" | sed '/^[[:space:]]*google()[[:space:]]*$/d' > "$tmp_file"
    else
        # Groovy DSL
        sed "s|mavenCentral()|$REPLACEMENT_GROOVY|g" "$file" | sed '/^[[:space:]]*google()[[:space:]]*$/d' > "$tmp_file"
    fi

    mv "$tmp_file" "$file" || true
done

echo "Done."
