project.afterEvaluate {
  project.repositories.maven { ->
    url = URI.create("https://maven.pkg.github.com/package")
    credentials { cred ->
      cred.username = System.getenv("MAVEN_USERNAME")
      cred.password = System.getenv("MAVEN_PASSWORD")
    }
  }
}
