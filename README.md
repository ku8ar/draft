package com.tealium.bridge

import com.tealium.core.Tealium

class RNTealium private constructor() {

    companion object {
        @JvmStatic
        fun shared(): RNTealium = Holder.INSTANCE
    }

    private object Holder {
        val INSTANCE = RNTealium()
    }

    @Volatile
    internal var tealiumInstance: Tealium? = null
        set(value) {
            field = value
            // Jeśli RNTealiumModule już istnieje ➔ próbujemy wstrzyknąć
            RNTealiumModule.tryInjectTealium()
        }

    fun configure(tealium: Tealium) {
        tealiumInstance = tealium
    }
}



package com.tealium.bridge

import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.bridge.ReactContextBaseJavaModule
import com.tealium.react.TealiumReact

class RNTealiumModule(private val reactContext: ReactApplicationContext) : ReactContextBaseJavaModule(reactContext) {

    init {
        instance = this
        tryInjectTealium()
    }

    override fun getName(): String = "RNTealium"

    companion object {
        @Volatile
        private var instance: RNTealiumModule? = null

        internal fun tryInjectTealium() {
            val module = instance ?: return
            val tealium = RNTealium.shared().tealiumInstance ?: return

            val tealiumReact = module.reactContext.nativeModuleRegistry.getModule(TealiumReact::class.java)
                ?: throw IllegalStateException("TealiumReact module not initialized.")

            tealiumReact.tealium = tealium
        }
    }
}
