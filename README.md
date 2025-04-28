package com.example.yourpackage

import android.app.Application
import android.os.Bundle
import android.widget.FrameLayout
import androidx.appcompat.app.AppCompatActivity
import com.facebook.react.ReactInstanceManager
import com.facebook.react.ReactRootView
import com.facebook.react.shell.MainReactPackage
import com.facebook.react.common.LifecycleState
import com.facebook.soloader.SoLoader
import com.facebook.react.soloader.OpenSourceMergedSoMapping

// Dołącz swoje własne paczki jeśli trzeba
import com.onfido.reactnative.sdk.OnfidoSdkPackage
import com.callstack.repack.ScriptManagerPackage
import com.th3rdwave.safeareacontext.SafeAreaContextPackage
import com.swmansion.rnscreens.RNScreensPackage

class RNBridgeActivity : AppCompatActivity() {

    private lateinit var reactRootView: ReactRootView
    private lateinit var instanceManager: ReactInstanceManager

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val app = application as Application

        // SoLoader musi być zainicjalizowany RAZ na proces!
        // Jeśli już gdzieś w host-app jest, to pomiń ten blok
        SoLoader.init(app, OpenSourceMergedSoMapping)

        reactRootView = ReactRootView(this)
        instanceManager = ReactInstanceManager.builder()
            .setApplication(app)
            .setCurrentActivity(this)
            .setBundleAssetName("index.android.bundle")
            .setJSMainModulePath("index")
            .addPackage(MainReactPackage())
            .addPackage(OnfidoSdkPackage())
            .addPackage(ScriptManagerPackage())
            .addPackage(SafeAreaContextPackage())
            .addPackage(RNScreensPackage())
            .setUseDeveloperSupport(false)
            .setInitialLifecycleState(LifecycleState.RESUMED)
            .build()

        // Tu możesz przekazać parametry do aplikacji JS:
        val params = intent.getStringExtra("paramsFromHost")
        val initialProps = Bundle().apply {
            putString("params", params)
        }

        reactRootView.startReactApplication(
            instanceManager,
            "rnbridge", // <- nazwa root componentu JS
            initialProps
        )

        setContentView(
            FrameLayout(this).apply {
                addView(reactRootView, FrameLayout.LayoutParams(
                    FrameLayout.LayoutParams.MATCH_PARENT,
                    FrameLayout.LayoutParams.MATCH_PARENT
                ))
            }
        )
    }

    override fun onPause() {
        super.onPause()
        instanceManager.onHostPause(this)
    }

    override fun onResume() {
        super.onResume()
        instanceManager.onHostResume(this, this)
    }

    override fun onDestroy() {
        super.onDestroy()
        instanceManager.onHostDestroy(this)
        reactRootView.unmountReactApplication()
    }
}
