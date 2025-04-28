package com.bridge

import android.app.Application
import com.facebook.react.soloader.OpenSourceMergedSoMapping
import com.facebook.soloader.SoLoader

object BridgeInitializer {
    private var initialized = false

    @JvmStatic
    fun init(application: Application) {
        if (!initialized) {
            SoLoader.init(application, OpenSourceMergedSoMapping)
            initialized = true
        }
    }
}
