# Amplify Analytics Notifications Sample

This sample uses 
- Amplify Analytics's Pinpoint plugin 
- registers for push notifications
- leverages the escape hatch to use pinpoint campaigns and journeys

## Install dependencies

If you are trying to install the latest changes from your local pod, make sure to run `pod cache clean --all` to clear the pods for a fresh install, otherwise, run

```bash
pod install
```
or in new project, `pod init` and add
```
pod 'Amplify'
pod 'AmplifyPlugins/AWSPinpointAnalyticsPlugin'
pod 'AmplifyPlugins/AWSCognitoAuthPlugin'
```
## Generate resources (amplifyconfiguration.json)

1. Run `amplify init` and choose `ios` for the type of app you're building

2. Add Analytics and Auth `amplify add analytics`

3. `amplify push`

## Open App - register bundle with push notifications

Prerequiste - enrolled in Apple developer program

1. `open <Project>.xcworkspace`.

2. Under the target, select "Signing & Capabilities", update the bundle identifer to something unique and select your Apple Developer account for the Team. Signing should be successful

3. In the app, it will already have Push Notifications added. If on a new app, click `+ Capability` and add Push Notification. 

4. Add `Background Modes` and check the Remote notifications option

5. If everything is set up correctly, you should see your Identifer at the (Apple Developer Program)[https://developer.apple.com/account/resources/identifiers/list] under Identifers.

## Run the app

1. Drag the `amplifyconfiguration.json` file over to the project. Check off `Copy files if needed`

2. CMD+K to clean, and CMD+B to build, plug in your device and CMD+R to run the app on the device

## Request for Push Notifications

Add the following code to request for push notifications.

1. Inside the AppDelegate's `application(didFinishLaunchingWithOptions)`, before `return true`, add
```swift
// Present the user with a request to authorize push notifications
registerForPushNotifications()
```
2. Inside this `class AppDelegate`, add the following methods
```swift
// MARK: Remote Notifications Lifecycle
func application(_ application: UIApplication,
    didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    let tokenParts = deviceToken.map { data in String(format: "%02.2hhx", data) }
    let token = tokenParts.joined()
    print("Device Token: \(token)")

    // Register the device token with Pinpoint as the endpoint for this user
    pinpoint!.notificationManager.interceptDidRegisterForRemoteNotifications(withDeviceToken: deviceToken)
}

func application(_ application: UIApplication,
    didFailToRegisterForRemoteNotificationsWithError error: Error) {
    print("Failed to register: \(error)")
}

func application(_ application: UIApplication,
                  didReceiveRemoteNotification userInfo: [AnyHashable: Any],
                  fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {

    if (application.applicationState == .active) {
        let alert = UIAlertController(title: "Notification Received",
                                      message: userInfo.description,
                                      preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "Ok", style: .default, handler: nil))

        UIApplication.shared.keyWindow?.rootViewController?.present(
            alert, animated: true, completion:nil)
    }

  // pinpoint!.notificationManager.interceptDidReceiveRemoteNotification(userInfo, fetchCompletionHandler: completionHandler)
}

// MARK: Push Notification methods

func registerForPushNotifications() {
    UNUserNotificationCenter.current()
        .requestAuthorization(options:[.alert, .sound, .badge]) {[weak self] granted, error in
            print("Permission granted: \(granted)")
            guard granted else { return }

            // Only get the notification settings if user has granted permissions
            self?.getNotificationSettings()
    }

}

func getNotificationSettings() {
    UNUserNotificationCenter.current().getNotificationSettings { settings in
        print("Notification settings: \(settings)")
        guard settings.authorizationStatus == .authorized else { return }

        DispatchQueue.main.async {
            // Register with Apple Push Notification service
            UIApplication.shared.registerForRemoteNotifications()
        }
    }
}
```

3. Run the app, if you run this in the simulator, you will get 
```
Failed to register: Error Domain=NSCocoaErrorDomain Code=3010 "remote notifications are not supported in the simulator" UserInfo={NSLocalizedDescription=remote notifications are not supported in the simulator}
```
so make sure you run this on a real device, note down the device token allowing notifications from the prompt in the app
```
Device Token: 123451234561234567f617f8a174f625ede9f519662f2ac1fb69e51234512346
```

## Generate a Push Notification

If you already your push notifications key (p8 file), you can skip to step 3

1. In the Apple developer portal, go over to Keys, click on + beside "Keys" to add a new key. 

2. Give it a name like "Push Notifications Key" and check off "Apple Push Notifications (APNs)

3. Install https://github.com/onmyway133/PushNotifications and open it

4. Under Authentication, select Token

5. P8: select your p8 file

6. KeyId: From the Push Notifications key in step 2

7. TeamID: top right corner beside your name is your apple developer account team ID

8. Bundle Identifier: bundle of your app

9. Device Token: The device token from the previous section

10. You can enter a body like:
```
{
  "aps": {
    "alert": "Breaking News!",
    "sound": "default",
    "link_url": "https://raywenderlich.com"
  }
}
```
reference: https://www.raywenderlich.com/8164-push-notifications-tutorial-getting-started

11. Press send. it should say Succeeded. The app on the device should get a notification.

## Enable Pinpoint Notifications

This enables the push notification channel to allow pinpoint to send push notifications to your users.

1. Run `amplify add notifications`
```
? Choose the push notification channel to enable. `APNS`
? Choose authentication method used for APNs `Key`
? The bundle id used for APNs Tokens:  `com.yourbundle.identifier`
? The team id used for APNs Tokens:  ABCDEX6Q6
? The key id used for APNs Tokens:  ABCDEGR62
? The key file path (.p8):  `path/to/your/p8/file.p8`
✔ The APNS channel has been successfully enabled.
```

This adds the p8 file and enables it for both APNS sandbox and production. You can check out the Pinpoint console by running `amplify console analytics`.

Under Settings, Push notifications, you will see that Apple Push Notification Service (APNS) is enabled.


## Enable pinpoint notifications manager

1. Add the following imports at the top of AppDelegate
```swift
import Amplify
import AmplifyPlugins
import AWSPinpoint
```


2. Inside `application(didFinishLaunchingWithOptions`, before `registerForPushNotifications()`, add
```swift
// Optionally enable logging
AWSDDLog.sharedInstance.logLevel = .verbose
AWSDDLog.add(AWSDDTTYLogger.sharedInstance)
Amplify.Logging.logLevel = .verbose

do {
    try Amplify.add(plugin: AWSCognitoAuthPlugin())
    try Amplify.add(plugin: AWSPinpointAnalyticsPlugin())
    try Amplify.configure()
    print("Amplify configured with Auth and Analytics plugins")
} catch {
    print("Failed to initialize Amplify with \(error)")
}
```

4. Get the AWSPinpoint instance
```swift
/// Add as property on the AppDelegate
var pinpoint: AWSPinpoint?
```
then Above call to `registerForPushNotifications()`

```swift
do {
    let plugin = try Amplify.Analytics.getPlugin(for: "awsPinpointAnalyticsPlugin") as! AWSPinpointAnalyticsPlugin
    pinpoint = plugin.getEscapeHatch()
    print("Sucessfully got AWSPinpoint instance from escape hatch")
} catch {
    print("Get escape hatch failed with error - \(error)")
}
```

5. Build and run the app, you should see this in the logs
```
Amplify configured with Auth and Analytics plugins
Sucessfully got AWSPinpoint instance from escape hatch
```

6. Now uncomment the following lines to register the device token with the pinpoint endpoint

```swift
pinpoint!.notificationManager.interceptDidRegisterForRemoteNotifications(withDeviceToken: deviceToken)
```

and

```swift
pinpoint!.notificationManager.interceptDidReceiveRemoteNotification(userInfo, fetchCompletionHandler: completionHandler)
```

You can put a breakpoint there to make sure when the app starts, device token is passed to the pinpoint's notificationManager.

If you have verbose logging turned on, search for the device token and you should see it set as the `Address` field on the Pinpoint Endpoint. In the same payload, note that OptOut should also be `None`. Take note of the endpointId

## Send test notification from Pinpoint

1. `amplify console analytics` to open pinpoint console

2. Click on Test messaging tab, enter the endpoint Id you got from the previous section in Destinations

3. Under Message, enter a title and body, and press Send message. You should see a notification set to your app.

## Create Pinpoint Campaign

1. Go to Campaign, click Create Campaign, provide a campaign name, and select Push Notifications as the channel, and click next.

2. In the segment section, select `Create a segment` and you should see 1 device as a targeted endpoint, which is the app we are running on the device. Choose this option and then choose **Next Step**.

3. Provide text for a sample title and body for push notification, enter the device token or endpoint ID retrieved from the app.

- Make sure the app is in the foreground, click on Test message and you should see an alert modal pop up with your test message wrapped in push notification data.
- Make sure the app is in the background, click on Test message and you should see push notification slide down from the top.


## Create Pinpoint Journey
