RN_VERSION=$(jq -r '.dependencies["react-native"]' package.json | sed 's/[^0-9.]//g')
