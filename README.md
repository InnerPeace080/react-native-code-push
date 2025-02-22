# React Native Plugin for CodePush

This plugin provides client-side integration for the [CodePush service](https://microsoft.github.io/code-push), allowing you to easily add a dynamic update experience to your React Native app(s).

* [How does it work?](#how-does-it-work)
* [Supported React Native Platforms](#supported-react-native-platforms)
* [Getting Started](#getting-started)
* [iOS Setup](#ios-setup)
    * [Plugin Installation](#plugin-installation-ios)
    * [Plugin Configuration](#plugin-configuration-ios)
* [Android Setup](#android-setup)
    * [Plugin Installation](#plugin-installation-android)
    * [Plugin Configuration](#plugin-configuration-android)
* [Plugin Usage](#plugin-usage)
* [Releasing Updates (JavaScript-only)](#releasing-updates-javascript-only)
* [Releasing Updates (JavaScript + images)](#releasing-updates-javascript--images)
* [API Reference](#api-reference)

## How does it work?

A React Native app is composed of JavaScript files and any accompanying [images](https://facebook.github.io/react-native/docs/images.html#content), which are bundled together by the [packager](https://github.com/facebook/react-native/tree/master/packager) and distributed as part of a platform-specific binary (i.e. an `.ipa` or `.apk` file). Once the app is released, updating either the JavaScript code (e.g. making bug fixes, adding new features) or image assets, requires you to recompile and redistribute the entire binary, which of course, includes any review time associated with the store(s) you are publishing to.

The CodePush plugin helps get product improvements in front of your end-users instantly, by keeping your JavaScript and images synchronized with updates you release to the CodePush server. This way, your app gets the benefits of an offline mobile experience, as well as the "web-like" agility of side-loading updates as soon as they are available. It's a win-win!

*Note: Any product changes which touch native code (e.g. modifying your `AppDelegate.m`/`MainActivity.java` file, adding a new plugin) cannot be distributed via CodePush, and therefore, must be updated via the appropriate store(s).*

## Supported React Native platforms

- iOS
- Android

*Note: CodePush v1.3.0 requires v0.14.0+ of React Native, and CodePush v1.4.0 requires v0.15.0+ of React Native, so make sure you are using the right version of the CodePush plugin.*

## Getting Started

Once you've followed the general-purpose ["getting started"](http://microsoft.github.io/code-push/docs/getting-started.html) instructions for setting up your CodePush account, you can start CodePush-ifying your React Native app by running the following command from within your app's root directory:

```
npm install --save react-native-code-push
```

As with all other React Native plugins, the integration experience is different for iOS and Android, so perform the following setup steps depending on which platform(s) you are targetting.

## iOS Setup

Once you've acquired the CodePush plugin, you need to integrate it into the Xcode project of your React Native app. To do this, take the following steps:

### Plugin Installation (iOS)

1. Open your app's Xcode project
2. Find the `CodePush.xcodeproj` file within the `node_modules/react-native-code-push` directory, and drag it into the `Libraries` node in Xcode

    ![Add CodePush to project](https://cloud.githubusercontent.com/assets/516559/10322414/7688748e-6c32-11e5-83c1-00d3e6758df4.png)

3. Select the project node in Xcode and select the "Build Phases" tab of your project configuration.
4. Drag `libCodePush.a` from `Libraries/CodePush.xcodeproj/Products` into the "Link Binary With Libraries" secton of your project's "Build Phases" configuration.

    ![Link CodePush during build](https://cloud.githubusercontent.com/assets/516559/10322221/a75ea066-6c31-11e5-9d88-ff6f6a4d6968.png)

5. Click the plus sign underneath the "Link Binary With Libraries" list and select the `libz.tbd` library underneath the `iOS 9.1` node. 

    ![Libz reference](https://cloud.githubusercontent.com/assets/116461/11605042/6f786e64-9aaa-11e5-8ca7-14b852f808b1.png)
    
    *Note: Alternatively, if you prefer, you can add the `-lz` flag to the `Other Linker Flags` field in the `Linking` section of the `Build Settings`.*
    
6. Under the "Build Settings" tab of your project configuration, find the "Header Search Paths" section and edit the value.
Add a new value, `$(SRCROOT)/../node_modules/react-native-code-push` and select "recursive" in the dropdown.

    ![Add CodePush library reference](https://cloud.githubusercontent.com/assets/516559/10322038/b8157962-6c30-11e5-9264-494d65fd2626.png)

### Plugin Configuration (iOS)

Once your Xcode project has been setup to build/link the CodePush plugin, you need to configure your app to consult CodePush for the location of your JS bundle, since it is responsible for synchronizing it with updates that are released to the CodePush server. To do this, perform the following steps:

1. Open up the `AppDelegate.m` file, and add an import statement for the CodePush headers:

    ```
    #import "CodePush.h"
    ```

2. Find the following line of code, which loads your JS Bundle from the app binary for production releases:

    ```
    jsCodeLocation = [[NSBundle mainBundle] URLForResource:@"main" withExtension:@"jsbundle"];
    ```
    
3. Replace it with this line:

    ```
    jsCodeLocation = [CodePush bundleURL];
    ```

This change configures your app to always load the most recent version of your app's JS bundle. On the first launch, this will correspond to the file that was compiled with the app. However, after an update has been pushed via CodePush, this will return the location of the most recently installed update.

*NOTE: The `bundleURL` method assumes your app's JS bundle is named `main.jsbundle`. If you have configured your app to use a different file name, simply call the `bundleURLForResource:` method (which assumes you're using the `.jsbundle` extension) or `bundleURLForResource:withExtension:` method instead, in order to overwrite that default behavior*

To let the CodePush runtime know which deployment it should query for updates against, perform the following steps:

1. Open your app's `Info.plist` file and add a new entry named `CodePushDeploymentKey`, whose value is the key of the deployment you want to configure this app against (e.g. the key for the `Staging` deployment for the `FooBar` app). You can retrieve this value by running `code-push deployment ls <appName>` in the CodePush CLI, and copying the value of the `Deployment Key` column which corresponds to the deployment you want to use (see below). Note that using the deployment's name (e.g. Staging) will not work. That "friendly name" is intended only for authenticated management usage from the CLI, and not for public consumption within your app.

    ![Deployment list](https://cloud.githubusercontent.com/assets/116461/11601733/13011d5e-9a8a-11e5-9ce2-b100498ffb34.png)
    
2. In your app's `Info.plist` make sure your `CFBundleShortVersionString` value is a valid [semver](http://semver.org/) version (e.g. 1.0.0 not 1.0)

## Android Setup

In order to integrate CodePush into your Android project, perform the following steps:

### Plugin Installation (Android)

1. In your `android/settings.gradle` file, make the following additions:
    
    ```gradle
    include ':app', ':react-native-code-push'
    project(':react-native-code-push').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-code-push/android/app')
    ```
2. In your `android/app/build.gradle` file, add the `:react-native-code-push` project as a compile-time dependency:
    
    ```gradle
    ...
    dependencies {
        ...
        compile project(':react-native-code-push')
    }
    ```
    
### Plugin Configuration (Android)

After installing the plugin and syncing your Android Studio project with Gradle, you need to configure your app to consult CodePush for the location of your JS bundle, since it will "take control" of managing the current and all future versions. To do this, perform the following steps:

1. Update the `MainActivity.java` file to use CodePush via the following changes:
    
    ```java
    ...
    // 1. Import the plugin class
    import com.microsoft.codepush.react.CodePush;
    
    // 2. Optional: extend FragmentActivity if you intend to show a dialog prompting users about updates.
    public class MainActivity extends FragmentActivity implements DefaultHardwareBackBtnHandler {
        ...
        
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            ...
            // 3. Initialize CodePush with your deployment key and an instance of your MainActivity.
            CodePush codePush = new CodePush("d73bf5d8-4fbd-4e55-a837-accd328a21ba", this);
            ...
            mReactInstanceManager = ReactInstanceManager.builder()
                    .setApplication(getApplication())
                    ...
                    // 4. DELETE THIS LINE --> .setBundleAssetName("index.android.bundle")
                    
                    // 5. Let CodePush determine which location to load the most updated bundle from.
                    // If there is no updated bundle from CodePush, the location will be the assets
                    // folder with the name of the bundle passed in, e.g. index.android.bundle
                    .setJSBundleFile(codePush.getBundleUrl("index.android.bundle"))
                    
                    // 6. Expose the CodePush module to JavaScript.
                    .addPackage(codePush.getReactPackage())
                    ...
        }
    }
    ```

2. Ensure that the `android.defaultConfig.versionName` property in your `android/app/build.gradle` file is set to a semver compliant value (i.e. "1.0.0" not "1.0")
    
    ```gradle
    android {
        ...
        defaultConfig {
            ...
            versionName "1.0.0"
            ...
        }
        ...
    }
    ```

## Plugin Usage

With the CodePush plugin downloaded and linked, and your app asking CodePush where to get the right JS bundle from, the only thing left is to add the necessary code to your app to control the following policies:

1. When (and how often) to check for an update? (e.g. app start, in response to clicking a button in a settings page, periodically at some fixed interval)
2. When an update is available, how to present it to the end-user?

The simplest way to do this is to perform the following in your app's root component:

1. Import the JavaScript module for CodePush:

    ```
    var CodePush = require("react-native-code-push")
    ```

2. Call the `sync` method from within the `componentDidMount` lifecycle event, to initiate a background update on each app start:

    ```
    CodePush.sync();
    ```

If an update is available, it will be silently downloaded, and installed the next time the app is restarted (either explicitly by the end-user or by the OS), which ensures the least invasive experience for your end-users. If you would like to display a confirmation dialog (an "active install"), or customize the update experience in any way, refer to the `sync` method's [API reference](#codepushsync) for information on how to tweak this default behavior.

## Releasing Updates (JavaScript-only)

Once your app has been configured and distributed to your users, and you've made some JS changes, it's time to release it to them instantly! If you're not using React Native to bundle images (via `require('./foo.png')`), then perform the following steps. Otherwise, skip to the next section instead:

1. Execute `react-native bundle` (passing the appropriate parameters) in order to generate the updated JS bundle for your app. You can place this file wherever you want via the `--bundle-output` flag, since the exact location isn't relevant for CodePush purposes.

2. Execute `code-push release <appName> <jsBundleFilePath> <appStoreVersion> --deploymentName <deploymentName>` in order to publish the generated JS bundle to the server. The `<jsBundleFilePath>` parameter should equal the value you provided to the `--bundle-output` flag in step #1. Additionally, the `<appStoreVersion>` parameter should equal the exact app store version (i.e. the semver version end-users would see when installing it) you want this CodePush update to target.

Example Usage:

```
react-native bundle --platform ios --entry-file index.ios.js --bundle-output codepush.js
code-push release MyApp codepush.js 1.0.2
```
    
And that's it! for more information regarding the CLI and how the release (or promote and rollback) commands work, refer to the [documentation](http://microsoft.github.io/code-push/docs/cli.html).

*Note: Instead of running `react-native bundle`, you could rely on running `xcodebuild` and/or `gradlew assemble` instead to generate the JavaScript bundle file, but that would be unneccessarily doing a native build, when all you need for distributing updates via CodePush is your JavaScript bundle.*

## Releasing Updates (JavaScript + images)

If you are using the new React Native [assets system](https://facebook.github.io/react-native/docs/images.html#content), as opposed to loading your images from the network and/or platform-specific mechanisms (e.g. iOS asset catalogs), then you can't simply pass your jsbundle to CodePush as demonstrated above. You need to provide your images as well. To do this, simply use the following workflow:

1. When calling `react-native bundle`, specify that your assets and JS bundle go into a new "release" folder (you can call this anything, but it shouldn't contain any other files). For example:

    ```
    react-native bundle \
    --platform ios \
    --entry-file index.ios.js \
    --bundle-output ./release/main.jsbundle \
    --assets-dest ./release
    ```

2. Execute `code-push release`, passing the path to the directory you used in #1 as the "package" parameter (e.g. `code-push release Foo ./release 1.0.0`). The code-push CLI will automatically handle zipping up the contents for you, so don't worry about handling that yourself.

Additionally, the CodePush client supports differential updates, so even though you are releasing your JS bundle and assets on every update, your end-users will only actually download the files they need. The service handles this automatically so that you can focus on creating awesome apps and we can worry about optimizing end-user downloads.

*Note: Releasing assets via CodePush is currently only supported on iOS, and requires that you're using React Native v0.15.0+.*

---

## API Reference

When you require `react-native-code-push`, the module object provides the following top-level methods:

* [checkForUpdate](#codepushcheckforupdate): Asks the CodePush service whether the configured app deployment has an update available. 

* [getCurrentPackage](#codepushgetcurrentpackage): Retrieves the metadata about the currently installed update (e.g. description, installation time, size).

* [notifyApplicationReady](#codepushnotifyapplicationready): Notifies the CodePush runtime that an installed update is considered successful. This is an optional API, but is useful when you want to expicitly enable "rollback protection" in the event that an exception occurs in code that you've deployed to production. 

* [restartApp](#codepushrestartapp): Immediately restarts the app. If there is an update pending, it will be immediately displayed to the end-user, and the rollback timer (if specified when installing the update) will begin. Otherwise, calling this method simply has the same behavior as the end-user killing and restarting the process.

* [sync](#codepushsync): Allows checking for an update, downloading it and installing it, all with a single call. Unless you need custom UI and/or behavior, we recommend most developers to use this method when integrating CodePush into their apps

### codePush.checkForUpdate

```javascript
codePush.checkForUpdate(deploymentKey: String = null): Promise<RemotePackage>;
```

Queries the CodePush service to see whether the configured app deployment has an update available. By default, it will use the deployment key that is configured in your `Info.plist` file (iOS), or `MainActivity.java` file (Android), but you can override that by specifying a value via the optional `deploymentKey` parameter. This can be useful when you want to dynamically "redirect" a user to a specific deployment, such as allowing "Early access" via an easter egg or a user setting switch.

This method returns a `Promise` which resolves to one of two possible values:

* `null` if there is no update available.
* A `RemotePackage` instance which represents an available update that can be inspected and/or subsequently downloaded.

Example Usage: 

```javascript
codePush.checkForUpdate()
.then((update) => {
    if (!update) {
        console.log("The app is up to date!"); 
    } else {
        console.log("An update is available! Should we download it?");
    }
});
```

### codePush.getCurrentPackage

```javascript
codePush.getCurrentPackage(): Promise<LocalPackage>;
```

Retrieves the metadata about the currently installed "package" (e.g. description, installation time). This can be useful for scenarios such as displaying a "what's new?" dialog after an update has been applied.

This method returns a `Promise` which resolves to the `LocalPackage` instance that represents the currently running update. 

Example Usage: 

```javascript
codePush.getCurrentPackage()
.then((update) => {
    // If the current app "session" represents the first time
    // this update has run, and it had a description provided
    // with it upon release, let's show it to the end-user
    if (update.isFirstRun && update.description) {
        // Display a "what's new?" modal
    }
});
```

### codePush.notifyApplicationReady

```javascript
codePush.notifyApplicationReady(): Promise<void>;
```

Notifies the CodePush runtime that a freshly installed update should be considered successful, and therefore, an automatic client-side rollback isn't necessary. Calling this function is required whenever the `rollbackTimeout` parameter is specified when calling either ```LocalPackage.install``` or `sync`. If you specify a `rollbackTimeout`, and don't call `notifyApplicationReady`, the CodePush runtime will assume that the installed update has failed and roll back to the previous version. This behavior exists to help ensure that your end-users aren't blocked by a broken update.

If the `rollbackTimeout` parameter was not specified, the CodePush runtime will not enforce any automatic rollback behavior, and therefore, calling this function is not required and will result in a no-op.

If you are using the `sync` function, and doing your update check on app start, then you don't need to manually call `notifyApplicationReady` since `sync` will call it for you. This behavior exists due to the assumption that the point at which `sync` is called in your app represents a good approximation of a successful startup.

### codePush.restartApp		
		
```javascript		
codePush.restartApp(): void;		
```		
		
Immediately restarts the app. If there is an update pending, it will be presented to the end-user and the rollback timer (if specified when installing the update) will begin. Otherwise, calling this method simply has the same behavior as the end-user killing and restarting the process. This method is for advanced scenarios, and is primarily useful when the following conditions are true:		
		
1. Your app is specifying an install mode value of `ON_NEXT_RESTART` or `ON_NEXT_RESUME` when calling the `sync` or `LocalPackage.install` methods. This has the effect of not applying your update until the app has been restarted (by either the end-user or OS)	or resumed, and therefore, the update won't be immediately displayed to the end-user 	.
2. You have an app-specific user event (e.g. the end-user navigated back to the app's home route) that allows you to apply the update in an unobtrusive way, and potentially gets the update in front of the end-user sooner then waiting until the next restart or resume.		

### codePush.sync

```javascript
codePush.sync(options: Object, syncStatusChangeCallback: function(syncStatus: Number), downloadProgressCallback: function(progress: DownloadProgress)): Promise<Number>;
```

Synchronizes your app's JavaScript bundle and image assets with the latest release to the configured deployment. Unlike the `checkForUpdate` method, which simply checks for the presence of an update, and let's you control what to do next, `sync` handles the update check, download and installation experience for you.

This method provides support for two different (but customizable) "modes" to easily enable apps with different requirements:

1. **Silent mode** *(the default behavior)*, which automatically downloads available updates, and applies them the next time the app restarts. This way, the entire update experience is "silent" to the end-user, since they don't see any update prompt and/or "synthetic" app restarts.

2. **Active mode**, which when an update is available, prompts the end-user for permission before downloading it, and then immediately applies the update. If an update was released using the `mandatory` flag, the end-user would still be notified about the update, but they wouldn't have the choice to ignore it.


Example Usage: 

```javascript
// Fully silent update which keeps the app in
// sync with the server, without ever 
// interrupting the end-user
codePush.sync();

// Active update, which lets the end-user know
// about each update, and displays it to them
// immediately after downloading it
codePush.sync({ updateDialog: true, installMode: codePush.InstallMode.IMMEDIATE });
```

*Note: If you want to decide whether you check and/or download an available update based on the end-user's device battery level, network conditions, etc. then simply wrap the call to `sync` in a condition that ensures you only call it when desired.*

While the `sync` method tries to make it easy to perform silent and active updates with little configuration, it accepts an "options" object that allows you to customize numerous aspects of the default behavior mentioned above:

* __deploymentKey__ *(String)* - Specifies the deployment key you want to query for an update against. By default, this value is derived from the `Info.plist` file (iOS) and `MainActivity.java` file (Android), but this option allows you to override it from the script-side if you need to dynamically use a different deployment for a specific call to `sync`.

* __installMode__ *(CodePush.InstallMode)* - Indicates when you would like to "install" the update after downloading it, which includes reloading the JS bundle in order for any changes to take affect. Defaults to `CodePush.InstallMode.ON_NEXT_RESTART`. Refer to the [`InstallMode`](#installmode) enum reference for a description of the available options and what they do.

* __rollbackTimeout__ *(Number)* - The number of milliseconds that you want the runtime to wait after an update has been installed before considering it failed and rolling it back. Defaults to `0`, which disables rollback protection. Note that if you set this parameter to a non-zero number, and are calling `sync` during app start, you don't need to call `notifyApplicationReady`. However, if you're calling `sync` from a non-app start scenario (e.g. a button click), then you need to manually call `notifyApplicationReady` when specifying a rollback timeout.

* __updateDialog__ *(UpdateDialogOptions)* - An "options" object used to determine whether a confirmation dialog should be displayed to the end-user when an update is available, and if so, what strings to use. Defaults to `null`, which has the effect of disabling the dialog completely. Setting this to any truthy value will enable the dialog with the default strings, and passing an object to this parameter allows enabling the dialog as well as overriding one or more of the default strings. The following list represents the available options and their defaults:
    * __appendReleaseDescription__ *(Boolean)* - Indicates whether you would like to append the description of an available release to the notification message which is displayed to the end-user. Defaults to `false`.
    
    * __descriptionPrefix__ *(String)* - Indicates the string you would like to prefix the release description with, if any, when displaying the update notification to the end-user. Defaults to `" Description: "`
    
    * __mandatoryContinueButtonLabel__ *(String)* - The text to use for the button the end-user must press in order to install a mandatory update. Defaults to `"Continue"`.
    
    * __mandatoryUpdateMessage__ *(String)* - The text used as the body of an update notification, when the update is specified as mandatory. Defaults to `"An update is available that must be installed."`.
    
    * __optionalIgnoreButtonLabel__ *(String)* - The text to use for the button the end-user can press in order to ignore an optional update that is available. Defaults to `"Ignore"`.
    
    * __optionalInstallButtonLabel__ *(String)* - The text to use for the button the end-user can press in order to install an optional update. Defaults to `"Install"`.
    
    * __optionalUpdateMessage__ *(String)* - The text used as the body of an update notification, when the update is optional. Defaults to `"An update is available. Would you like to install it?"`.
    
    * __title__ *(String)* - The text used as the header of an update notification that is displayed to the end-user. Defaults to `"Update available"`.

Example Usage:

```javascript
// Use a different deployment key for this
// specific call, instead of the one configured
// in the Info.plist file
codePush.sync({ deploymentKey: "KEY" });

// Download the update silently
// but install is on the next resume
// instead of waiting until the app is restarted
codePush.sync({ installMode: codePush.InstallMode.ON_NEXT_RESUME });

// Changing the title displayed in the
// confirmation dialog of an "active" update
codePush.sync({ updateDialog: { title: "An update is available!" } });
```

In addition to the options, the `sync` method also accepts two optional function parameters which allow you to subscribe to the lifecycle of the `sync` "pipeline" in order to display additional UI as needed (e.g. a "checking for update modal or a download progress modal):

* __syncStatusChangedCallback__ *((syncStatus: Number) => void)* - Called when the sync process moves from one stage to another in the overall update process. The method is called with a status code which represents the current state, and can be any of the [`SyncStatus`](#syncstatus) values.

* __downloadProgressCallback__ *((progress: DownloadProgress) => void)* - Called periodically when an available update is being downloaded from the CodePush server. The method is called with a `DownloadProgress` object, which contains the following two properties:
    * __totalBytes__ *(Number)* - The total number of bytes expected to be received for this update package
    * __receivedBytes__ *(Number)* - The number of bytes downloaded thus far.

Example Usage:

```javascript
// Prompt the user when an update is available
// and then display a "downloading" modal 
codePush.sync({ updateDialog: true }, (status) => {
    switch (status) {
        case CodePush.SyncStatus.DOWNLOADING_PACKAGE:
            // Show "downloading" modal
            break;
        case CodePush.SyncStatus.INSTALLING_UPDATE:
            // Hide "downloading" modal
            break;
    }
});
```

This method returns a `Promise` which is resolved to a `SyncStatus` code that indicates why the `sync` call succeeded. This code can be one of the following `SyncStatus` values:

* __CodePush.SyncStatus.UP_TO_DATE__ *(4)* - The app is up-to-date with the CodePush server.
* __CodePush.SyncStatus.UPDATE_IGNORED__ *(5)* - The app had an optional update which the end-user chose to ignore. (This is only applicable when the `updateDialog` is used)
* __CodePush.SyncStatus.UPDATE_INSTALLED__ *(6)* - The update has been installed and will be run either immediately after the `syncStatusChangedCallback` function returns or the next time the app resumes/restarts, depending on the `InstallMode` specified in `SyncOptions`.

If the update check and/or the subsequent download fails for any reason, the `Promise` object returned by `sync` will be rejected with the reason.

The `sync` method can be called anywhere you'd like to check for an update. That could be in the `componentWillMount` lifecycle event of your root component, the onPress handler of a `<TouchableHighlight>` component, in the callback of a periodic timer, or whatever else makes sense for your needs. Just like the `checkForUpdate` method, it will perform the network request to check for an update in the background, so it won't impact your UI thread and/or JavaScript thread's responsiveness.

### Package objects

The `checkForUpdate` and `getCurrentPackage` methods return promises, that when resolved, provide acces to "package" objects. The package represents your code update as well as any extra metadata (e.g. description, mandatory?). The CodePush API has the distinction between the following types of packages:

* [LocalPackage](#localpackage): Represents a downloaded update package that is either already running, or has been installed and is pending an app restart.
* [RemotePackage](#remotepackage): Represents an available update on the CodePush server that hasn't been downloaded yet.

#### LocalPackage

Contains details about an update that has been downloaded locally or already installed. You can get a reference to an instance of this object either by calling the module-level `getCurrentPackage` method, or as the value of the promise returned by the `RemotePackage.download` method.

##### Properties
- __appVersion__: The app binary version that this update is dependent on. This is the value that was specified via the `appStoreVersion` parameter when calling the CLI's `release` command. *(String)*
- __deploymentKey__: The deployment key that was used to originally download this update. *(String)*
- __description__: The description of the update. This is the same value that you specified in the CLI when you released the update. *(String)*
- __failedInstall__: Indicates whether this update has been previously installed but was rolled back. The `sync` method will automatically ignore updates which have previously failed, so you only need to worry about this property if using `checkForUpdate`. *(Boolean)*
- __isFirstRun__: Indicates whether this is the first time the update has been run after being installed. This is useful for determining whether you would like to show a "What's New?" UI to the end-user after installing an update. *(Boolean)*
- __isMandatory__: Indicates whether the update is considered mandatory.  This is the value that was specified in the CLI when the update was released. *(Boolean)*
- __label__: The internal label automatically given to the update by the CodePush server. This value uniquely identifies the update within it's deployment. *(String)*
- __packageHash__: The SHA hash value of the update. *(String)*
- __packageSize__: The size of the code contained within the update, in bytes. *(Number)*

##### Methods
- __install(rollbackTimeout: Number = 0, installMode: CodePush.InstallMode = CodePush.InstallMode.ON_NEXT_RESTART): Promise&lt;void&gt;__: Installs the update by saving it to the location on disk where the runtime expects to find the latest version of the app. The `installMode` parameter controls when the changes are actually presented to the end-user. The default value is to wait until the next app restart to display the changes, but you can refer to the [`InstallMode`](#installmode) enum reference for a description of the available options and what they do.
<br /><br />
If a value greater than zero is provided to the `rollbackTimeout` parameter, the application will wait for the `notifyApplicationReady` method to be called for the given number of milliseconds before considering the update failed and automatically rolling back to the previous version.
<br /><br />
*Note: The "rollback timer" doesn't start until the update has actually become active. If the `installMode` is `IMMEDIATE`, then the rollback timer will also start immediately. However, if the `installMode` is `UPDATE_ON_RESTART` or `UPDATE_ON_RESUME`, then the rollback timer will start the next time the app starts or resumes, not at the point that you called `install`.*

#### RemotePackage

Contains details about an update that is available for download from the CodePush server. You get a reference to an instance of this object by calling the `checkForUpdate` method when an update is available. If you are using the `sync` API, you don't need to worry about the `RemotePackage`, since it will handle the download and installation process automatically for you.

##### Properties

The `RemotePackage` inherits all of the same properties as the `LocalPackage`, but includes one additional one:

- __downloadUrl__: The URL at which the package is available for download. This property is only needed for advanced usage, since the `download` method will automatically handle the acquisition of updates for you. *(String)*

##### Methods
- __download(downloadProgressCallback?: Function): Promise&lt;LocalPackage&gt;__: Downloads the available update from the CodePush service. If a `downloadProgressCallback` is specified, it will be called periodically with a `DownloadProgress` object (`{ totalBytes: Number, receivedBytes: Number }`) that reports the progress of the download until it completes. Returns a Promise that resolves with the `LocalPackage`.

### Enums

The CodePush API includes the following enums which can be used to customize the update experience:

#### InstallMode

This enum specified when you would like an installed update to actually be applied, and can be passed to either the `sync` or `LocalPackage.install` methods. It includes the following values:
* __CodePush.InstallMode.IMMEDIATE__ *(0)* - Indicates that you want to install the update and restart the app immediately. This value is appropriate for debugging scenarios as well as when displaying an update prompt to the user, since they would expect to see the changes immediately after accepting the installation.

* __CodePush.InstallMode.ON_NEXT_RESTART__ *(1)* - Indicates that you want to install the update, but not forcibly restart the app. When the app is "naturally" restarted (due the OS or end-user killing it), the update will be seamlessly picked up. This value is appropriate when performing silent updates, since it would likely be disruptive to the end-user if the app suddenly restarted out of nowhere, since they wouldn't have realized an update was even downloaded. This is the default mode used for both the `sync` and `LocalPackage.install` methods.
 
* __CodePush.InstallMode.ON_NEXT_RESUME__ *(2)* - Indicates that you want to install the update, but don't want to restart the app until the next time the end-user resumes it from the background. This way, you don't disrupt their current session, but you can get the update in front of them sooner then having to wait for the next natural restart. This value is appropriate for silent installs that can be applied on resume in a non-invasive way.

#### SyncStatus

This enum is provided to the `syncStatusChangedCallback` function that can be passed to the `sync` method, in order to hook into the overall update process. It includes the following values:

* __CodePush.SyncStatus.CHECKING_FOR_UPDATE__ *(0)* - The CodePush server is being queried for an update.
* __CodePush.SyncStatus.AWAITING_USER_ACTION__ *(1)* - An update is available, and a confirmation dialog was shown to the end-user. (This is only applicable when the `updateDialog` is used)
* __CodePush.SyncStatus.DOWNLOADING_PACKAGE__ *(2)* - An available update is being downloaded from the CodePush server.
* __CodePush.SyncStatus.INSTALLING_UPDATE__ *(3)* - An available update was downloaded and is about to be installed.
* __CodePush.SyncStatus.UP_TO_DATE__ *(4)* - The app is fully up-to-date with the configured deployment.
* __CodePush.SyncStatus.UPDATE_IGNORED__ *(5)* - The app has an optional update, which the end-user chose to ignore. (This is only applicable when the `updateDialog` is used)
* __CodePush.SyncStatus.UPDATE_INSTALLED__ *(6)* - An available update has been installed and will be run either immediately after the `syncStatusChangedCallback` function returns or the next time the app resumes/restarts, depending on the `InstallMode` specified in `SyncOptions`.
* __CodePush.SyncStatus.UNKNOWN_ERROR__ *(-1)* - The sync operation encountered an unknown error. 
