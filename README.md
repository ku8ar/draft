package com.bridge

import android.app.Application
import android.content.Context
import android.widget.FrameLayout
import com.facebook.react.ReactInstanceManager
import com.facebook.react.ReactRootView
import com.facebook.react.ReactPackage
import com.facebook.react.shell.MainReactPackage
import com.facebook.react.common.LifecycleState

class BridgeReactView(context: Context) : FrameLayout(context) {

    private val reactRootView: ReactRootView
    private val reactInstanceManager: ReactInstanceManager

    init {
        val application = context.applicationContext as Application

        reactRootView = ReactRootView(context)
        reactInstanceManager = ReactInstanceManager.builder()
            .setApplication(application)
            .setCurrentActivity(null) // możesz później dodać, jeśli potrzebujesz dostępu do Activity
            .setBundleAssetName("index.android.bundle") // bundle osadzony w .aar
            .setJSMainModulePath("index") // nieużywane przy assetach
            .addPackage(MainReactPackage())
            .setUseDeveloperSupport(false)
            .setInitialLifecycleState(LifecycleState.RESUMED)
            .build()

        reactRootView.startReactApplication(
            reactInstanceManager,
            "BridgeApp", // to musi się zgadzać z AppRegistry.registerComponent w JS
            null
        )

        addView(reactRootView, LayoutParams(
            LayoutParams.MATCH_PARENT,
            LayoutParams.MATCH_PARENT
        ))
    }
}
