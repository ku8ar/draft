package com.tealium.bridge

import com.facebook.react.ReactApplication
import com.facebook.react.ReactInstanceManager
import com.tealium.core.Tealium
import com.tealium.react.TealiumReact

class RNTealium private constructor() {

    companion object {
        @Volatile
        private var instance: RNTealium? = null

        fun getInstance(): RNTealium {
            return instance ?: synchronized(this) {
                instance ?: RNTealium().also { instance = it }
            }
        }
    }

    fun configure(tealiumInstance: Tealium, reactApp: ReactApplication) {
        val reactInstanceManager: ReactInstanceManager = reactApp.reactNativeHost.reactInstanceManager
        val nativeModule = reactInstanceManager.currentReactContext?.nativeModuleRegistry
            ?.getModule(TealiumReact::class.java)

        nativeModule?.tealium = tealiumInstance
    }
}
