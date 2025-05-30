# SumUp mPOS SDK - iOS

[![Platform](https://img.shields.io/badge/Platform-iOS-lightgrey.svg?style=flat)](#prerequisites)
[![Created](https://img.shields.io/badge/Made%20by-SumUp-blue.svg?style=flat)](https://sumup.com)
[![Supports](https://img.shields.io/badge/Requires-iOS%2014+-red.svg?style=flat)]()
[![License](https://img.shields.io/badge/License-SumUp-brightgreen.svg?style=flat)](LICENSE)
[![CocoaPods](https://img.shields.io/cocoapods/v/SumUpSDK.svg?style=flat)](SumUpSDK.podspec)
[![SPM compatible](https://img.shields.io/badge/SPM-compatible-4BC51D.svg?style=flat)](Package.swift)

This repository provides a native iOS SDK that enables you to integrate SumUp's proprietary
card terminal(s) and its payment platform to accept credit and debit card payments
(incl. VISA, MasterCard, American Express and more). SumUp's SDK communicates transparently
to the card terminal(s) via Bluetooth (BLE 4.0) or an audio cable connection. Upon initiating
a checkout, the SDK guides your user using appropriate screens through each step of the payment
process. As part of the process, SumUp also provides the card terminal setup screen, along with the
cardholder signature verification screen. The checkout result is returned with the relevant
data for your records.

No sensitive card data is ever passed through to or stored on the merchant’s phone.
All data is encrypted by the card terminal, which has been fully certified to the highest
industry standards (PCI, EMV I & II, Visa, MasterCard & Amex).

For more information, please refer to
[SumUp API documentation.](https://developer.sumup.com/).

### Prerequisites
1. Registered for a merchant account via SumUp's [country websites](https://sumup.com/) (or received a test account).
2. Received SumUp card terminal: Solo, Air, Air Lite, PIN+ terminal, Chip & Signature reader, or SumUp Air Register.
3. Requested an Affiliate (Access) Key via [SumUp Dashboard](https://me.sumup.com/developers) for Developers.
4. Deployment Target iOS 14.0 or later.
5. Recommended to use on Xcode 14.3.1 and iOS SDK 16 or later.
6. iPhone, iPad or iPod touch.

### Compatibility

* Starting with firmware version 1.0.1.84, Air card readers with serial numbers starting with 108, 109 or later require iOS SDK 4.3.0 and later. Please update to the latest iOS SDK version if you need to support these readers.
* From version 4.4.0 of the SDK, iOS 14 or later is required

### Tipping

There are three modes for tipping:

1. No tipping. Leave tipAmount set to nil when creating the SMPCheckoutRequest object.

2. Programmatic tipping via the tipAmount property. Ask the user in your own UI for an appropriate tip amount and then set the tipAmount property on SMPCheckoutRequest. This will be added to the total amount, but will be displayed to the user separately during checkout.

3. Tip on Card Reader. TCR prompts the customer directly on the card reader's display for a tip amount, rather than prompting for a tip amount on the iPhone or iPad display.

#### Tip on Card Reader (TCR)

Important: Not all card readers support this feature. To find out if the feature is supported for the last-used card reader, you should always check SMPSumUpSDK.isTipOnCardReaderAvailable. You must handle this case yourself in order to avoid no tip from being prompted.

To do this:

Before calling SMPSumUpSDK checkoutWithRequest:fromViewController:completion:, check SMPSumUpSDK.isTipOnCardReaderAvailable:

- If NO, you should prompt the user for a tip amount yourself and set tipAmount on SMPCheckoutRequest

- If YES, you may set tipOnCardReaderIfAvailable on SMPCheckoutRequest to YES. Do not prompt the user for a tip amount or set tipAmount if you do this.

### Table of Contents
* [Installation](#installation)
  * [Manual Integration](#manual-integration)
  * [Integration via CocoaPods](#integration-via-cocoapods)
  * [Integration via Carthage](#integration-via-carthage)
  * [Integration via Swift PM](#integration-via-swift-pm)
  * [Supported device orientation](#supported-device-orientation)
  * [Privacy Info plist keys](#privacy-info-plist-keys)
* [Getting started](#getting-started)
  * [Import the SDK](#import-the-sdk)
  * [Authenticate app](#authenticate-app)
  * [Login](#login)
  * [Accept card payments](#accept-card-payments)
  * [Update checkout preferences](#update-checkout-preferences)
* [Tap to Pay on iPhone](#tap-to-pay-on-iphone)
* [Out of Scope](#out-of-scope)
* [Community](#community)
* [Changelog](#changelog)
* [License](#license)

## Installation

### Manual Integration

The SumUp SDK is provided as an XCFramework `SumUpSDK.xcframework` that contains
the headers and bundles bundles containing resources such as images and localizations.
Please follow the steps below to prepare your project:

1. Drag and drop the `SumUpSDK.xcframework` to your Xcode project's "Frameworks,
Libraries, and Embedded Content" on the General settings tab.
2. Make sure the [required Info.plist keys](#privacy-info-plist-keys) are present.

> Note:
> You can use the [sample app](SampleApp/SumUpSDKSampleApp)
> that is provided with the SumUp SDK as a reference project.
> The Xcode project contains sample apps written in Objective-C and Swift.
> See [Test your integration](#test-your-integration) for more information.

### Integration via CocoaPods

The SumUp SDK can be integrated via CocoaPods.

```ruby
target '<Your Target Name>' do
    pod 'SumUpSDK', '~> 4.0'
end
```

Make sure the [required Info.plist keys](#privacy-info-plist-keys) are present.

To learn more about setting up your project for CocoaPods, please refer to the [official documentation](https://cocoapods.org/#install).

### Integration via Carthage

:warning: Distributing XCFrameworks with the latest Carthage version (0.35.0) is not yet available.
There is an open issue ([#2799](https://github.com/Carthage/Carthage/issues/2799)) to solve this.
Once that issue is fixed, we expect Carthage to work again.

The SumUp SDK can be integrated with Carthage by following the steps below:

1. Add the following line to your `Cartfile`:

        github "sumup/sumup-ios-sdk"

2. Run `carthage update sumup-ios-sdk`
3. Drag and drop the `Carthage/Build/iOS/SumUpSDK.xcframework` to your Xcode project's "Frameworks,
Libraries, and Embedded Content" on the General settings tab.
4. Make sure the [required Info.plist keys](#privacy-info-plist-keys) are present.

To learn more about setting up your project for Carthage, please refer to the [official documentation](https://github.com/Carthage/Carthage#if-youre-building-for-ios-tvos-or-watchos).

> Note:
> See [Test your integration](#test-your-integration) for more information.

### Integration via Swift PM

The latest Swift Package Manager version added support to [distribute binary frameworks as Swift Packages](https://developer.apple.com/documentation/swift_packages/distributing_binary_frameworks_as_swift_packages). 

If you're using Xcode 12.3 or later, this should work out of the box.

Unfortunately, Xcode 12 releases before that had an issue, ([Bug Report SR-13343](https://bugs.swift.org/browse/SR-13343)), that added the framework as a static library, not as an embedded dynamic framework. 

Follow this workaround to manage SumUp SDK versions via Swift PM in those cases:

1. Add the package dependency to the repository `https://github.com/sumup/sumup-ios-sdk` (*File > Swift Packages > Add Package Dependency...*) with the version `Up to Next Major: 4.0.0`
2. Leave the checkbox unchecked for the SumUpSDK at the integration popup (*Add Package to ...:*)
![Swift PM - do not auto-integrate SDK](.github/assets/setup_swiftpm_skipautointegrate.png)
3. From the Project Navigator, drag and drop the `SumUpSDK/Referenced Binaries/SumUpSDK.xcframework` to your Xcode project's "Frameworks, Libraries, and Embedded Content" on the General settings tab.
4. Make sure the [required Info.plist keys](#privacy-info-plist-keys) are present.

To learn more about adding Swift Package dependencies, please refer to the [official documentation](https://developer.apple.com/documentation/xcode/adding_package_dependencies_to_your_app).

### Test your integration
In your debug setup you can call `+[SMPSumUpSDK testSDKIntegration]`.
It will run various checks and print its findings to the console.
Please do not call it in your Release build.

### Supported device orientation
The SDK supports all device orientations on iPad and portrait on iPhone.
Feel free to support other orientations on iPhone but please keep in mind that
the SDK's UI will be presented in portrait on iPhone.
See `UISupportedInterfaceOrientations` in the sample app's
[Info.plist](SampleApp/SumUpSDKSampleApp/SumUpSDKSampleApp-Info.plist#L54-L65)
or the "General" tab in Xcode's Target Editor.

### Privacy Info plist keys
The SumUp SDK requires access to the user's location and Bluetooth peripherals. If your app has not asked for the user's permission,
the SumUp SDK will ask at the time of the first login or checkout attempt. Please add the
following keys to your info plist file:

        NSLocationWhenInUseUsageDescription
        NSBluetoothAlwaysUsageDescription
        NSBluetoothPeripheralUsageDescription (unless your deployment target is at least iOS 13)

> Note:
> - Please refer to the sample app's [Info.plist](SampleApp/SumUpSDKSampleApp/SumUpSDKSampleApp-Info.plist#L38-L43)
for more information regarding the listed permissions required.
> - You can provide localization by providing a localized
[InfoPlist.strings](SampleApp/SumUpSDKSampleApp/en.lproj/InfoPlist.strings) file.
> - For further information, see the iOS Developer Library on
[location usage on iOS 8 and later](https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW26),
[Bluetooth peripheral usage](https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW20).

## Getting started

### Import the SDK

To import the SDK in Objective-C source files, you can use `#import <SumUpSDK/SumUpSDK.h>`. If module
support is enabled in your project, you can use `@import SumUpSDK;` instead.

In Swift, use `import SumUpSDK`. You do not have to add any headers to your bridging header.

### Authenticate app
Before calling any additional feature of the SumUp SDK, you are required to set up the SDK with your Affiliate (Access) Key:
```objc
[SMPSumUpSDK setupWithAPIKey:@"MyAPIKey"];
```

> Note:
> `setupWithAPIKey:` checks for the user's location permission. Consequently,
do not call this method as part of the app launch. This method must be called on
the main queue.

### Login

Following app authentication, a registered SumUp merchant account needs to be logged in.
Present a login screen from your `UIViewController`, using the following method:
```objc
[SMPSumUpSDK presentLoginFromViewController:vc
                                   animated:YES
                            completionBlock:nil];
```


> Note:
>  It is also possible to login an account with a token, without the user entering their SumUp login credentials in the SDK. Please refer to section [Transparent Authentication](#transparent-authentication). 
>  
> To log out of the SDK, please refer to `logoutWithCompletionBlock:`.

### Accept card payments
Once logged in, you can start using the SumUp SDK to accept card payments.

#### Prepare checkout request
Prepare a checkout request that encapsulates the information regarding the transaction.

For this, you will need to create an instance of `SMPCheckoutRequest`:


```objc
SMPCheckoutRequest *request = [SMPCheckoutRequest requestWithTotal:[NSDecimalNumber decimalNumberWithString:@"10.00"]
                                                             title:@"your title"
                                                      currencyCode:[[SMPSumUpSDK currentMerchant] currencyCode]];
```

Please note that you need to pass an `NSDecimalNumber` as the total value.
While `NSDecimalNumber` is a subclass of `NSNumber` it is not advised to use the
convenience method of `NSNumber` to create an `NSDecimalNumber`.

#### Additional checkout parameters
When setting up the `SMPCheckoutRequest` object, the following optional parameters can be included:

##### Tip amount
A tip amount can be processed in addition to the `totalAmount` using the `tipAmount` parameter.
The tip amount will then be shown during the checkout process and be included in the response.
Please note that a tip amount cannot be changed during/after the checkout.
Just like the `totalAmount` it is an `NSDecimalNumber` so make sure to
not accidentally pass an `NSNumber`.

##### Transaction identifier
The `foreignTransactionID` identifier will be associated with the transaction
and can be used to retrieve details related to the transaction.
See [API documentation](https://docs.sumup.com/rest-api/#tag/Transactions) for details.
Please make sure that this ID is unique within the scope of the SumUp merchant account
and sub-accounts. It must not be longer than 128 characters.

```
// set an optional identifier
[request setForeignTransactionID:@"my-unique-id"];
```

##### Skip success screen
To skip the screen shown at the end of a successful transaction, the
`SMPSkipScreenOptionSuccess` option can be used.
When setting this option your application is responsible for displaying
the transaction result to the customer.
In combination with the Receipts API, your application can also send your own receipts,
see [API documentation](https://docs.sumup.com/rest-api/#tag/Transactions) for details.
Please note that success screens will still be shown when using the SumUp Air Lite readers.

##### Skip failed screen
To skip the screen shown at the end of a failed transaction, the
`SMPSkipScreenOptionFailed` option can be used.
When setting this option your application is responsible for displaying
the transaction result to the customer.
Please note that failed screens will still be shown when using the SumUp Air Lite readers.

### Prepare the SumUp Card terminal in advance
In order to prepare a SumUp card terminal for checkout, `prepareForCheckout` can be called in advance. A registered SumUp merchant account needs to be logged in, and the card terminal must already be setup. You should use this method to let the SDK know that the user is most likely starting a checkout attempt soon; for example when entering an amount or adding products to a shopping cart. This allows the SDK to take appropriate measures, like attempting to wake a connected card terminal.

### Transparent Authentication
To authenticate an account without the user typing in their SumUp credentials each time, you can generate an access token using OAuth2.0 and use it to transparently login to the SumUp SDK.

```objc
[SMPSumUpSDK loginWithToken:@"MY_ACCESS_TOKEN" 
                 completion:nil];
```

For information about how to obtain a token, please see the [API Documentation](https://developer.sumup.com/docs/authorization/).

If the token is invalid, `SMPSumUpSDKErrorInvalidAccessToken` will be returned.

#### Initiate Checkout Request
Start a payment by using the checkout request below:


```objc
[SMPSumUpSDK checkoutWithRequest:request
              fromViewController:vc
                      completion:^(SMPCheckoutResult *result, NSError *error) {
                      // handle completed and failed payments here
                      // retrieve information via result.additionalInfo
}];
```

#### Checkout Error Codes
Possible values of [error code](./SumUpSDK.xcframework/ios-armv7_arm64/SumUpSDK.framework/Headers/SMPSumUpSDK.h#L155) received in the `checkoutWithRequest:` completion block.

### Update checkout preferences
When logged in you can let merchants check and update their checkout
preferences. Merchants can select their preferred card terminal and set up a
new one if needed. The preferences available to a merchant depend on their
respective account settings.

```objc
[SMPSumUpSDK presentCheckoutPreferencesFromViewController:self
                                                 animated:YES
                                               completion:^(BOOL success, NSError * _Nullable error) {
                                                 if (!success) {
                                                   // there was a problem presenting the preferences
                                                 } else {
                                                   // next checkout will reflect the merchant's changes.
                                                 }
                                               }];
```

## Tap to Pay on iPhone

With Tap to Pay on iPhone merchants can accept contactless card payments on their iPhone without needing a card reader.

To add Tap to Pay on iPhone to your app:

* Request the Tap to Pay on iPhone entitlement from Apple, receive approval, and then add the `com.apple.developer.proximity-reader.payment.acceptance` entitlement to your app. [Setting up the entitlement](https://developer.apple.com/documentation/proximityreader/setting-up-the-entitlement-for-tap-to-pay-on-iphone?language=objc).
* This feature requires an iPhone XS or later, running iOS 16.4 (iOS 16.7 starting July 8th) or later (ideally 17.5 or later.) The feature does not work on iPad.
* For debugging and testing you will need to be logged into an iPhone with a non-Sandbox Apple ID. Using a Sandbox Apple ID requires both Apple and SumUp implementations to connect to their respective non-production (test) backends, which the SDK does not support.
* During testing, to avoid transactions going to the acquirer and transferring real money, use a SumUp test account. To create a test account, please log in [here](https://auth.sumup.com/flows/login?client_id=developer_portal&login_challenge=fd21867fbc224ee985c0e206ef7f7549). Hover over "Account" on the top right corner of the page and click on “Test Profile” in the drop-down menu.

In your code:

* Make a call to check feature availability: is the Tap to Pay on iPhone payment method available for the current merchant?
* Trigger activation if needed. Activation sets up the iPhone to receive payments, shows the merchant how to use the feature, and links the SumUp account and Apple ID.
* Start the checkout.

### Check feature availability

* Call `checkTapToPayAvailability` on `SMPSumUpSDK` to check the availability of the Tap to Pay on iPhone payment method. This call, which requires the SDK to be in a logged-in state, may internally perform one or more network calls.
* If the feature is not available, your app could, as an example, hide or disable a button or menu item representing the Tap to Pay on iPhone payment method. 
* The feature is generally available when the following criteria are fulfilled: 
    * the iPhone model and iOS version requirements are met 
    * the user logs in with a SumUp account registered in one of the countries where SumUp supports Tap to Pay on iPhone (temporarily with exception of Brazil) 

### Perform activation if needed

* Activation must be completed before the first transaction can be performed. Activation means:
    * the merchant links their Apple ID with their SumUp account
    * the iPhone is prepared, which can take 45 seconds or longer
* This needs to be done once per merchant account, per device. 
* In addition to determining feature availability, `checkTapToPayAvailability` also indicates whether Tap to Pay on iPhone has been activated yet for the current merchant. 
* If it has not yet been activated then you should trigger activation by calling `presentTapToPayActivation` at a convenient time. Calling it more than once will still show the user education screens each time. Independently, the activation from the initial setup will remain valid.

Code example: checking availability and activation status

Objective-C

```obj-c
[SMPSumUpSDK checkTapToPayAvailability:^(BOOL isAvailable, BOOL isActivated, NSError * _Nullable error) {
    if (error == nil) {
        if (!isAvailable) {
            // Tap to Pay on iPhone is not available for the merchant
            return;
        }

        if (!isActivated) {
            // Tap to Pay on iPhone needs activation - call presentTapToPayActivation
            return;
        }

    // The app is ready to take Tap to Pay on iPhone payments!

    } else {
        // An error occurred
    }
}];
```

Swift

```swift
SumUpSDK.checkTapToPayAvailability { isAvailable, isActivated, error in

    if let error {
        // An error occurred
        return
    }

    if !isAvailable {
        // Tap to Pay on iPhone is not available for the merchant
        return
    }

    if !isActivated {
        // Tap to Pay on iPhone needs activation - call presentTapToPayActivation
        return
    }

    // The app is ready to take Tap to Pay on iPhone payments!
}
```

### Do the checkout

* Use the initializer of `SMPCheckoutRequest`/`CheckoutRequest` that takes a `paymentMethod` parameter, and specify `SMPPaymentMethodTapToPay`/`tapToPay`:

Objective-C

```objc
SMPCheckoutRequest request = [SMPCheckoutRequest requestWithTotal:total
                                                            title:itemDescription
                                                     currencyCode:SMPSumUpSDK.currentMerchant.currencyCode
                                                    paymentMethod:SMPPaymentMethodTapToPay];
```

Swift

```swift
let request = CheckoutRequest(total: total, 
                              title: itemDescription,
                              currencyCode: SumUpSDK.currentMerchant?.currencyCode,
                              paymentMethod: .tapToPay)
```

* The rest of the checkout process is the same as when using a card reader.

### Background initialization

When `SMPSumUpSDK` initializes, it triggers a background initialization process. This can take about 10-15 seconds and must be completed before the first Tap to Pay on iPhone card-read can occur. If the user quickly initiates a transaction after your app launches, it may take longer than usual as it first waits for the initialization to complete. For subsequent transactions, however, (until the app is relaunched) this delay should not happen.

### Card Verification Method

Tap to Pay on iPhone supports contactless payments with physical cards or wallets (eg. Apple or Google Pay).
When accepting payments with physical cards, additional card holder verification in a form of PIN may be required. In the majority of such cases, the UI will display a PIN pad where the customer can enter and confirm the digits. 
However, not every physical card allows this type of PIN processing. Some can only be processed by inserting the card into a physical card reader, which is not possible in the case of Tap to Pay on iPhone. 
In such an instance, the merchant should ask the customer to try a different card or payment method.

### Best practices

* Follow the [Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/tap-to-pay-on-iphone) to save time when Apple reviews your app. 
* Also consider [Apple’s marketing guidelines](https://developer.apple.com/tap-to-pay/marketing-guidelines/) and use standard assets where possible. 

### Known issues

* If entitlements are not correctly set up in your app, `presentTapToPayActivation` may show an error Alert with "Failed to show Terms of Service".
* Bluetooth permissions will be requested even if the merchant does not plan to use any of the SumUp Bluetooth  card readers. This will be fixed in an upcoming release.
* Businesses using SumUp sub-accounts must first activate the feature on their main account before using it on devices logged in with sub-accounts, otherwise an error message will appear during activation for the sub-account user.
* The `title` field in `SMPCheckoutRequest` does not currently appear on receipts.

## Out of Scope
The following functions are handled by the [SumUp APIs](http://docs.sumup.com/rest-api/):
* [Refunds](https://docs.sumup.com/rest-api/#tag/Refunds)
* [Transaction history](https://docs.sumup.com/rest-api/#tag/Transactions)
* [Receipts](https://docs.sumup.com/rest-api/#tag/Receipts)
* [Account management](https://docs.sumup.com/rest-api/#tag/Account-Details)

## Community
- **Questions?** Get in contact with our integration team by sending an email to integration@sumup.com.
- **Found a bug?** [Open an issue](https://github.com/sumup/sumup-ios-sdk/issues/new).
Please provide as much information as possible.

## Changelog
[SumUp iOS SDK Changelog](CHANGELOG.md)

## License
[SumUp iOS SDK License](LICENSE)
