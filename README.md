## Preface
The purpose of this re-factoring is to enable product building *in one click*. Before that the application was indeed a *plain single jar* within Eclipse RCP, where all modules and working libraries resided. The previous solution didn't use the most powerful approach provided by Eclipse RCP out of the box &ndash; *OSGi*. The downsides of the previous design were:

- long workflow (for example when you make amendments in some `olca-modules`, then you had to install it's Maven artifact and then copy to `lib` directory of `olca-app`
- inability to update just a part of bundle

## What was done
1. Eclipse Tycho and Eclipse PDE share the common concept of the `target.platform`, but with the clause that Tycho supports only remote URLs as P2 repository. For that reason I have created a [P2 site for artifacts that are not hosted on Eclipse sites](https://olca-s3-repo.s3-eu-west-1.amazonaws.com) - the P2 repository is generated by [olca-repo](https://github.com/denis-kalinin/olca-repo) project.
2. [OSGi-fied GeoTools](https://github.com/denis-kalinin/olca-geo): `gt-all` and `gt-swt`
3. OSGi-fied artifacts in [olca-modules](https://github.com/denis-kalinin/olca-modules/tree/tycho-releng)
4. grouping `olca-modules` in Eclipse Feature ([olca.feature.core](https://github.com/denis-kalinin/olca-modules/tree/tycho-releng/olca-core-feature))
5. grouping `olca-app` and `olca-updates` in Eclipse Feature ([olca.feature.app](https://github.com/denis-kalinin/olca-app-feature)). *NB!* to keep current Git structure clear I created feature's project in separate repository, so they MUST be cloned into the same parent folder into the folders `olca-app`, `olca-updates`, `olca-app-feature` respectively - as they are defined in the [pom.xml](https://github.com/denis-kalinin/olca-app-feature/blob/master/pom.xml#L115) file.
6. The new design supports *duality*, i.e. you can run and build application both in PDE and Tycho. To build in PDE - projects from `olca-target-runtime`, `olca-models`, `olca-app`, `olca-updates`, `olca-app-feature` should be imported in Eclipse IDE. For Tycho-building the current version of artifacts from the above list should be deployed in [our Maven Snapshot repository](http://ec2-54-90-248-145.compute-1.amazonaws.com:8081/nexus/)


#### How to build OpenLCA application
##### The easiest way
```
git clone https://github.com/denis-kalinin/olca-product.git
cd olca-proudct
mvn
```
&mdash; under the hood it fetches everything from `download.eclipse.org` sites, our P2 and our Maven snapshot repositories.
Then open `olca-product\target\products\org.openlca.olca-app.lcaproduct\win32\win32` and choose your `arch` (x86 or
x86_64), depending on what Java (32-bit or 64-bit) is installed on your computer. Double-click on `OpenLCA.exe`.

The application start but is not fully-fledged - a class-loading issue with EclipseLink JPA, because we are in OSGi now -  it can be resolved with some Java code adjustments and [Eclipse Gemini JPA project](https://www.eclipse.org/gemini/jpa/). There are also possible class-loading issues with Jython and GeoTools that may arise after fixing JPA.  


##### Work in Eclipse IDE
Please, follow the instruction very thoroughly!

1. Import `target.platform`:
    - `git clone https://github.com/denis-kalinin/olca-target-platform.git`
    - import it into Eclipse IDE as Maven -> Existing Maven projects
    - when project is in Eclipse IDE double click `olca-target-runtime.target`
    - wait while Eclipse is resolving target definition
    - in the editor there is a link `Set as Target Platform` &mdash; click on it! From now all that will be imported to Eclipse IDE is using it for resolving dependencies.
2. Import core modules:
    - `git clone -b tycho-releng https://github.com/denis-kalinin/olca-modules.git`
    - `cd olca-modules/olca-core`
    - `mvn dependency:copy` - to download some jars that, alas for now, are to be embedded in some artifacts (EclipseLink)
    - Import -> General -> [Existing Projects into Workspace](docs/olca-modules.md)
3. Import `olca-updates`
    - `git clone -b tycho-releng https://github.com/denis-kalinin/olca-updates.git`
    - `cd olca-updates`
    - `mvn dependency:copy` - to download embedded dependency - jython-standalone.jar
    - Import -> General -> Existing Projects into Workspace
4. Import `olca-app`
    - `git clone -b tycho-releng https://github.com/denis-kalinin/olca-app.git`
    - `cd olca-app/olca-app`
    - `mvn dependency:copy` - to download and embed jython-standalone.jar
    - Import -> General -> [Existing Projects into Workspace](docs/olca-app.md)
5. HTML content for `olca-app` from `olca-app-html`
   - [follow instructions to generate and copy HTML files](https://github.com/denis-kalinin/olca-app/tree/master/olca-app-html)
6. Import `olca-app-feature`
    - `git clone https://github.com/denis-kalinin/olca-app-feature.git`
    - Import -> General -> Existing Projects into Workspace
7. Import `olca-product`
    - `git clone https://github.com/denis-kalinin/olca-product.git`
    - Import -> Maven -> Existing Maven projects
    - if project is marked with red, then [adjust it](docs/olca-product.md)
8. Now you SHOULD be ready to start OLCA from Eclipse IDE
    - double-click on `olca-product/olca.product`- the product definition editor will open
    - Press on `Synchronize` button in editor' view
    - then `Launch an Eclipse application`
    
###### Possible improvements
- `mvn dependency:copy` will be removed if we OSGi-fy JPA and Jython
- step 5 could be removed if HTML-build will be handled by Maven (implies some job for 2-3 days)
- steps 3-6 could be merged if GitHub repository layout is changed accordingly
- Java (JRE) can be embedded for particular OS while building with Tycho
