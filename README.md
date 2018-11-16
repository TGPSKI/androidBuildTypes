# Android + React Native + Fastlane: Working with multiple build types

This is the companion repository for [my medium post](https://medium.com/@tgpski/android-react-native-fastlane-working-with-multiple-build-types-a9a6641c5704).


## Check out the **[React Native DevOps Guide!!!](https://medium.com/@tgpski/react-native-devops-guide-2d8e4755ebee)**

The DevOps Guide includes concept discussion and step by step walkthroughs for setting up your own Jenkins agent and running iOS and Android builds, plus a whole lot more.

[Running Android Builds, Part 4 - React Native DevOps Guide](https://medium.com/@tgpski/running-android-builds-part-4-react-native-devops-guide-ddc36c12061
)

---------------


## Gradle Configurations

### Global gradle.properties - defining node executable

```sh
# This file lives in ~/.gradle/gradle.properties

MYAPP_RELEASE_STORE_FILE=foo.keystore
MYAPP_RELEASE_KEY_ALIAS=foo-alias
MYAPP_RELEASE_STORE_PASSWORD=foo_store_release
MYAPP_RELEASE_KEY_PASSWORD=foo_key_release

org.gradle.daemon=true

NODE_PATH=[YOUR_HOME_PATH]/.nvm/versions/node/v[YOUR_NODE_VERSION]/bin/node
```

### React ExtraPropertiesExtension

```java
project.ext.react = [
    entryFile: "index.js",
    nodeExecutableAndArgs: hasProperty('NODE_PATH')?[NODE_PATH]:["node"],
    bundleInDebug: false,
    bundleInStaging: true,
    bundleInRelease: true,
    jsBundleDirDebug: "$buildDir/intermediates/assets/debug",
    resourcesDirDebug: "$buildDir/intermediates/res/merged/debug",
    jsBundleDirStaging: "$buildDir/intermediates/assets/staging",
    resourcesDirStaging: "$buildDir/intermediates/res/merged/staging",
    jsBundleDirRelease: "$buildDir/intermediates/assets/release",
    resourcesDirRelease: "$buildDir/intermediates/res/merged/release",
    devDisabledInDebug: false,
    devDisabledInStaging: true,
    devDisabledInRelease: true,
    inputExcludes: ["ios/**", "__tests__/**"]
];
```

### new buildType, applicationIdSuffix, signingConfigs

```java
buildTypes {
  ...
  staging {
      minifyEnabled enableProguardInReleaseBuilds
      proguardFiles getDefaultProguardFile("proguard-android.txt"), "proguard-rules.pro"
      signingConfig signingConfigs.release
      applicationIdSuffix ".staging"
  }
```

## Fastlane Actions

### Yarn install

```ruby
yarn(
  command: "install",
  package_path: "./package.json"
)
```

### Android versioning

```ruby
ANDROID_VERSION_NAME = get_version_name(
  gradle_file_path:"android/app/build.gradle",
  ext_constant_name:"versionName"
)
ANDROID_VERSION_CODE = get_version_code(
  gradle_file_path:"android/app/build.gradle",
  ext_constant_name:"versionCode"
)
```

### Adding badges

```ruby
if options[:badge]
    add_badge(
      shield: "#{ANDROID_VERSION_NAME}-#{ANDROID_VERSION_CODE}-orange",
      glob: "/android/app/src/main/res/mipmap-*/ic_launcher.png",
      alpha: true,
      shield_scale: "0.75"
    )
  end
```

### Upload to Crashlytics

```ruby
if ENV["CRASHLYTICS_API_KEY"] && ENV["CRASHLYTICS_BUILD_SECRET"]
  crashlytics(
    api_token: ENV["CRASHLYTICS_API_KEY"],
    build_secret: ENV["CRASHLYTICS_BUILD_SECRET"],
    groups: ["your-test-groups-here"],
    notes: "#{ANDROID_VERSION_NAME}-#{ANDROID_VERSION_CODE} beta release - fastlane generated",
    notifications: true
  )
end
```

### Upload to Play Store

```ruby
if options[:play_store]
  # upload to alpha track on google play developer's console
  supply(track: "alpha", apk: "android/app/build/outputs/apk/app-release.apk")
end
```
