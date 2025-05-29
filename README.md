import groovy.json.JsonSlurper

def rootDir = rootProject.rootDir
def packageJsonFile = new File(rootDir, "package.json")

if (!packageJsonFile.exists()) {
  throw new GradleException("❌ Nie znaleziono package.json w ${packageJsonFile}")
}

def packageJson = new JsonSlurper().parseText(packageJsonFile.text)
def rnVersionRaw = packageJson.dependencies["react-native"] ?: packageJson.devDependencies["react-native"]

if (!rnVersionRaw) {
  throw new GradleException("❌ Nie znaleziono react-native w package.json")
}

// Oczyść wersję z np. "^0.79.0", "~0.78.2" itp.
def rnVersion = rnVersionRaw.replaceAll(/^[^0-9]+/, "") // usunie ^, ~, >= itp.

println "🔧 Zastępuję react-native:+ → react-android:$rnVersion"

allprojects {
  afterEvaluate { project ->
    project.configurations.matching { it.canBeResolved }.all { config ->
      config.dependencies.withType(ModuleDependency).configureEach { dep ->
        if (dep.group == "com.facebook.react" && dep.name == "react-native") {
          println "🔄 ${project.name}: ${dep.group}:${dep.name}:${dep.version} → react-android:$rnVersion"
          config.dependencies.remove(dep)
          config.dependencies.add(
            project.dependencies.create("com.facebook.react:react-android:$rnVersion")
          )
        }
      }
    }
  }
}
