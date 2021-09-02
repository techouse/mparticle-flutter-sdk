# mparticle_flutter_sdk

***Please note that this is the alpha version of the mParticle Flutter Plugin.  Public APIs are subject to change before v1.0.0 is released.***

Flutter allows developers to use a single code base to deploy to multiple platforms.  Now, with the mParticle Flutter Plugin, you can leverage a single API to deploy your data to hundreds of integrations from your iOS, Android, and Web apps.

See the table below to see what features are currently supported. The Flutter mParticle plugin is under ongoing development and the goal is to fully support all the below features on all 3 core platforms.

### Supported Features
| Method | Android | iOS | Web | Notes |
|---|---|---|---| --- |
| Custom Events | X | X | X |  |
| Page Views | X | X | X |  |
| Identity | X | X | X |  |
| eCommerce |  |  |  | Coming Soon |
| Consent |  |  |  | Coming Soon |


# Installation

1. Add the Flutter SDK as a dependency to your Flutter application:

```
flutter pub add mparticle_flutter_sdk
```

Specifying this dependency adds a line like the following to your package's `pubspec.yaml`:

```
dependencies:
    mparticle_flutter_sdk: ^0.0.1
```

2.  Import the package into your Dart code:

```
import 'package:mparticle_flutter_sdk/mparticle_flutter_sdk.dart'
```

Now that you have the mParticle Dart plugin, install mParticle on your native/web platforms.  Be sure to include an API Key and Secret where required or you will see errors in your logs when launching your app.

## <a name="Android"></a>Android

To install mParticle on an Android platform:

1. Add the following dependencies to your app's `build.gradle`:

```groovy
dependencies {
    implementation 'com.mparticle:android-core:5+'

    // Required for gathering Android Advertising ID (see below)
    implementation 'com.google.android.gms:play-services-ads-identifier:16.0.0'

    // Recommended to query the Google Play install referrer
    implementation 'com.android.installreferrer:installreferrer:1.0'
}
```

2. Grab your mParticle key and secret from [your workspace's dashboard](https://app.mparticle.com/apps) and construct an `MParticleOptions` object.

3. Call `start` from the `onCreate` method of your app's `Application` class. It's crucial that the SDK be started here for proper session management. If you don't already have an `Application` class, create it and then specify its fully-qualified name in the `<application>` tag of your app's `AndroidManifest.xml`.

```java
package com.example.myapp;

import android.app.Application;
import com.mparticle.MParticle;

public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        MParticleOptions options = MParticleOptions.builder(this)
            .credentials("REPLACE ME WITH KEY","REPLACE ME WITH SECRET")
            .setLogLevel(MParticle.LogLevel.VERBOSE)
            .identify(identifyRequest)
            .identifyTask(
                new BaseIdentityTask()
                        .addFailureListener(this)
                        .addSuccessListener(this)
                    )
            .attributionListener(this)
            .build();

        MParticle.start(options);
    }
}
```

```kotlin
import com.mparticle.MParticle
import com.mparticle.MParticleOptions

class ExampleApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        val options = MParticleOptions.builder(this)
            .credentials("REPLACE ME WITH KEY", "REPLACE ME WITH SECRET")
            .build()
        MParticle.start(options)
    }
}
```

> **Warning:** Don't log events in your `Application.onCreate()`. Android may instantiate your `Application` class for a lot of reasons, in the background, while the user isn't even using their device. 
For more help, see [the Android set up docs](https://docs.mparticle.com/developers/sdk/android/getting-started/#create-an-input).

## <a name="iOS"></a>iOS

Configuring iOS:

To install mParticle on an iOS platform:


1. Copy your mParticle key and secret** from [your app's dashboard][1].

[1]: https://app.mparticle.com/apps

2. Install the SDK using CocoaPods:

```bash
$ # Update your Podfile to depend on 'mParticle-Apple-SDK' version 8.5.0 or later
$ pod install
```

The mParticle SDK is initialized by calling the `startWithOptions` method within the `application:didFinishLaunchingWithOptions:` delegate call. Preferably the location of the initialization method call should be one of the last statements in the `application:didFinishLaunchingWithOptions:`. The `startWithOptions` method requires an options argument containing your key and secret and an initial Identity request.

> Note that you must initialize the SDK in the `application:didFinishLaunchingWithOptions:` method. Other parts of the SDK rely on the `UIApplicationDidBecomeActiveNotification` notification to function properly. Failing to start the SDK as indicated will impair it. Also, please do **not** use _GCD_'s `dispatch_async` to start the SDK.
For more help, see [the full iOS set up docs](https://docs.mparticle.com/developers/sdk/ios/getting-started/#create-an-input).

3. Import and start the mParticle Apple SDK into Swift or Objective-C.
### Swift Example

```swift
import mParticle_Apple_SDK

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        
       // Override point for customization after application launch.
        let mParticleOptions = MParticleOptions(key: "<<<App Key Here>>>", secret: "<<<App Secret Here>>>")
        
       //Please see the Identity page for more information on building this object
        let request = MPIdentityApiRequest()
        request.email = "email@example.com"
        mParticleOptions.identifyRequest = request
        mParticleOptions.onIdentifyComplete = { (apiResult, error) in
            NSLog("Identify complete. userId = %@ error = %@", apiResult?.user.userId.stringValue ?? "Null User ID", error?.localizedDescription ?? "No Error Available")
        }
        mParticleOptions.onAttributionComplete = { (attributionResult, error) in
                    NSLog(@"Attribution Complete. attributionResults = %@", attributionResult.linkInfo)
        }
        
       //Start the SDK
        MParticle.sharedInstance().start(with: mParticleOptions)
        
       return true
}
```

### Objective-C Example

For apps supporting iOS 8 and above, Apple recommends using the import syntax for **modules** or **semantic import**. However, if you prefer the traditional CocoaPods and static libraries delivery mechanism, that is fully supported as well.

If you are using mParticle as a framework, your import statement will be as follows:

```objective-c
@import mParticle_Apple_SDK;                // Apple recommended syntax, but requires "Enable Modules (C and Objective-C)" in pbxproj
#import <mParticle_Apple_SDK/mParticle.h>   // Works when modules are not enabled

```

Otherwise, for CocoaPods without `use_frameworks!`, you can use either of these statements:

```objective-c
#import <mParticle-Apple-SDK/mParticle.h>
#import "mParticle.h"
```

Next, you'll need to start the SDK:

```objective-c
- (BOOL)application:(UIApplication *)application
        didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    MParticleOptions *mParticleOptions = [MParticleOptions optionsWithKey:@"REPLACE ME"
                                                                   secret:@"REPLACE ME"];
    
    //Please see the Identity page for more information on building this object
    MPIdentityApiRequest *request = [MPIdentityApiRequest requestWithEmptyUser];
    request.email = @"email@example.com";
    mParticleOptions.identifyRequest = request;
    mParticleOptions.onIdentifyComplete = ^(MPIdentityApiResult * _Nullable apiResult, NSError * _Nullable error) {
        NSLog(@"Identify complete. userId = %@ error = %@", apiResult.user.userId, error);
    };
    mParticleOptions.onAttributionComplete(MPAttributionResult * _Nullable attributionResult, NSError * _Nullable error) {
        NSLog(@"Attribution Complete. attributionResults = %@", attributionResult.linkInfo)
    }
    
    [[MParticle sharedInstance] startWithOptions:mParticleOptions];
    
    return YES;
}
```

See [Identity](https://docs.mparticle.com/developers/sdk/ios/idsync/) for more information on supplying an `MPIdentityApiRequest` object during SDK initialization.


## <a name="Web"></a>Web


Add the mParticle snippet to your `web/index.html` file as high as possible on the page within the <head> tag, per our [Web Docs](https://docs.mparticle.com/developers/sdk/web/getting-started/).
```
<script type="text/javascript">
  //configure the SDK
  window.mParticle = {
      config: {
          isDevelopmentMode: true,
          identifyRequest: {
              userIdentities: {
                  email: 'email@example.com',
                  customerid: '123456',
              },
          },
          identityCallback: (response) {
              console.log(response);
          },
          dataPlan: {
            planId: 'my_plan_id',
            planVersion: 2
          }
      },
  };

  //load the SDK
  (
  function(t){window.mParticle=window.mParticle||{};window.mParticle.EventType={Unknown:0,Navigation:1,Location:2,Search:3,Transaction:4,UserContent:5,UserPreference:6,Social:7,Other:8};window.mParticle.eCommerce={Cart:{}};window.mParticle.Identity={};window.mParticle.config=window.mParticle.config||{};window.mParticle.config.rq=[];window.mParticle.config.snippetVersion=2.3;window.mParticle.ready=function(t){window.mParticle.config.rq.push(t)};var e=["endSession","logError","logBaseEvent","logEvent","logForm","logLink","logPageView","setSessionAttribute","setAppName","setAppVersion","setOptOut","setPosition","startNewSession","startTrackingLocation","stopTrackingLocation"];var o=["setCurrencyCode","logCheckout"];var i=["identify","login","logout","modify"];e.forEach(function(t){window.mParticle[t]=n(t)});o.forEach(function(t){window.mParticle.eCommerce[t]=n(t,"eCommerce")});i.forEach(function(t){window.mParticle.Identity[t]=n(t,"Identity")});function n(e,o){return function(){if(o){e=o+"."+e}var t=Array.prototype.slice.call(arguments);t.unshift(e);window.mParticle.config.rq.push(t)}}var dpId,dpV,config=window.mParticle.config,env=config.isDevelopmentMode?1:0,dbUrl="?env="+env,dataPlan=window.mParticle.config.dataPlan;dataPlan&&(dpId=dataPlan.planId,dpV=dataPlan.planVersion,dpId&&(dpV&&(dpV<1||dpV>1e3)&&(dpV=null),dbUrl+="&plan_id="+dpId+(dpV?"&plan_version="+dpV:"")));var mp=document.createElement("script");mp.type="text/javascript";mp.async=true;mp.src=("https:"==document.location.protocol?"https://jssdkcdns":"http://jssdkcdn")+".mparticle.com/js/v2/"+t+"/mparticle.js" + dbUrl;var c=document.getElementsByTagName("script")[0];c.parentNode.insertBefore(mp,c)}
  )("REPLACE WITH API KEY");
</script>
```
For more help, see the [full Web set up docs](https://docs.mparticle.com/developers/sdk/web/getting-started/#create-an-input).

# Usage

Each of our Dart methods is mapped to an underlying mParticle SDK at the platform level. Note that per Dart's [documentation](https://flutter.dev/docs/development/platform-integration/platform-channels#architecture, calling into platform specific code is asynchronous to ensure the user interface remains responsive.  In your code, you can swap usage between `async` and `then` in accordance to your app's requirements.

## Import

**Importing** the module:
```dart
import 'package:mparticle_flutter_sdk/mparticle_flutter_sdk.dart';
```

You must first call `getInstance` on `MparticleFlutterSdk` before each method is called.  This ensures the underlying mParticle SDK has been initialized.  Per Flutter's [plugin documentation](https://flutter.dev/docs/development/platform-integration/platform-channels),  messages between the Dart plugin and underlying platforms must be passed asynchronously to ensure the user interface remains responsive. Therefore, to ensure code is performant to your team's requirements, you may refactor instances of `await` with `then` and vice versa in the examples below.

```dart
MparticleFlutterSdk? mpInstance = await MparticleFlutterSdk.getInstance();
```

## Log Events

To log events, import mParticle `EventTypes` to write proper event logging calls:

```dart
import 'package:mparticle_flutter_sdk/events/event_type.dart';

mpInstance?.logEvent(
    eventName: 'Clicked Search Bar',
    eventType: EventType.Search,
    customAttributes: {'key1': 'value1'},
    customFlags: {'flag1': 'flagValue1'});
```


To log screen events:

```dart
mpInstance?.logScreenEvent(
    eventName: 'Screen event logged',
    customAttributes: {'key1': 'value1'},
    customFlags: {'flag1': 'flagValue1'});
}
```

## User
Get the current user in order to apply and remove attributes, tags, etc.

```dart
var user = await mpInstance?.getCurrentUser();
```

User Attributes:

```dart
user.setUserAttribute('age', '45');
```

```dart
user.setUserAttributeArray('Test key', ['Test value 1', 'Test value 2']);
```

```dart
user.setUserTag('tag1');
```

```dart
user.getUserAttributes('tag1');
```


```dart
user.removeUserAttribute('age')
```

```dart
user.getUserIdentities().then((identities) {
    print(identities); // Map<IdentityType, String>
});
```

## IdentityRequest

```dart
var identityRequest = MparticleFlutterSdk.identityRequest;
identityRequest
    .setIdentity(IdentityType.CustomerId, 'customerid5')
    .setIdentity(IdentityType.Email, 'email5@gmail.com');
```

## Identity

```dart
mpInstance.getCurrentUser((currentUser) => {
    print(currentUser?.getMPID());
});
```

Import the `SuccessResponse` and `FailureResponse` classes to write proper callbacks for Identity methods.  For brevity, we included an example of full error handling in only the `identify` example below, but this error handling can be used for any of the Identity calls.

```dart
import 'package:mparticle_flutter_sdk/identity/success_response.dart';
import 'package:mparticle_flutter_sdk/identity/failure_response.dart';

var request = MparticleFlutterSdk.identityRequest();

mpInstance?.identity
    .identify(identityRequest: identityRequest)
    .then(
        (SuccessResponse successResponse) =>
            print("Success Response: $successResponse"),
        onError: (error) {
            var failureResponse = error as FailureResponse;
            print("Failure Response: $failureResponse");

            // It is possible for either a client error or a server error to occur during identity calls.
            // First check for the client side error, then you can check the http code for the server error.
            // More details can be found in the platform specific IDSync error handling:
                // iOS - https://docs.mparticle.com/developers/sdk/ios/idsync/#error-handling
                // Web - https://docs.mparticle.com/developers/sdk/web/idsync/#error-handling
                // Android - https://docs.mparticle.com/developers/sdk/android/idsync/#idsync-status-codes
            if (failureResponse.clientErrorCode != null) {
                switch (failureResponse.clientErrorCode) {
                case IdentityClientErrorCodes.RequestInProgress:
                    // there is an Identity request in progress, wait for it to complete before attempting another
                case IdentityClientErrorCodes.ClientSideTimeout:
                case IdentityClientErrorCodes.ClientNoConnection:
                    // retry the IDSync request
                case IdentityClientErrorCodes.SSLError:
                    // SSL configuration issue. 
                case IdentityClientErrorCodes.OptOut:
                    // The user has opted out of data collection
                case IdentityClientErrorCodes.Unknown:
                    // 
                case IdentityClientErrorCodes.ActiveSession:
                case IdentityClientErrorCodes.ValidationIssue:
                    // A web error that should never arise due to Dart's stronger typing
                case IdentityClientErrorCodes.NativeIdentityRequest:
                default:
                    print(failureResponse.clientErrorCode);
                }
            }
            int? httpCode = failureResponse.httpCode;
            if (httpCode != null && httpCode >= 400) {
                switch (httpCode) {
                case 400:
                case 401:
                case 429:
                case 529:
                default:
                    failureResponse.errors.forEach(
                        (error) => print('${error.code}\n${error.message}'));
                }
            }
        }
    );
```

```dart
var identityRequest = MparticleFlutterSdk.identityRequest;
identityRequest
    .setIdentity(IdentityType.CustomerId, 'customerid2')
    .setIdentity(IdentityType.Email, 'email2@gmail.com');

mpInstance?.identity
    .login(identityRequest: identityRequest)
    .then(
        (SuccessResponse successResponse) =>
            print("Success Response: $successResponse"),
        onError: (error) {
            var failureResponse = error as FailureResponse;
            print("Failure Response: $failureResponse");
        }
    );
```

```dart
var identityRequest = MparticleFlutterSdk.identityRequest;
identityRequest
    .setIdentity(IdentityType.CustomerId, 'customerid3')
    .setIdentity(IdentityType.Email, 'email3@gmail.com');

mpInstance?.identity
    .modify(identityRequest: identityRequest)
    .then(
        (SuccessResponse successResponse) =>
            print("Success Response: $successResponse"),
        onError: (error) {
            var failureResponse = error as FailureResponse;
            print("Failure Response: $failureResponse");
        }
    );
```

```dart
var identityRequest = MparticleFlutterSdk.identityRequest;
// depending on your identity strategy, you may have identities added to your identityRequestk

mpInstance?.identity
    .logout(identityRequest: identityRequest)
    .then(
        (SuccessResponse successResponse) =>
            print("Success Response: $successResponse"),
        onError: (error) {
            var failureResponse = error as FailureResponse;
            print("Failure Response: $failureResponse");
        }
    );
```



# Native-only Methods
A few methods are currently supported only on iOS/Android SDKs:

* Get the SDK's opt out status -

    ```dart
    var isOptedOut = await mpInstance.getOptOut;
    MParticle.setOptOut(!isOptedOut);
    ```

* Check if a kit is active

    ```dart
    mpInstance.isKitActive(kitId).then((isActive) {
        print(isActive);
    });
    ```

* Push Registration

    The method `mpInstance.logPushRegistration()` accepts two parameters. For Android, provide both `pushToken` and `senderId`. For iOS, provide the push token in the first parameter, and simply pass `null` for the second parameter

    ### Android

    ```dart
    mpInstance.logPushRegistration(pushToken, senderId);
    ```

    ### iOS

    ```dart
    mpInstance.logPushRegistration(pushToken, null);
    ```


# License

Apache 2.0