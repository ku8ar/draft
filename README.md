gradle.projectsLoaded {
  allprojects {
    afterEvaluate { project ->
      if (!project.hasProperty("android")) return

      def androidExt = project.android

      // 1. compileOptions
      if (!androidExt.hasProperty("compileOptions") || !androidExt.compileOptions?.sourceCompatibility) {
        androidExt.compileOptions {
          sourceCompatibility = JavaVersion.VERSION_1_8
          targetCompatibility = JavaVersion.VERSION_1_8
        }
        println "✅ ${project.name}: ustawiono compileOptions na Java 1.8"
      }

      // 2. kotlinOptions.jvmTarget
      if (project.plugins.hasPlugin("kotlin-android")) {
        androidExt.kotlinOptions {
          if (!delegate.hasProperty("jvmTarget") || !jvmTarget) {
            jvmTarget = "1.8"
            println "✅ ${project.name}: ustawiono kotlinOptions.jvmTarget = 1.8"
          }
        }
      }
    }
  }
}
