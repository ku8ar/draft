package com.yourpackage

import com.facebook.react.bridge.*
import java.io.BufferedReader
import java.io.InputStreamReader
import java.net.HttpURLConnection
import java.net.URL

class DBSFetchModule(reactContext: ReactApplicationContext) :
    ReactContextBaseJavaModule(reactContext) {

    override fun getName(): String = "DBSFetchModule"

    @ReactMethod
    fun fetch(request: ReadableMap, promise: Promise) {
        val urlString = request.getString("url")
        val method = request.getString("method") ?: "GET"
        val headers = request.getMap("headers")
        val body = if (request.hasKey("body")) request.getString("body") else null

        if (urlString.isNullOrEmpty()) {
            promise.reject("TypeError", "Invalid URL")
            return
        }

        try {
            val url = URL(urlString)
            val connection = url.openConnection() as HttpURLConnection

            connection.requestMethod = method
            connection.doInput = true
            connection.connectTimeout = 10000
            connection.readTimeout = 15000

            // Headers
            headers?.entryIterator?.forEach { entry ->
                val key = entry.key
                val value = entry.value?.toString() ?: ""
                connection.setRequestProperty(key, value)
            }

            // Body (optional, for POST/PUT/etc)
            if (!body.isNullOrEmpty() && method in listOf("POST", "PUT", "PATCH")) {
                connection.doOutput = true
                connection.outputStream.use { os ->
                    os.write(body.toByteArray())
                }
            }

            // Read response
            val responseCode = connection.responseCode
            val responseHeaders = Arguments.createMap().apply {
                for ((key, value) in connection.headerFields) {
                    if (key != null && value != null) {
                        putString(key, value.joinToString(", "))
                    }
                }
            }

            val inputStream = if (responseCode >= 400)
                connection.errorStream ?: connection.inputStream
            else
                connection.inputStream

            val bodyString = inputStream?.bufferedReader()?.use(BufferedReader::readText) ?: ""

            val response = Arguments.createMap().apply {
                putInt("status", responseCode)
                putMap("headers", responseHeaders)
                putString("body", bodyString)
            }

            promise.resolve(response)
        } catch (e: Exception) {
            promise.reject("TypeError", e.message, e)
        }
    }
}
