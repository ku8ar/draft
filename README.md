// Ładowanie loginu i hasła z local.properties
def localProps = new Properties()
def localPropsFile = rootProject.file('local.properties')
if (localPropsFile.exists()) {
    localProps.load(new FileInputStream(localPropsFile))
    localProps.each { key, value -> project.ext.set(key, value) }
}

buildscript {
    ext {
        buildToolsVersion = "35.0.0"
        minSdkVersion = 24
        compileSdkVersion = 35
        targetSdkVersion = 35
        ndkVersion = "27.1.12297006"
        kotlinVersion = "2.0.21"
    }

    repositories {
        maven {
            url "https://nexus.twojafirma.pl/repository/maven-public/"
            credentials {
                username = project.findProperty("nexusUsername") ?: ""
                password = project.findProperty("nexusPassword") ?: ""
            }
        }
        google() // opcjonalnie, można też zmirrorować przez Nexusa
    }

    dependencies {
        classpath("com.android.tools.build:gradle")
        classpath("com.facebook.react:react-native-gradle-plugin")
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin")
    }
}

allprojects {
    repositories {
        maven {
            url "https://nexus.twojafirma.pl/repository/maven-public/"
            credentials {
                username = project.findProperty("nexusUsername") ?: ""
                password = project.findProperty("nexusPassword") ?: ""
            }
        }
        google()
    }
}

apply plugin: "com.facebook.react.rootproject"
