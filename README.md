import java.util.Properties

val userGradleProperties = Properties()
val userGradleFile = File(System.getProperty("user.home"), ".gradle/gradle.properties")

if (userGradleFile.exists()) {
    userGradleFile.inputStream().use { userGradleProperties.load(it) }
} else {
    println("WARNING: ~/.gradle/gradle.properties not found")
}

val username = userGradleProperties.getProperty("myUsername") ?: "defaultUser"
val password = userGradleProperties.getProperty("myPassword") ?: "defaultPass"

println("Loaded credentials: $username / $password")
