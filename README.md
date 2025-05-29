gradle.projectsLoaded {
  allprojects {
    afterEvaluate { project ->
      if (!project.hasProperty("android")) return

      def androidExt = project.android

      // 1. compileOptions
      androidExt.compileOptions {
        if (!sourceCompatibility) sourceCompatibility = JavaVersion.VERSION_1_8
        if (!targetCompatibility) targetCompatibility = JavaVersion.VERSION_1_8
      }

      // 2. kotlinOptions
      if (project.plugins.hasPlugin("kotlin-android")) {
        androidExt.kotlinOptions {
          if (!delegate.hasProperty("jvmTarget") || !jvmTarget) {
            def jvmTargetStr = androidExt.compileOptions.targetCompatibility.toString()
              .replace("VERSION_", "")
              .replace("_", ".")
            jvmTarget = jvmTargetStr
            println "✅ ${project.name}: ustawiono kotlinOptions.jvmTarget = $jvmTarget"
          }
        }
      }
    }
  }
}
