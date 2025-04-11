def localPropertiesFile = new File(rootDir, "local.properties")
def nexusUsername = ""
def nexusPassword = ""

if (localPropertiesFile.exists()) {
    def props = new Properties()
    props.load(new FileInputStream(localPropertiesFile))
    nexusUsername = props.getProperty("nexusUsername") ?: ""
    nexusPassword = props.getProperty("nexusPassword") ?: ""
}
