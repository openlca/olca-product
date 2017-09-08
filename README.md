## Preface
The purpose of this re-factoring is to enable product building *in one click*. I have forked from *Master* branch, so all that had worked there should work the same - I change nothing in Java code, just worked with pom.xml-files, product and target-platform definitions.

## What was done
1. Eclipse Tycho and Eclipse PDE share the common concept of the `target.platform`, but with the clause that Tycho supports only remote URLs as P2 repository. For that reason I have created a number of own P2 repositories for deprecated versions of Eclipse.org projects (like GEF, Equinox Executor). I didn't include Babel_eclipse - because it has very-very many dependencies to resolve. Also, Tycho can automatically choose native bundles (depending on OS) if product configuration is based on features (not on plugin-ins) - so, I redefined [product configuration](olca.product)
2. `olca-modules` and `olca-updates` are changed to version 1.7.0-SNAPSHOT
3. `olca-app-feature` is parent module for `olca-app` and the Eclipse feature embracing it.
4. The new design supports *duality*, i.e. you can run and build application both in PDE and Tycho. To build in PDE - projects from `olca-target-runtime`, `olca-app-feature`, `olca-product` should be imported in Eclipse IDE. For Tycho-building the current version of `olca-modules` and `olca-updates` artifacts should be already deployed in [our Maven Snapshot repository](http://ec2-54-90-248-145.compute-1.amazonaws.com:8081/nexus/) or locally installed.


#### How to build OpenLCA application
##### The easiest way
- assuming olca-modules and ocla-updates are already deployed on Maven snapshot repo
- assuming olca-app-feature is already deployed on Maven snapshot repo
- Java 8 installed

```
git clone https://github.com/openlca/olca-product.git
cd olca-proudct
mvnw
```

&mdash; under the hood it fetches everything from `download.eclipse.org` sites, our P2 and our Maven snapshot repositoriy.
Then open `olca-product\target\products\org.openlca.olca-app.lcaproduct\win32\win32` and choose your `arch` (x86 or
x86_64), depending on what Java (32-bit or 64-bit) is installed on your computer. Double-click on `OpenLCA.exe`.


##### Work in Eclipse IDE
Please, follow the instruction very thoroughly! I assume that olca-modules and olca-updates are *just libraries that developed independently* and they are already installed either locally (~/.m2) or deployed on Maven repo.

Your Eclipse IDE - *SHOULD* be PDE + e(fx)clipse installed, search Eclipse Marketplace for the latter one ([see also](https://stackoverflow.com/questions/22812488/using-javafx-in-jre-8)).

1. Import `target.platform`:
    - `git clone https://github.com/openlca/olca-target-platform.git`
    - import it into Eclipse IDE as Maven -> Existing Maven projects
    - when project is in Eclipse IDE double click `olca-target-runtime.target`
    - wait while Eclipse is resolving target definition
    - in the editor there is a link `Set as Target Platform` &mdash; click on it! From now all that will be imported to Eclipse IDE is using it for resolving dependencies.
3. `olca-app-feature` - in terminal do:
    - `git clone https://github.com/openlca/olca-app-feature.git`
    - `cd olca-app-feature`
    - `mvn` - will download embedded libraries from maven repo, [generates MANIFEST.MF](manifest-generation.md) file and builds+zips html-pages for `olca-app-feature/olca-app` - there is no need for `ocla-app-html` anymore. This I would call the initialization step&mdash;the reason is that `olca-app/pom.xml` is declared tycho's `eclipse-plugin` and it should be valid when invoked in maven build&mdash;but it is not true&mdash;we don't store embedded libraries and `html_zip` in Git. So, we do all this operation in the parent `olca-app-feature` POM, to avoid invoking Tycho while `olca-app` doesn't have valid META-INF/MANIFEST.MF file.
    - In Eclipse IDE: Import -> General -> [Existing Projects Maven Project] - check all 3 projects
4. Import `olca-product`
    - `git clone https://github.com/openlca/olca-product.git`
    - Import -> Maven -> Existing Maven projects
5. Now you SHOULD be ready to start OLCA from Eclipse IDE
    - double-click on `olca-product/olca.product`- the product definition editor will open
    - Press on `Synchronize` button in editor' view
    - then `Launch an Eclipse application`

When OpenLCA started from IDE - an exception may appear in console:

```
java.lang.ClassNotFoundException: org.eclipse.ui.internal.ide.application.addons.ModelCleanupAddon cannot be found
```

&mdash; maybe ignored as they are specific to IDE-run and not in real app.
