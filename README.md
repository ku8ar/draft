# Source directory (local Gradle cache)
GRADLE_CACHE="$HOME/.gradle/caches/modules-2/files-2.1"

# Target directory (Maven proxy repo)
PROXY_CACHE="./.maven-proxy-cache"

# Check if Gradle cache exists
if [ ! -d "$GRADLE_CACHE" ]; then
    echo "❌ ERROR: Gradle cache directory not found: $GRADLE_CACHE"
    exit 1
fi

echo "✅ Syncing Gradle cache from $GRADLE_CACHE to $PROXY_CACHE..."

# Iterate over all relevant artifact files
find "$GRADLE_CACHE" -type f \( -name "*.jar" -o -name "*.aar" -o -name "*.pom" -o -name "*.module" \) | while read -r file; do
    # Relative path inside files-2.1 structure
    relpath="${file#$GRADLE_CACHE/}"

    # Target path in .maven-proxy-cache
    target="$PROXY_CACHE/$relpath"

    # Ensure target directory exists
    mkdir -p "$(dirname "$target")"

    # Copy the file
    cp "$file" "$target"

    echo "[SYNC] $relpath"
done

echo "✅ Gradle cache sync completed."

# Optional: Generate missing SHA1 checksums (if generate-sha1.sh exists)
if [ -f "./generate-sha1.sh" ]; then
    echo "🔄 Running SHA1 checksum generation..."
    ./generate-sha1.sh
fi

echo "✅ All done. You can now push .maven-proxy-cache to Bitbucket."
