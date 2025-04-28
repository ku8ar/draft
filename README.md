package com.bridge

import android.app.Activity
import android.os.Bundle
import com.facebook.react.ReactActivityDelegate
import com.facebook.react.ReactRootView
import com.facebook.react.ReactInstanceManager
import com.facebook.react.common.LifecycleState
import com.facebook.react.soloader.OpenSourceMergedSoMapping
import com.facebook.soloader.SoLoader
import com.facebook.react.shell.MainReactPackage

// MANUAL LINKING
import com.onfido.reactnative.sdk.OnfidoSdkPackage
import com.callstack.repack.ScriptManagerPackage
import com.th3rdwave.safeareacontext.SafeAreaContextPackage
import com.swmansion.rnscreens.RNScreensPackage
// END MANUAL LINKING

class BridgeReactActivity : Activity() {

    private var reactRootView: ReactRootView? = null
    private var instanceManager: ReactInstanceManager? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // SoLoader + instanceManager jak poprzednio
        val application = application as android.app.Application
        SoLoader.init(application, OpenSourceMergedSoMapping)

        instanceManager = ReactInstanceManager.builder()
            .setApplication(application)
            .setCurrentActivity(this)
            .setBundleAssetName("index.android.bundle")
            .setJSMainModulePath("index")
            .addPackage(MainReactPackage())
            .addPackage(OnfidoSdkPackage())
            .addPackage(ScriptManagerPackage())
            .addPackage(SafeAreaContextPackage())
            .addPackage(RNScreensPackage())
            .setUseDeveloperSupport(false)
            .setInitialLifecycleState(LifecycleState.BEFORE_CREATE)
            .build()

        reactRootView = ReactRootView(this)
        reactRootView?.startReactApplication(
            instanceManager,
            "rnbridge",
            null
        )

        setContentView(reactRootView)
    }

    override fun onResume() {
        super.onResume()
        instanceManager?.onHostResume(this, null)
    }

    override fun onPause() {
        super.onPause()
        instanceManager?.onHostPause(this)
    }

    override fun onDestroy() {
        super.onDestroy()
        instanceManager?.onHostDestroy(this)
        reactRootView?.unmountReactApplication()
    }

    // (opcjonalnie) obsłuż back button
    override fun onBackPressed() {
        instanceManager?.onBackPressed()
        // Jeżeli RN nie obsłuży, wywołaj domyślnie
        super.onBackPressed()
    }
}
