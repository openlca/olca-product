META-INF/MANIFEST.MF file in `olca-app` is generated by `olca-app-feature`/pom.xml Maven instruction if it doesn't exist at the moment:

- gets template from `olca-app/bundle-resources/MANIFEST.MF`
- get Maven dependencies with transitive ones to `olca-modules` and `olca-updates`
- add them to `Bundle-ClassPath` in resulting `META-INF/MANIFEST.MF`

So, if you want to manually edit MANIFEST.MF file - edit one in `olca-app/bundle-resources/MANIFEST.MF`.