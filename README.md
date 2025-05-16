android {
    packagingOptions {
        pickFirst 'lib/arm64-v8a/libc++_shared.so'
        pickFirst 'lib/x86_64/libc++_shared.so'
        pickFirst 'lib/x86/libc++_shared.so'
        pickFirst 'lib/armeabi-v7a/libc++_shared.so'
    }
}


android {
    packaging {
        jniLibs {
            pickFirsts += [
                'lib/arm64-v8a/libc++_shared.so',
                'lib/x86_64/libc++_shared.so',
                'lib/x86/libc++_shared.so',
                'lib/armeabi-v7a/libc++_shared.so'
            ]
        }
    }
}
