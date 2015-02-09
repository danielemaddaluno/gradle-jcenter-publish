**Publish an aar file to jCenter with Gradle:**
----------------------------------------------------------------------

I'm going to list how to upload an Android libray to jCenter and then to the Maven Central:


1. I use "Android Studio" and I have this simple android lib that I would like to be available on maven: [Android-Update-Checker](https://github.com/danielemaddaluno/Android-Update-Checker)

2. In the library folder(module) I have the lib code abovementioned. And applying in the build.gradle of this folder `apply plugin: 'com.android.library'` i got as output an .aar in the build/outputs/aar/ directory of the module's directory

3. Register to [Sonatype](https://issues.sonatype.org/secure/Dashboard.jspa), i registered with username `danielemaddaluno`

4. In the [Sonatype OSS Repository](https://issues.sonatype.org/secure/Dashboard.jspa) I registered a project opening a new Issue:<br>
`Create → Create Issue → Community Support - Open Source Project Repository Hosting → New Project) with groupid com.github.danielemaddaluno`<br>
Remember that "only one JIRA issue per top-level groupId is necessary. You have all the necessary permissions to deploy any new artifacts to this groupId or any sub-groups".

5. Register to Bintray with a `username`, the one used by me is: `danielemaddaluno`

6. Enable the automatically signing of the uploaded content:<br>
from Bintray [profile url](https://bintray.com/profile/edit) → GPG Signing → copy paste your gpg armored `public` and `private` keys.<br>
You can find respectively these two keys in files `public_key_sender.asc` and `private_key_sender.asc`
if you execute the following code (the `-a` or `--armor` option in `gpg`is used to generate ASCII-armored key pair):
  ``` bash
  gpg --gen-key
  gpg -a --export daniele.maddaluno@gmail.com > public_key_sender.asc
  gpg -a --export-secret-key daniele.maddaluno@gmail.com > private_key_sender.asc
  ```

7. In the same [web page](https://bintray.com/profile/edit) you can configure the auto-signing from:<br>
`Repositories → Maven → Check the "GPG Sign uploaded files automatically" → Update`

8. In the same [web page](https://bintray.com/profile/edit) you can find your `Bintray API Key` (simply copy it for a later use)

9. In the same [web page](https://bintray.com/profile/edit) you can configure your `Sonatype OSS User` (created at previous step 3)

10. Add these two lines to the `build.gradle` in the root of your project
  ``` gradle
  classpath 'com.github.dcendents:android-maven-plugin:1.2'
  classpath "com.jfrog.bintray.gradle:gradle-bintray-plugin:1.1"
  ```
So that your `build.gradle` in the root looks like this:
  ``` gradle
  buildscript {
      repositories {
          jcenter()
      }
      dependencies {
          classpath 'com.android.tools.build:gradle:1.0.0'
          classpath 'com.github.dcendents:android-maven-plugin:1.2'
          classpath "com.jfrog.bintray.gradle:gradle-bintray-plugin:1.1"
      }
  }
  
  allprojects {
      repositories {
          jcenter()
      }
  }
  ```
11. Modify the `build.gradle` located inside the library folder.<br>
  I modified mine from this:
  ``` gradle
  apply plugin: 'com.android.library'
  
  android {
      compileSdkVersion 21
      buildToolsVersion "21.0.0"
  
      defaultConfig {
          //applicationId "com.madx.updatechecker.lib"
          minSdkVersion 8
          targetSdkVersion 21
          versionCode 1
          versionName "1.0.0"
      }
      buildTypes {
          release {
              minifyEnabled false
              proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
          }
      }
  }
  
  dependencies {
      compile fileTree(dir: 'libs', include: ['*.jar'])
      compile 'org.jsoup:jsoup:+'
  }
  ```
  To this one:


  ``` gradle
  apply plugin: 'com.android.library'
  apply plugin: 'com.github.dcendents.android-maven'
  apply plugin: "com.jfrog.bintray"
  
  // This is the library version used when deploying the artifact
  version = "1.0.0"
  
  android {
      compileSdkVersion 21
      buildToolsVersion "21.1.2"
  
      defaultConfig {
          //applicationId "com.madx.updatechecker.lib"
          minSdkVersion 8
          targetSdkVersion 21
          versionCode 1
          versionName "1.0.0"
      }
      buildTypes {
          release {
              minifyEnabled false
              proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
          }
      }
  }
  
  dependencies {
      compile fileTree(dir: 'libs', include: ['*.jar'])
      compile 'org.jsoup:jsoup:1.8.1'
  }
  
  
  def siteUrl = 'https://github.com/danielemaddaluno/Android-Update-Checker'      // Homepage URL of the library
  def gitUrl = 'https://github.com/danielemaddaluno/Android-Update-Checker.git'   // Git repository URL
  group = "com.github.danielemaddaluno.androidupdatechecker"                      // Maven Group ID for the artifact
  
  
  install {
      repositories.mavenInstaller {
          // This generates POM.xml with proper parameters
          pom {
              project {
                  packaging 'aar'
  
                  // Add your description here
                  name 'Android Update Checker'
                  url siteUrl
  
                  // Set your license
                  licenses {
                      license {
                          name 'The Apache Software License, Version 2.0'
                          url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                      }
                  }
                  developers {
                      developer {
                          id 'danielemaddaluno'
                          name 'Daniele Maddaluno'
                          email 'daniele.maddaluno@gmail.com'
                      }
                  }
                  scm {
                      connection gitUrl
                      developerConnection gitUrl
                      url siteUrl
  
                  }
              }
          }
      }
  }
  
  task sourcesJar(type: Jar) {
      from android.sourceSets.main.java.srcDirs
      classifier = 'sources'
  }
  
  task javadoc(type: Javadoc) {
      source = android.sourceSets.main.java.srcDirs
      classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
  }
  
  task javadocJar(type: Jar, dependsOn: javadoc) {
      classifier = 'javadoc'
      from javadoc.destinationDir
  }
  artifacts {
      archives javadocJar
      archives sourcesJar
  }
  
  Properties properties = new Properties()
  properties.load(project.rootProject.file('local.properties').newDataInputStream())
  
  // https://github.com/bintray/gradle-bintray-plugin
  bintray {
      user = properties.getProperty("bintray.user")
      key = properties.getProperty("bintray.apikey")
  
      configurations = ['archives']
      pkg {
          repo = "maven"
          // it is the name that appears in bintray when logged
          name = "androidupdatechecker"
          websiteUrl = siteUrl
          vcsUrl = gitUrl
          licenses = ["Apache-2.0"]
          publish = true
  //        version {
  //            gpg {
  //                sign = true //Determines whether to GPG sign the files. The default is false
  //                passphrase = 'bintray.gpg.password' //Optional. The passphrase for GPG signing'
  //            }
  //            mavenCentralSync {
  //                sync = true //Optional (true by default). Determines whether to sync the version to Maven Central.
  //                user = 'bintray.oss.user' //OSS user token
  //                password = 'bintray.oss.password' //OSS user password
  //                close = '1' //Optional property. By default the staging repository is closed and artifacts are released to Maven Central. You can optionally turn this behaviour off (by puting 0 as value) and release the version manually.
  //            }
  //        }
      }
  }
  ```

12. Add to the `local.properties` in the root of the project the following lines (remember that this file should never be uploaded on your public repository):
  ``` gradle
  bintray.user=<your bintray username>
  bintray.apikey=<your bintray api key>
  
  bintray.gpg.password=<your gpg signing password>
  bintray.oss.user=<your sonatype username>
  bintray.oss.password=<your sonatype password>
  ```

13. Added to my PATH the default gradle 2.2.1 actually used by "Android Studio", for example:
  ``` bash
  sudo gedit .bashrc
  ```
  Add to the bottom of file `.bashrc` the following line:<br> `PATH=$PATH:/etc/android-studio/gradle/gradle-2.2.1/bin`
  
14. Open "Android Studio" terminal and execute:
  ``` bash
  gradle bintrayUpload
  ```

15. From [Bintray](https://bintray.com/) → My Recent Packages → androidupdatechecker (this is here only after the execution of the previous point 14 ) → Add to Jcenter → Check the box → Group Id = "com.github.danielemaddaluno.androidupdatechecker".

16. Finally Sync all with Maven Central following: Bintray → My Recent Packages → androidupdatechecker → Maven Central → Sync.

17. Now your library should be automatically imported both from Maven Central and from Bintray, you can import with something like this:
  **Automatically with Gradle**
  ``` gradle
  dependencies {
      repositories {
          maven {
              url 'http://dl.bintray.com/danielemaddaluno/maven/'
          }
      }
      compile 'com.github.danielemaddaluno.androidupdatechecker:library:1.0.0'
  }
  ```
  **Automatically with Maven**
  ``` xml
  
  <project>
  ...
      <repositories>
          <repository>
              <id>danielemaddaluno</id>
              <name>Daniele Maddaluno Bintray Repository</name>
              <url>http://dl.bintray.com/danielemaddaluno/maven/</url>
          </repository>
      </repositories>
      ...
      <dependecies>
          <dependency>
              <groupId>com.github.danielemaddaluno.androidupdatechecker</groupId>
              <artifactId>library</artifactId>
              <version>1.0.0</version>
          </dependency>
      </dependecies>
  ...
  </project>
  ```
