package com.yourpackage

import com.facebook.react.bridge.*
import okhttp3.*
import java.io.IOException

class DBSFetchModule(reactContext: ReactApplicationContext) :
    ReactContextBaseJavaModule(reactContext) {

    override fun getName(): String = "DBSFetchModule"

    @ReactMethod
    fun fetch(request: ReadableMap, promise: Promise) {
        val url = request.getString("url")
        val method = request.getString("method")?.uppercase() ?: "GET"
        val headersMap = request.getMap("headers")
        val body = if (request.hasKey("body")) request.getString("body") else null

        if (url.isNullOrBlank()) {
            promise.reject("TypeError", "Invalid URL")
            return
        }

        val client = DBSFetch.shared().getClient()
        val requestBuilder = Request.Builder().url(url)

        // Headers
        headersMap?.entryIterator?.forEach { entry ->
            val key = entry.key
            val value = entry.value?.toString() ?: ""
            requestBuilder.addHeader(key, value)
        }

        // Body
        val requestBody: RequestBody? = when {
            method == "POST" || method == "PUT" || method == "PATCH" -> {
                val mediaType = "application/json".toMediaTypeOrNull()
                body?.toRequestBody(mediaType)
            }
            else -> null
        }

        requestBuilder.method(method, requestBody)

        client.newCall(requestBuilder.build()).enqueue(object : Callback {
            override fun onFailure(call: Call, e: IOException) {
                promise.reject("TypeError", e.message, e)
            }

            override fun onResponse(call: Call, response: Response) {
                val responseBody = response.body?.string() ?: ""
                val headersMap = Arguments.createMap()

                for ((name, value) in response.headers) {
                    headersMap.putString(name, value)
                }

                val result = Arguments.createMap().apply {
                    putInt("status", response.code)
                    putMap("headers", headersMap)
                    putString("body", responseBody)
                }

                promise.resolve(result)
            }
        })
    }
}
