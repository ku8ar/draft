#!/bin/bash

set -e

echo "Replacing mavenCentral() with env-based maven block..."

FIND_DIR="../node_modules"
REPLACEMENT='maven {\n    url "https://my.rpoxy"\n    credentials {\n        username = System.getenv("MY_REPO_USERNAME") ?: "defaultUser"\n        password = System.getenv("MY_REPO_PASSWORD") ?: "defaultPass"\n    }\n}'

find "$FIND_DIR" -type f -name "build.gradle" | while read -r file; do
  echo "Processing $file"
  sed -i '' "s|mavenCentral()|$REPLACEMENT|g" "$file"
done

echo "Done."
