# Use Maven Local as default
if [ -n "$MAVEN_REPO_LOCAL" ]; then
  MY_MAVEN_REPO="$MAVEN_REPO_LOCAL"
elif [ -n "$MAVEN_USER_HOME" ]; then
  MY_MAVEN_REPO="$MAVEN_USER_HOME/repository"
else
  MY_MAVEN_REPO="$HOME/.m2/repository"
fi

GROUP_ID="com.yourorg.rn"

get_version() {
  PKGDIR="$1"
  if [ -f "$PKGDIR/package.json" ]; then
    VERSION=$(grep '"version"' "$PKGDIR/package.json" | head -n 1 | sed -E 's/[^0-9.]//g')
    echo "$VERSION"
  else
    echo "1.0.0"
  fi
}

echo "Maven local repo: $MY_MAVEN_REPO"
echo "Scanning for .aar files in ../node_modules..."

find ../node_modules -type f -name "*.aar" | while read aarfile; do
  # Extract package directory (the last folder before /android/)
  PKGDIR=$(echo "$aarfile" | grep -o '../node_modules/[^/]*/' | head -n 1)
  PKGNAME=$(basename "$PKGDIR")
  ARTIFACT_ID="$PKGNAME"

  # Get version from package.json or .aar file name
  ABS_PKGDIR=$(cd "$PKGDIR" 2>/dev/null && pwd)
  VERSION=$(get_version "$ABS_PKGDIR")
  AAR_BASENAME=$(basename "$aarfile")
  if [[ "$AAR_BASENAME" =~ ([0-9]+\.[0-9]+\.[0-9]+) ]]; then
    VERSION="${BASH_REMATCH[1]}"
  fi

  DEST="$MY_MAVEN_REPO/$(echo $GROUP_ID | tr '.' '/')/$ARTIFACT_ID/$VERSION"

  mkdir -p "$DEST"
  cp "$aarfile" "$DEST/$ARTIFACT_ID-$VERSION.aar"

  cat > "$DEST/$ARTIFACT_ID-$VERSION.pom" <<EOF
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>$GROUP_ID</groupId>
  <artifactId>$ARTIFACT_ID</artifactId>
  <version>$VERSION</version>
  <packaging>aar</packaging>
</project>
EOF

  echo "Published $ARTIFACT_ID-$VERSION.aar to $DEST/"
done

echo ""
echo "Done! Your Maven local repo path: $MY_MAVEN_REPO"
echo "In your host app, use:"
echo "repositories { mavenLocal(); ... }"
