REPLACEMENT_GROOVY=$(cat <<EOF
maven {
    url "$REPO_URL"
    credentials {
        username = System.getenv("$REPO_USERNAME_VAR") ?: "defaultUser"
        password = System.getenv("$REPO_PASSWORD_VAR") ?: "defaultPass"
    }
}
EOF
)

REPLACEMENT_KTS=$(cat <<EOF
maven {
    url = uri("$REPO_URL")
    credentials {
        username = System.getenv("$REPO_USERNAME_VAR") ?: "defaultUser"
        password = System.getenv("$REPO_PASSWORD_VAR") ?: "defaultPass"
    }
}
EOF
)

find "$FIND_DIR" -type f \( -name "build.gradle" -o -name "build.gradle.kts" -o -name "settings.gradle" -o -name "settings.gradle.kts" \) | while read -r file; do
    echo "Processing $file"
    tmp_file="${file}.tmp"

    if [[ "$file" == *.kts ]]; then
        sed "s|mavenCentral()|$REPLACEMENT_KTS|g" "$file" | sed '/^[[:space:]]*google()[[:space:]]*$/d' > "$tmp_file"
    else
        sed "s|mavenCentral()|$REPLACEMENT_GROOVY|g" "$file" | sed '/^[[:space:]]*google()[[:space:]]*$/d' > "$tmp_file"
    fi

    mv "$tmp_file" "$file" || true
done

echo "Done."
