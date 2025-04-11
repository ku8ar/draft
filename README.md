pluginManagement {
    includeBuild("../node_modules/@react-native/gradle-plugin")

    repositories {
        maven {
            url "https://nexus.twojafirma.pl/repository/maven-public/"
            credentials {
                username = nexusUsername
                password = nexusPassword
            }
        }
        gradlePluginPortal()
        google()
    }
}

plugins {
    id("com.facebook.react.settings")
}

extensions.configure(com.facebook.react.ReactSettingsExtension) {
    it.autolinkLibrariesFromCommand()
}

rootProject.name = "YourApp"
include(":app")
includeBuild("../node_modules/@react-native/gradle-plugin")

// 🔐 Wczytaj dane logowania z local.properties
def localProps = new Properties()
def localPropsFile = new File(rootDir, "local.properties")
def nexusUsername = ""
def nexusPassword = ""

if (localPropsFile.exists()) {
    localProps.load(new FileInputStream(localPropsFile))
    nexusUsername = localProps.getProperty("nexusUsername") ?: ""
    nexusPassword = localProps.getProperty("nexusPassword") ?: ""
}

// 🧱 Główne miejsce: wymuś repozytoria tylko z settings.gradle
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)

    repositories {
        maven {
            url "https://nexus.twojafirma.pl/repository/maven-public/"
            credentials {
                username = nexusUsername
                password = nexusPassword
            }
        }
        google()
    }
}
