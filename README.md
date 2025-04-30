package com.yourapp

import com.facebook.react.bridge.*
import kotlinx.coroutines.*
import okhttp3.Request
import java.io.IOException

class DBSFetchModule(reactContext: ReactApplicationContext) :
    ReactContextBaseJavaModule(reactContext) {

    override fun getName(): String = "DBSFetchModule"

    @ReactMethod
    fun fetch(url: String, promise: Promise) {
        CoroutineScope(Dispatchers.IO).launch {
            try {
                val request = Request.Builder().url(url).build()
                val response = DBSFetch.shared().getClient().newCall(request).execute()

                if (!response.isSuccessful) {
                    promise.reject("HTTP_ERROR", "HTTP error: ${response.code}")
                    return@launch
                }

                val body = response.body?.string() ?: ""
                promise.resolve(body)

            } catch (e: IOException) {
                promise.reject("NETWORK_ERROR", e.message, e)
            }
        }
    }
}
