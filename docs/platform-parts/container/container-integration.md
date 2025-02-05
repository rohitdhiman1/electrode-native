This section describes how to integrate an Electrode Native container in your Android or iOS mobile application.

#### Android

A Container library can be added to a mobile Android application project in one of two ways:

- By adding a dependency on the Electrode Native container AAR _(recommended way)_, or
- By directly adding the Electrode Native container module to the Android project (as a git submodule for example)

You will also need to update your `build.gradle` files with the following:

- `jcenter` repository

We publish the `react-native` Maven artifact to `jcenter`. Therefore, you must make sure that `jcenter` is present in your list of repositories. The repositories are most commonly defined in your top-level project `build.gradle`.

```groovy
repositories {
  jcenter()
  //...
}
```

- resolution strategy

React Native includes some third-party libraries that might conflict with the versions you are using. For example, you might have issues with `jsr305`. If that is the case, add the following to your application module `build.gradle`

```groovy
configurations.all {
  resolutionStrategy.force 'com.google.code.findbugs:jsr305:3.0.0'
  //...
}
```

In addition to the above resolution strategy for handling the `jsr305` conflict, you might also run into a conflict with `OkHttp`. React Native depends on a specific version of the very popular networking library, `OkHttp`. If you are using this library in your application, you might be forced to align your version of `OkHttp` with the version included with the React Native version that you are using. This is due to the current React Native design.

- okio linting

You might run into a conflict with the `okio` third party library which comes with React Native. It is a known issue. To resolve this issue, disable the lint check for `InvalidPackage`. You can also find solutions by searching for the `okio` conflict on the web.

```groovy
lintOptions {
  disable 'InvalidPackage'
  //...
}
```

##### Adding the Container as an AAR

If a Maven publisher has been configured in the Electrode Native cauldron, Electrode Native will package and publish the Electrode Native container project as a Maven artifact containing the AAR file(either to a local or remote Maven repository)

If you are implicitly publishing a container from a Cauldron (through a change in container content or the use of [cauldron regen-container] command), the Maven artifact will have the following data:

- Group ID : `com.walmartlabs.ern`
- Artifact ID : `{mobile-app-name}-ern-container`
- Version string : `{container-version}`

{mobile-app-name} is the name of the mobile application in the cauldron for the Electrode Native container that is being generated. For example, if the application name is `walmart`, the Electrode Native container artifact ID will be `walmart-ern-container`.

{container-version} is the version of the generated Electrode Native container. The container version can be in the form: `x.y.z` where x, y and z are integers. For example `1.2.3` is a valid container version. You can specify a version for a Container or, by default, the current version will be patched-bumped to the new version.

To add a dependency on the Electrode Native container, in your mobile application add the following code to the `dependencies` object of your application module `build.gradle`. Be sure to substitute the `{mobile-app-name}` and `{container-version}` to represent your application.

```groovy
dependencies {
  api 'com.walmartlabs.ern:{mobile-app-name}-ern-container:{container-version}'
  //...
}
```

If you are explicitly publishing a container through the use of [publish-container] command, the Maven artifact id will be `local-ern-container`. The group id will remain the same though `com.walmartlabs.ern`. You can pass options to the command to change the artifact id and group id at your convenience. Please see [publish-container] documentation for more details.
Also, if you use or plan to use a locally published Electrode Native container (to your maven local repository), make sure to declare `mavenLocal` in the list of repositories. This is located in the top-level project `build.gradle`.

```groovy
repositories {
  mavenLocal()
  //...
}
```

##### Proguard

Container library does not proguard itself, but supports proguarding. In your module's `proguard.txt` add the rule to be during the applications proguarding phase.

```
# keep rules for react-native-electrode-bridge
-keep class com.walmartlabs.electrode.** {*;}
```

##### Adding the container as a Git submodule

Alternatively, you can include an Electrode Native container in a mobile application by adding it as an Android module. Although this is not the recommended way to add third-party dependencies (the container being one) to an Android project, it is however possible and this might be the best process if you don't have a remote Maven repository that you can publish the container to.

To add the container library as an Android module, add a GitHub publisher to the cauldron (or use [publish-container] with the `git` publisher option). Then, when a new Container version is published, Electrode Native will publish the resulting project to a private or public GitHub repository. It will create a Git tag for each version of a container. You can then add the container Android module to your application project--managed as a Git submodule.

**Note** Do not edit the code of the container if you use this procedure even though adding the Container directly in your project makes its code editable. The container code should not be modified manually, as any custom modification will be lost the next time the container is generated.

Be sure to include the module in your project `settings.gradle`, and add a `api project` directive to your application module `build.gradle`. Find more information on [declaring API and implementation dependencies](https://docs.gradle.org/current/userguide/java_library_plugin.html)

##### Configure Android build configuration versions

The following android build parameters can be configured with application specific needs.

- `buildToolsVersion` - Android SDK build tools is a component of the Android SDK required for building Android apps. The version specified will update the app level `build.gradle`
  ```groovy
  android {
    buildToolsVersion "28.0.3"
  }
  ```

- `compileSdkVersion` - The API level designated to compile the application.

- `kotlinVersion` - The version of Kotlin to use for the container _(only used in case at least one Kotlin native module is injected in the container)_.

- `minSdkVersion` - The minimum API level that the application targets.

- `sourceCompatibility` - Defines which language version of Java your source files should be treated as. It can take a valid [Java Version](https://docs.gradle.org/current/javadoc/org/gradle/api/JavaVersion.html)

- `targetCompatibility` - Defines the minimum JVM version your code should run on, i.e. it determines the version of byte code the compiler generates. It can take a valid [Java Version](https://docs.gradle.org/current/javadoc/org/gradle/api/JavaVersion.html)

- `targetSdkVersion` - The designated API Level that the application targets

  ```groovy
    android {
      compileSdkVersion 28
      defaultConfig {
          minSdkVersion 19
          targetSdkVersion 28
      }
    }
  ```

- `supportLibraryVersion` - You may want a standard way to provide newer features on earlier versions of Android or gracefully fall back to equivalent functionality. You can leverage these libraries to provide that compatibility layer.

  ```grovy
  compile 'com.android.support:appcompat-v7:28.0.0'
  ```

You can configure `androidConfig` in the cauldron as show below.

```json
{
  "containerGenerator": {
    "androidConfig": {
      "buildToolsVersion": "28.0.3",
      "compileSdkVersion": "28",
      "minSdkVersion": "19",
      "supportLibraryVersion": "28.0.0",
      "targetSdkVersion": "28"
    }
  }
}
```

##### Android Dynamic Feature Module Support

If the Android client mobile application consuming the container is keeping the container dependency in an Android [dynamic feature module](https://developer.android.com/codelabs/on-demand-dynamic-delivery), there will be issues with resources loading _(your MiniApps images won't be visible for example)_. This is because of React Native Android implementation, that is loading some resources via reflection, using the base package name of the application, instead of the package name of the dynamic module _[see Android documentation for more details](https://developer.android.com/guide/playcore/feature-delivery#resource-uri)_. For this reason we had no way but to fork React Native to update the implementation to properly handle this use case.\
Our fork of React Native is kept in [electrode-io/react-native](https://github.com/electrode-io/react-native) repository.\
We are only using it to publish special React Native AARs for Android, not for any iOS changes nor JS ones _(i.e we're not publishing anything to npm)_.\
Starting with 0.63 line, we will publish custom releases of the AAR, in addition to the official versions, to include support for dynamic feature modules. These versions will have a patch number starting at 100 _(0.63.100, 0.64.100 ...)_.

If you are facing this fringe scenario with dynamic feature modules, here is what can be done:

1. Generate the container with a custom AAR version of React Native that includes support for Dynamic Feature Modules.\
This can be done by supplying such a configuration to the container generator _(via --extra option or through Cauldron config)_

```json
{
  "containerGenerator": {
    "androidConfig": {
      "reactNativeAarVersion": "0.64.100"
    }
  }
}
```

Always use the latest custom AAR version matching the React Native version line that your miniapps(s) are using _(for example if your miniapp is using 0.63.4, you should use 0.63.100 here)_.

2. Update the client application to pass the dynamic feature module package name to the container.\
For example if the client application base package name is `com.foo` and the dynamic feature module containing the container dependency is named `bar`, the package name used for resources resolution in the dynamic feature module would be `com.foo.bar`.\
In that case, the client application would need to call the following, prior to initializing the container.

```java
ElectrodeReactContainer.setPackageName("com.foo.bar");
```

##### JavaScript Engine (RN 0.60 and above)

Starting with React Native 0.60, the JavaScript engine is distributed separately from the React Native AAR.
Also, prior to this version, JavaScriptCore was the only JavaScript engine that could be used on Android for React Native applications. Starting with this new version, it is now possible to use alternative JavaScript engines such as Hermes or V8.

Electrode Native currently support both JavaScriptCore and Hermes engines.
By default, without explicit configuration, Electrode Native will use the non international variant of JavaScriptCore engine.

_JavaScriptCore_

With React Native 0.60.0, JavaScriptCore engine now comes in two variants : `android-jsc` and `android-jsc-intl`. The later is the international variant. It includes ICU i18n library and necessary data allowing to use e.g. Date.toLocaleString and String.localeCompare that give correct results when using with locales other than en-US. This variant is about 6MB larger per architecture.

By default, the version of JavaScriptCore used by Electrode Native will be set to the latest version available at the time of Electrode Native version release and will be communicated in the release notes. The default JavaScriptCore variant will always be the non international one.

It is possible to change these defaults, using the `androidConfig` object of `containerGenerator` as shown below.

```json
{
  "containerGenerator": {
    "androidConfig": {
      "jsEngine": "jsc",
      "jscVersion": "^245459.0.0",
      "jscVariant": "android-jsc"
    }
  }
}
```

`jscVersion` is the version (fixed or range) of the JavaScriptCore engine while `jscVariant` is the variant (`android-jsc` or `android-jsc-intl`).

_Hermes_

To use [Hermes](https://hermesengine.dev/) engine rather than JavaScriptCore, you should set the `jsEngine` in `androidConfig` to `hermes`.

```json
{
  "containerGenerator": {
    "androidConfig": {
      "jsEngine": "hermes",
      "hermesVersion": "0.2.1"
    }
  }
}
```

#### iOS

##### An Electrode Native container can be retrieved in a few ways:

- Use a dependency manager such as Carthage or Cocoapods or,
- Perform a manual `git clone` of the container

**Using CocoaPods (RN >= 0.61, XCode >= 11.0)**

If the client application is using CocoaPods to manage its dependencies, it is possible to package and distribute the container in a way that it can be added as a pod dependency *(in the Podfile)* of the client application.

One thing to note here though, is that due to the fact that the container itself is using CocoaPods and is depending on third-party pods, it is not possible *(as far as we know)* to add it 'as-is' to the client application.\
One way to work around this, is to distribute the container as a pre-compiled binary instead of as its raw source code.\
One advantage of such an approach is that it will reduce build time of the client application *(as it doesn't have to build the container during application build)*.\
One inconvenient of this approach is that the source code of the container will not be visible/accessible in the client app which can make debugging issues a bit more complex.

The high level steps are to build & package the container as an XCFramework, and to publish it along with an associated podpsec file, to a git repository. The client application can then add the container as a pod depenndency in its Podfile.

1. Build & Package the container as an XCFramework

After the container is generated *(via create-container command or other way)* the [XCFramework container transformer](1) can be used to build & package the container as an XCFramework.

One way to achieve this is to use the [transform-container] command as follow:

```
ern transform-container -p ios -t xcframework
```

This can also be achieved through cauldron by adding the following step in container generation pipeline config:

```json
{
  "name": "ern-container-transformer-xcframework",
}
```

2. Publish the container XCFramework as a pod to a git repository

After an XCFramework has been generated for the container, the [CocoaPod git publisher](2) can be used to publish the container XCFramework to a git repository, along with its associated podspec.

One way to achieve this is to use the [publish-container] command as follow:

```
ern publish-container --platform ios -p cocoapod-git -u [ssh_or_https_url_to_git_repo] -v [container_version]
```

This can also be achieved through cauldron by adding the follwing step in container generation pipeline config:

```json
{
  "name": "ern-container-publisher-cocoapod-git",
  "url": "[ssh_or_https_url_to_git_repo]",
}
```

This publisher will only upload the pre-compiled XCFramework to the repository, along with an adequatly generated ElectrodeContainer.podspec file. It will also create a git tag matching the container version.

3. Add the container pod dependency to client application Podfile

The container can then be added as a one line entry to the Podfile of the client application as follow:

```
pod 'ElectrodeContainer', :git => '[ssh_or_https_url_to_git_repo]', :tag => '[container_version]'
```

The `tag` can be omitted in case the client application prefer to always pull the latest container version from default branch of the repository.\
`:branch` can be used in place of `:tag`, to always pull the latest container from a specific branch.

Then, running `pod install` from the client application, will properly retrieve the container as a pod dependency.

**Using Carthage**

1. Create a Cartfile if you don't already have one, or open an existing Cartfile.
2. Add the following line to your Cartfile.

   ```bash
   git "git@github.com:username/myweatherapp-ios-container.git" "v1.0.0"
   ```

3. Create a `Cartfile.resolved` file if you don't have one or open your existing `Cartfile.resolved` file.
4. Add the following line to your `Cartfile.resolved` file:

   ```bash
   git "git@github.com:username/myweatherapp-ios-container.git" "v1.0.0"
   ```

5. Install your dependencies using the following command:

   ```bash
   carthage bootstrap --no-build --platform ios
   ```

**Use Git to clone container**

1. Clone the container.

   ```bash
   git clone git@github.com:username/myweatherapp-ios-container.git
   ```

##### Add Container to your mobile application

**React Native >= 0.61**

1. Check if your mobile application is using a workspace, (ie: you have an `.xcworkspace` file in your project directory). If not, open the `.xcodeproj` file of your mobile application in Xcode.
2. In Xcode's menu bar, **File** -> **Save As Workspace**... You can use the same name as the `.xcodeproj` file for your workspace name. Save at the same level as the `.xcodeproj`.
3. Close Xcode and re-open, this time selecting the workspace file.
4. Make sure nothing is selected in the project navigator, In Xcode's menu bar, **File** -> **Add Files** to `<your workspace name>`. If you used Carthage, look for `ElectrodeContainer.xcodeproj` in the Carthage/Checkouts directory. If you cloned the container, find the `ElectrodeContainer.xcodeproj` in the cloned container directory.
5. **Add Files** again, this time adding the `Pods.xcodeproj` that is located in the Pods directory of the container.

**React Native < 0.61**

1. Open your mobile application project file in Xcode.
2. Right click your `<your project name>` in the project navigator. Select **Add Files** to `<your project name>`. If you used Carthage, look for `ElectrodeContainer.xcodeproj` in the Carthage/Checkouts directory. If you cloned the container, find the `ElectrodeContainer.xcodeproj` in the cloned container directory.

**Additional Configuration**

After installing the dependency, you will need to add additional configurations.

1. In Xcode, choose `<your project name>` from the Project Navigator panel.
2. Click `<your project name>` under TARGETS.
3. From the General tab, locate **Frameworks, Libraries, and Embedded Content** and click **+**
4. Select `ElectrodeContainer.framework` and click Add.
5. In Build Phases, verify that `ElectrodeContainer` is in Link Binary With Libraries and Embed Frameworks.
6. Edit Scheme for your `<your project name>` target. Locate Build Options and uncheck Parallelize Build.

##### Skip installing dependencies (RN >= 0.61)

When using React Native >= 0.61, you can configure the iOS container generator
to skip the automatic installation of dependencies. **This container generation mode is the default one when generating iOS containers on Linux/Windows**, as the `pod install` command cannot be run on these platforms.\
To also generate such a container on MacOS, you can set the `skipInstall` flag to `true` in the container generator iOS configuration, as follow:

```json
{
  "containerGenerator": {
    "iosConfig": {
      "skipInstall": true
    }
  }
}
```

Alternatively, for containers that are not generated using a cauldron, you can set the `--skipInstall` option of the `create-container` command.

##### Extra configuration

You can override the iOS deployment target version to use by setting `iosConfig` in the cauldron as show below.

```json
{
  "containerGenerator": {
    "iosConfig": {
      "deploymentTarget": "11.0"
    }
  }
}
```

[1]: https://github.com/electrode-io/ern-container-transformer-xcframework
[2]: https://github.com/electrode-io/ern-container-publisher-cocoapod-git
[transform-container]: ../../cli/transform-container
[publish-container]: ../../cli/publish-container
