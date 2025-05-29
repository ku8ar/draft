gradle.projectsLoaded {
  allprojects {
    afterEvaluate { project ->
      if (!project.hasProperty("android")) return

      def androidExt = project.android

      // 1. Ustaw compileOptions, jeśli brak
      if (!androidExt.hasProperty("compileOptions") || !androidExt.compileOptions?.sourceCompatibility) {
        androidExt.compileOptions {
          sourceCompatibility = JavaVersion.VERSION_1_8
          targetCompatibility = JavaVersion.VERSION_1_8
        }
        println "✅ ${project.name}: ustawiono compileOptions na Java 1.8"
      }

      // 2. Ustaw kotlinOptions.jvmTarget = compileOptions.targetCompatibility
      if (project.plugins.hasPlugin("kotlin-android")) {
        def targetJava = androidExt.compileOptions?.targetCompatibility ?: JavaVersion.VERSION_1_8
        def jvmTargetStr = targetJava.toString().replace("VERSION_", "") // np. VERSION_1_8 → 1_8
                                                  .replace("_", ".")     // → 1.8

        androidExt.kotlinOptions {
          if (!delegate.hasProperty("jvmTarget") || !jvmTarget) {
            jvmTarget = jvmTargetStr
            println "✅ ${project.name}: ustawiono kotlinOptions.jvmTarget = $jvmTarget"
          }
        }
      }
    }
  }
}
