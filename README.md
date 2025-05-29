import groovy.json.JsonSlurper

// 1. Odczytaj wersję react-native z package.json
def rootDir = rootProject.rootDir
def packageJsonFile = new File(rootDir, "package.json")

if (!packageJsonFile.exists()) {
  throw new GradleException("❌ Nie znaleziono package.json w ${packageJsonFile}")
}

def packageJson = new JsonSlurper().parseText(packageJsonFile.text)
def rnVersionRaw =
  packageJson.dependencies?.get("react-native") ?:
  packageJson.devDependencies?.get("react-native")

if (!rnVersionRaw) {
  throw new GradleException("❌ Nie znaleziono react-native w dependencies w package.json")
}

// Usuń prefixy ^, ~, >=, itp.
def rnVersion = rnVersionRaw.replaceAll(/^[^\d]*/, "")

println "🔧 Ustawiam wersję React Native: $rnVersion"
def group = "com.facebook.react"

allprojects {
  afterEvaluate { project ->
    project.configurations.matching { it.canBeResolved }.all { config ->
      config.dependencies.withType(ModuleDependency).configureEach { dep ->
        if (dep.group == group &&
            (dep.name == "react-native" || dep.name == "react-android") &&
            (dep.version == "+" || dep.version == "latest.release")) {

          println "🔄 ${project.name}: ${dep.group}:${dep.name}:${dep.version} → $rnVersion"
          config.dependencies.remove(dep)
          config.dependencies.add(
            project.dependencies.create("$group:${dep.name}:$rnVersion")
          )
        }
      }
    }
  }
}
