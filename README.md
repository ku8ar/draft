ROOT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"

# Extract React Native version from package.json using grep/sed (no jq)
RN_VERSION=$(grep -A1 '"react-native"' "$ROOT_DIR/package.json" | grep -Eo '[0-9]+\.[0-9]+\.[0-9]+' | head -n1)

if [[ -z "$RN_VERSION" ]]; then
  echo "Could not find React Native version in package.json"
  exit 1
fi

echo "Detected React Native version: $RN_VERSION"

# Find and update build.gradle and build.gradle.kts files
find "$ROOT_DIR/node_modules" \( -name "build.gradle" -o -name "build.gradle.kts" \) | while read -r FILE; do
  if grep -qE "['\"]com\.facebook\.react:react-android:\+['\"]" "$FILE"; then
    echo "Updating: $FILE"
    sed -i.bak -E "s|([\"']com\.facebook\.react:react-android:)\+([\"'])|\1$RN_VERSION\2|g" "$FILE"
    rm -f "${FILE}.bak"
  fi
done

echo "Finished updating react-android version"
