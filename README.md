package com.yourpackage

import com.facebook.react.bridge.*
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch
import okhttp3.*
import okhttp3.MediaType.Companion.toMediaTypeOrNull
import okhttp3.RequestBody.Companion.toRequestBody

class DBSFetchModule(private val reactContext: ReactApplicationContext) :
    ReactContextBaseJavaModule(reactContext) {

    override fun getName(): String = "DBSFetchModule"

    private val client: OkHttpClient
        get() = DBSFetch.shared().getClient()

    private val scope = CoroutineScope(Dispatchers.IO)

    @ReactMethod
    fun fetch(request: ReadableMap, promise: Promise) {
        val url = request.getString("url")
        val method = request.getString("method")?.uppercase() ?: "GET"
        val headers = request.getMap("headers")
        val body = if (request.hasKey("body")) request.getString("body") else null

        if (url.isNullOrBlank()) {
            promise.reject("TypeError", "Invalid URL")
            return
        }

        scope.launch {
            try {
                val requestBuilder = Request.Builder().url(url)

                // Headers
                headers?.entryIterator?.forEach { entry ->
                    val key = entry.key
                    val value = entry.value?.toString() ?: ""
                    requestBuilder.addHeader(key, value)
                }

                // Body
                val requestBody = when {
                    method in listOf("POST", "PUT", "PATCH") && body != null -> {
                        "application/json".toMediaTypeOrNull()?.let {
                            body.toRequestBody(it)
                        }
                    }
                    else -> null
                }

                requestBuilder.method(method, requestBody)

                val response = client.newCall(requestBuilder.build()).execute()

                val responseHeaders = Arguments.createMap().apply {
                    for ((name, value) in response.headers) {
                        putString(name, value)
                    }
                }

                val responseBody = response.body?.string().orEmpty()

                val result = Arguments.createMap().apply {
                    putInt("status", response.code)
                    putMap("headers", responseHeaders)
                    putString("body", responseBody)
                }

                promise.resolve(result)
            } catch (e: Exception) {
                promise.reject("TypeError", e.message, e)
            }
        }
    }
}
