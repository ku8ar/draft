#!/bin/bash

set -e

echo "Fixing mavenCentral() in node_modules..."

FIND_DIR="../node_modules"
REPLACEMENT='maven {\n    url "https://my.rpoxy"\n    credentials {\n        username dupa\n        password dupa\n    }\n}'

find "$FIND_DIR" -type f -name "build.gradle" | while read -r file; do
  echo "Processing $file"
  # MacOS (BSD) sed syntax
  sed -i '' "s/mavenCentral()/$REPLACEMENT/g" "$file"
done

echo "Done."
