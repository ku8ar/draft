#!/bin/bash

set -e

# Optional: Which group to use in Maven
GROUP_ID="com.yourorg.rn"

# Function to inject maven-publish plugin and publishing block if needed
inject_publishing() {
  local gradlefile="$1"
  local artifact="$2"
  local version="$3"
  local publishblock="
apply plugin: 'maven-publish'

afterEvaluate {
  publishing {
    publications {
      release(MavenPublication) {
        groupId = '$GROUP_ID'
        artifactId = '$artifact'
        version = '$version'
        from components.release
      }
    }
    repositories {
      mavenLocal()
    }
  }
}
"
  # Only inject if not present
  if ! grep -q "maven-publish" "$gradlefile"; then
    echo "Injecting maven-publish to $gradlefile"
    echo "$publishblock" >> "$gradlefile"
  fi
}

echo ""
echo "Scanning for native RN modules (with android/build.gradle) in ../node_modules..."

find ../node_modules -type f -path "*/android/build.gradle" | while read gradlefile; do
  PKGDIR=$(dirname "$(dirname "$gradlefile")")
  PKGNAME=$(basename "$PKGDIR")
  ARTIFACT_ID="$PKGNAME"
  # Get version from package.json
  VERSION="1.0.0"
  if [ -f "$PKGDIR/package.json" ]; then
    VERSION=$(grep '"version"' "$PKGDIR/package.json" | head -n 1 | sed -E 's/[^0-9.]//g')
  fi

  inject_publishing "$gradlefile" "$ARTIFACT_ID" "$VERSION"

  echo ""
  echo "Building and publishing $ARTIFACT_ID@$VERSION to mavenLocal..."
  cd "$PKGDIR/android"
  ./gradlew publishReleasePublicationToMavenLocal --quiet || true
  cd - > /dev/null
done

echo ""
echo "All done! Check your ~/.m2/repository/$GROUP_ID/"
echo "Now you can use these packages as Maven dependencies in your bridge or host app."
