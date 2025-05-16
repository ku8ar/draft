dependencies {
    implementation(name: 'nazwa-bez-.aar', ext: 'aar') {
        // wyłącza lint analizę dla tego aar
        attributes {
            attribute(Attribute.of("lint", String), "disable")
        }
    }
}
