![cirrus-logo](https://user-images.githubusercontent.com/12459/33736599-70d3255e-db50-11e7-80db-74e5b3899b77.png)

# CirrusMDSDK

CirrusMDSDK is an embeddable SDK. It enables customers of CirrusMD to provide the CirrusMD patient chat experience in their own applications. While the example application will work in a sandboxed environment when built, integrating CirrusMDSDK into your own application will require you to be a CirrusMD customer. Integration requires a unique `secret` and SSO `token` to work correctly. Please contact your CirrusMD account representative for more information.

![chat](https://user-images.githubusercontent.com/12459/33736363-9149bb00-db4f-11e7-89d6-50151ca1761f.png)

- [Example Application](#example-application)
- [Screen Shots](https://github.com/CirrusMD/CirrusMD-iOS-SDK-Example/wiki/Screenshots)
- [Requirements](#requirements)
- [Installation](#installing-cirrusmdsdk-in-your-own-project)
- [Basic Usage](#basic-usage)
- [Advanced Usage](#advanced-usage)
  - [Logout](#logout)
  - [Custom Status Views](#custom-status-views)
  - [Push notifications](#push-notifications)
  - [Additional API](#additional-api)
- [License](#license)

## Example Application

To run the example project, clone the repo, and run `pod install` from the Example directory first.

## Requirements

- iOS 9.0+
- CirrusMDSDK works in both Swift and Objective-C projects
- `Build Settings > Build Options > Always Embed Swift Standard Libraries` must be set to `Yes`

## Installing CirrusMDSDK in your own project

The CirrusMDSDK installs as a prebuilt framework, `CirrusMDSDK.framework`.

### CocoaPods

[CocoaPods](http://cocoapods.org) is a dependency manager for Cocoa projects. You can install it with the following command:

```bash
$ gem install cocoapods
```

To integrate CirrusMDSDK into your Xcode project using CocoaPods, specify it in your `Podfile`:

```ruby
source 'https://github.com/CocoaPods/Specs.git'
source 'https://github.com/CirrusMD/podspecs.git' # <-- Make sure to add this line

platform :ios, '11.0' # must be >= 9.0
use_frameworks!

target '<Your Target Name>' do
    pod 'CirrusMDSDK'
end
```

Then, run the following command:

```bash
$ pod install
```

### Carthage

[Carthage](https://github.com/Carthage/Carthage) is a decentralized dependency manager that builds your dependencies and provides you with binary frameworks.

You can install Carthage with [Homebrew](http://brew.sh/) using the following command:

```bash
$ brew update
$ brew install carthage
```

To integrate CirrusMDSDK into your Xcode project using Carthage, specify it in your `Cartfile`:

```ogdl
github "CirrusMD/CirrusMD-iOS-SDK-Example"
```

Run `carthage update` to build the framework and drag the built `CirrusMDSDK.framework` into your Xcode project.

Follow the instructions found on [Carthage](https://github.com/Carthage/Carthage) to add both an Input File and Output File in your `Build Phase` for Carthage.

Add an Import Path to your build settings:
> `Build Settings > Import Paths > Carthage/Build/iOS/CirrusMDSDK.framework/zlib`


### Manually

If you prefer not to use any of the aforementioned dependency managers, you can integrate CirrusMDSDK into your project manually.

#### NEED TO DOCUMENT THE STEPS FOR INTEGRATING IT MANUALLY HERE

## Basic Usage

Basic usage of of the CirrusMDSDK is very simple.
1. import CirrusMDSDK
1. retrieve a token via SSO
1. set the CirrusMD provided secret on the SDK
1. set the SSO provided token on the SDK
1. ask the SDK for a messages controller and display it

### The details

1. import CirrusMDSDK

#### Swift

```swift
import CirrusMDSDK
```

#### Objective-C

```obj-c
#import <CirrusMDSDK/CirrusMDSDK-Swift.h>
```

2. Our team works with your technical staff to provide SSO for your patients using the CirrusMD platform. The CirrusMDSDK uses tokens retrieved via SSO from CirrusMD's SSO service. Each SSO integration is slightly customized based on your needs. In general, your backend service requests a token representing a patient from our SSO service which provides the token that should be set on the SDK.

*** **_Do not cache CirrusMD provided JWTs_** ***

3. Set the CirrusMD provided secret. The secret is unique to your organization. To receive a valid client secret contact your account representative at CirrusMD. The secret must be set prior to setting the token in the next step. You must also set set the secret after logging out of the SDK.

#### Swift

```swift
CirrusMDSDKSession.singleton.setSecret("CIRRUSMD-PROVIDED-SECRET")
```

#### Objective-C

```obj-c
[CirrusMDSDKSession.singleton setSecret: @"CIRRUSMD-PROVIDED-SECRET"];
```

4. **_AFTER_** setting the secret on the SDK, set the token. This is your opportunity to verify that the token provided to the SDK is able to load a patient.

#### Swift

```swift
/*
    The completion closure is retained throughout the life of the SDK. It is called with
    `.success` one time after the token is set if successful.

    The completion closure is called with `.invalidToken`, `.noSecretProvided` or
    `.serviceUnavailable` every time the status changes.
*/

CirrusMDSDKSession.singleton.setToken("RETRIEVED_TOKEN") { result in
    switch result {
    case .success:
        // The token is valid and the patient's profile loaded successfully.
        // Showing a MessageViewController will load properly.

    case .invalidToken:
        // The token is expired, incomplete, invalid and/or the patient was
        // unable to load. You must retrieve a new token before proceeding.

    case .noSecretProvided:
        // No secret has been set on the SDK. Set the secret and retry
        // setting the token.

    case .serviceUnavailable:
        // CirrusMD servers are unreachable.

    }
}
```

#### Objective-C

```obj-c
/*
    The completion block is retained throughout the life of the SDK. It is called with
    `CirrusMDSDKSessionResultSuccess` one time after the token is set if successful.

    The completion block is called with `CirrusMDSDKSessionResultInvalidToken`,
    `CirrusMDSDKSessionResultNoSecretProvided` or `CirrusMDSDKSessionResultServiceUnavailable`
    every time the status changes.
*/

[CirrusMDSDKSession.singleton setToken:"RETRIEVED_TOKEN"
                            completion:^(CirrusMDSDKSessionResult result)
{
    switch (result) {
        case CirrusMDSDKSessionResultSuccess:
            // The token is valid and the patient's profile loaded successfully.
            // Showing a MessageViewController will load properly.

        case CirrusMDSDKSessionResultInvalidToken:
            // The token is expired, incomplete, invalid and/or the patient was
            // unable to load. You must retrieve a new token before proceeding.

      case CirrusMDSDKSessionResultNoSecretProvided:
          // No secret has been set on the SDK. Set the secret and retry
          // setting the token.

        case CirrusMDSDKSessionResultServiceUnavailable:
            // CirrusMD servers are unreachable

    }
}];
```

5. Retrieve a `MessageViewController` and present it.

#### Swift

```swift
let controller = CirrusMDSDKSession.singleton.messageViewController()

self.navigationController?.pushViewController(controller, animated: true)

// or

self.present(controller, animated: true) {...}
```

#### Objective-C

```obj-c
UIViewController *controller = [CirrusMDSDKSession.singleton messageViewController];

[self.navigationController pushViewController:controller animated:YES];

// or

[self presentViewController:controller animated:YES completion:^{...}];
```

If the SDK has been provided with a valid secret and token the SSO user's messages will display in the retrieved view controller. A user's first look at their messages will look something like this:

![welcome-message](https://user-images.githubusercontent.com/12459/33579479-304386aa-d907-11e7-99df-b5b163196fd7.png)

## Advanced Usage

### Logout

You may wish to log the user out of the SDK when they sign out of your application. Logging the user out destroys the associated CirrusMD server session and unregisters the device from CirrusMD delivered push notifications if previously registred.

#### Swift

```swift
CirrusMDSDKSession.singleton.logOut()
```

#### Objective-C

```obj-c
[CirrusMDSDKSession.singleton logout];
```


### Custom Status Views

Ideally, your patients always see a working messages view when you present a `MessagesViewController`. However, there are certain times when we're unable to show messages.

The first is when you have explicity called `logout()`. We recommend calling logout, _only_ when your patient logs out of your appication. In that case they will not see the _logged out view_ because they will be logged out of your application as well. You should log them back into the SDK when they next log back into your application.

The second is when the SDK is unable to verify the secret and/or token. In either case, an _error view_  is shown. We recommend you handle all errors passed to the last argument of `CirrusMDSDKSession.singleton.setToken`'s completion handler prior to showing the `MessagesViewController` if possible. Doing so will provide a better experience for your user. Some errors may happen after the `MessagesViewController` is already on screen. In that case, _error view_ is displayed.

Two screens displayed by the SDK have default values that can be customized. The _logged out view_ and _error view_. We strongly recommend that you provide your own custom views for both cases. Because the CirrusMDSDK uses SSO to authenticate your patients, we are unable to provide logged out UI that helps the patient log back in. By providing your patients with a custom _logout out view_ you can, for example, provide relevant messaging and a button to log back in using the same SSO you implemented to log them in originally. Every time the _error view_ is shown the resolution is retrieving a new SSO token and setting it via `CirrusMDSDKSession.singleton.setToken(:)`. Providing a custom _error view_ gives you the ability to display relevant messaging and interactions the user can take, most likely a button to re-attempt SSO.

By default they will look similar to the screens below:

The default logged out screen is shown after you call `CirrusMDSDKSession.singleton.logOut()`.

![logged-out](https://user-images.githubusercontent.com/12459/33667547-605ac368-da5a-11e7-8abc-bfd6457b6ee0.png)

The error screen can be shown for several reasons, such as providing an expired token or invalid secret.

![error-message](https://user-images.githubusercontent.com/12459/33579485-365a9a60-d907-11e7-85e8-603060b0ac1e.png)

Customization of both the _logged out view_ and _error view_ happens via the `CirrusMDSDKSession.delegate`.

1. Implement `CirrusMDSKSessionDelegate`
2. Provide custom views for any or all of the optional delegates.
3. Set a frame on the view you provide and it will be centered in the yellow area of the screen below.

![customized-message](https://user-images.githubusercontent.com/12459/33580171-6152b174-d909-11e7-90a6-ff551369382f.png)

#### Swift

```swift
CirrusMDSDKSession.singleton.delegate = self

func viewForError(code: CirrusMDSDKSessionResult) -> UIView {
    // return a custom logged out view
}
func viewForLoggedOut() -> UIView {
    // return a custom view appropriate for the CirrusMDSDKSessionResult
}
```

#### Objective-C

```obj-c
CirrusMDSDKSession.singleton.delegate = self;

- (UIView *)viewForLoggedOut {
    // return a custom logged out view
}

- (UIView *)viewForErrorWithCode:(enum CirrusMDSDKSessionResult)code {
    // return a custom view appropriate for the CirrusMDSDKSessionResult
}
```

### Push notifications
In order to enable push notifications for your patients you'll need to provide CirrusMD with the APNS certificate used for the Bundle Identifier associated with your application. Contact your account representative at CirrusMD to enable CirrusMD push notification delivery.

#### Registering for remote notifications
**_AFTER_** providing CirrusMD with your APNS Certificate, register for push notifications.


#### Swift

```swift
let settings = UIUserNotificationSettings(types: [.alert, .sound, .badge], categories: nil)
UIApplication.shared.registerUserNotificationSettings(settings)
UIApplication.shared.registerForRemoteNotifications()
```

#### Objective-C

```obj-c
UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:
  (UIUserNotificationTypeAlert | UIUserNotificationTypeBadge | UIUserNotificationTypeSound) categories:nil];
[[UIApplication sharedApplication] registerUserNotificationSettings:settings];
[[UIApplication sharedApplication] registerForRemoteNotifications];
```

Use the `deviceToken` provided by Apple when [application(_:didRegisterForRemoteNotificationsWithDeviceToken:))](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622958-application) is called.

#### Swift

```swift
/*
    `deviceToken` is the deviceToken provided by Apple when
    `application(
      _ application: UIApplication,
      didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data
    )` is called.
*/

CirrusMDSDKSession.singleton.registerforRemoteNotifications(deviceToken)
```

#### Objective-C

```obj-c
/*
    `deviceToken` is the deviceToken provided by Apple when
                                `- (void)application:(UIApplication *)application
    didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken` is called.
*/

[CirrusMDSDKSession.singleton registerforRemoteNotifications: deviceToken];
```

#### Unregistering for remote notifications

#### Swift

```swift
CirrusMDSDKSession.singleton.unregisterforRemoteNotifications()
```

#### Objective-C

```obj-c
[CirrusMDSDKSession.singleton unregisterforRemoteNotifications];
```

It is up to the host application to handle incoming push notifications appropriately. The payload of the `userInfo` dictionary passed to [application(_:didReceiveRemoteNotification:)](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623117-application) or the newer [application(_:didReceiveRemoteNotification:fetchCompletionHandler:)](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623013-application) has the following shape.

```json
{
  "aps" : {
    "alert" : "You have a new message.",
    "badge" : 1,
    "sound" : "default"
  },
  "custom_data" : {
    "event" : "message:new",
    "owner_id" : 1234,
    "stream_id" : "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  }
}
```

After inspecting the incoming payload and finding `custom_data.event.message:new` you may want to display the `UIViewController` provided by `CirrusMDSDKSession.singleton.messageViewController()`.

### Additional API

Access `CirrusMDSDKSession.singleton.tokenState` for the state of the token. The possible values are `invalid`, `valid`, and `unknown`. The `tokenState` can be used to troubleshoot issues with the SDK, `invalid` and `unknown` require retrieving a new token from the server. `unknown` usually indicates that a token has not yet been set.

#### Swift

```swift
let tokenState = CirrusMDSDKSession.singleton.tokenState

switch tokenState {
case .invalid:
case .unknown:
case .valid
}
```

#### Objective-C

```obj-c
CirrusMDSKSessionTokenState tokenState = CirrusMDSDKSession.singleton.tokenState;

switch (tokenState) {
    case CirrusMDSKSessionTokenStateInvalid:
    case CirrusMDSKSessionTokenStateUnknown:
    case CirrusMDSKSessionTokenStateValid:
}
```

## Author

CirrusMD

## License

CirrusMD-iOS-SDK-Example is available under the MIT license. See the LICENSE file for more info.

CirrusMDSDK is available under the XXXX license. See the LICENSE file for more info.
