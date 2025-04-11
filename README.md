gradle.beforeProject {
    project.repositories.all {
        removeIf { repo ->
            repo instanceof MavenArtifactRepository && (
                repo.url.toString().contains("repo.maven.apache.org") ||
                repo.url.toString().contains("jcenter") ||
                repo.url.toString().contains("dl.google.com")
            )
        }
        // Dodaj własny Nexus ponownie, jeśli usunięto
    }
}
