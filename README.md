find ../node_modules -type d -path '*/android' -exec bash -c 'cd "$0" && ./gradlew assembleRelease || true' {} \;
