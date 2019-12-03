### U6 fork of AWS Amplify

AWS Amplify is a beast. It is in the critical login and signup paths of our
app. It also manages push notification tokens which is difficult to test fully.
We don't upgrade it lightly. Whenever we find a bug we prefer patching the
existing version over upgrading to a new release containing the bug fix.

AWS Amplify is a monorepo. It uses lerna to manage its release process. Lerna
builds individual components under the `@aws-amplify/` namespace. These built
components cannot be simplfy forked at the source. Instead we have to build the
entire release locally and push the select build artifacts for each patched
component up to our github repo.

This file documents the exact process we use to build and release these forked
AWS Amplify components.

### Build procedures

1. Clone our fork of `aws-amplify/amplify-js` at this exact location on your laptop:

```
➜  sudo mkdir /root
➜  sudo chown $USER /root
➜  cd /root
➜  git clone git@github.com:ubiquity6/amplify-js.git app
➜  cd /root/app
```

We clone to `/root/app` because this is where AWS builds amplify on their
machines and the path is embedded in the source maps. We want to minimize the
difference.

2. Check out `u6dev_rn61`

```
➜  git checkout u6dev_rn61
Switched to branch 'u6dev_rn61'
Your branch is up to date with 'origin/u6dev_rn61'.
```

3. Cherry pick any fixes from `aws-amplify`

4. Build

```
➜  yarn
➜  yarn bootstrap
➜  yarn build
```

5. Identify the amplify components that were patched and required to fork.
   Verify a repo is already created for it on our private github account. For
   example, we forked `@aws-amplify/pushnotification` here:
   https://github.com/ubiquity6/aws-amplify-pushnotification

6. Continue with `@aws-amplify/pushnotification` as an example. The build artifacts
   are in `packages/pushnotification`. They are mixed with other files that should
   not be put in the forked repo. Exercise care here. One way to find
   the files already installed in our u6 repo's `node_modules` and copy only those files
   to the forked repo:

```
➜  ubiquity) find node_modules/@aws-amplify/pushnotification
node_modules/@aws-amplify/pushnotification
node_modules/@aws-amplify/pushnotification/CHANGELOG.md
node_modules/@aws-amplify/pushnotification/__tests__
node_modules/@aws-amplify/pushnotification/package.json
node_modules/@aws-amplify/pushnotification/android
node_modules/@aws-amplify/pushnotification/android/libs
node_modules/@aws-amplify/pushnotification/android/libs/aws-android-sdk-pinpoint-2.6.13.jar
node_modules/@aws-amplify/pushnotification/android/libs/aws-android-sdk-cognito-2.6.13.jar
node_modules/@aws-amplify/pushnotification/android/libs/aws-android-sdk-mobile-client-2.6.13.aar
node_modules/@aws-amplify/pushnotification/android/libs/aws-android-sdk-core-2.6.13.jar
node_modules/@aws-amplify/pushnotification/android/libs/aws-android-sdk-auth-core-2.6.13.aar
node_modules/@aws-amplify/pushnotification/android/build.gradle
node_modules/@aws-amplify/pushnotification/android/src
node_modules/@aws-amplify/pushnotification/android/src/main
node_modules/@aws-amplify/pushnotification/android/src/main/AndroidManifest.xml
node_modules/@aws-amplify/pushnotification/android/src/main/java
node_modules/@aws-amplify/pushnotification/android/src/main/java/com
node_modules/@aws-amplify/pushnotification/android/src/main/java/com/amazonaws
node_modules/@aws-amplify/pushnotification/android/src/main/java/com/amazonaws/amplify
node_modules/@aws-amplify/pushnotification/android/src/main/java/com/amazonaws/amplify/pushnotification
node_modules/@aws-amplify/pushnotification/android/src/main/java/com/amazonaws/amplify/pushnotification/RNPushNotificationMessagingService.java
node_modules/@aws-amplify/pushnotification/android/src/main/java/com/amazonaws/amplify/pushnotification/RNPushNotificationDeviceIDService.java
node_modules/@aws-amplify/pushnotification/android/src/main/java/com/amazonaws/amplify/pushnotification/RNPushNotificationPackage.java
node_modules/@aws-amplify/pushnotification/android/src/main/java/com/amazonaws/amplify/pushnotification/RNPushNotificationModule.java
node_modules/@aws-amplify/pushnotification/android/src/main/java/com/amazonaws/amplify/pushnotification/modules
node_modules/@aws-amplify/pushnotification/android/src/main/java/com/amazonaws/amplify/pushnotification/modules/RNPushNotificationBroadcastReceiver.java
node_modules/@aws-amplify/pushnotification/android/src/main/java/com/amazonaws/amplify/pushnotification/modules/RNPushNotificationCommon.java
node_modules/@aws-amplify/pushnotification/android/src/main/java/com/amazonaws/amplify/pushnotification/modules/RNPushNotificationJsDelivery.java
node_modules/@aws-amplify/pushnotification/android/src/main/java/com/amazonaws/amplify/pushnotification/modules/RNPushNotificationPublisher.java
node_modules/@aws-amplify/pushnotification/android/src/main/java/com/amazonaws/amplify/pushnotification/modules/RNPushNotificationAttributes.java
node_modules/@aws-amplify/pushnotification/android/src/main/java/com/amazonaws/amplify/pushnotification/modules/RNPushNotificationHelper.java
node_modules/@aws-amplify/pushnotification/lib
node_modules/@aws-amplify/pushnotification/lib/PushNotification.d.ts
node_modules/@aws-amplify/pushnotification/lib/index.js
node_modules/@aws-amplify/pushnotification/lib/PushNotification.js.map
node_modules/@aws-amplify/pushnotification/lib/PushNotification.js
node_modules/@aws-amplify/pushnotification/lib/index.js.map
node_modules/@aws-amplify/pushnotification/lib/index.d.ts
node_modules/@aws-amplify/pushnotification/src
node_modules/@aws-amplify/pushnotification/src/PushNotification.ts
node_modules/@aws-amplify/pushnotification/src/index.ts
```

7. As a precaution, diff the files and make sure you see the expected changes:

```
➜  diff packages/pushnotification/lib/PushNotification.js ~/src/github.com/ubiquity6/u3/node_modules/@aws-amplify/pushnotification/lib/PushNotification.js
16d15
< var push_notification_ios_1 = require("@react-native-community/push-notification-ios");
100c99
<         push_notification_ios_1.default.requestPermissions({
---
>         react_native_1.PushNotificationIOS.requestPermissions({
112c111
<             push_notification_ios_1.default.getInitialNotification().then(function (data) {
---
>             react_native_1.PushNotificationIOS.getInitialNotification().then(function (data) {
251c250
<             push_notification_ios_1.default.addEventListener('register', function (data) {
---
>             react_native_1.PushNotificationIOS.addEventListener('register', function (data) {
256c255
<             push_notification_ios_1.default.addEventListener('notification', handler);
---
>             react_native_1.PushNotificationIOS.addEventListener('notification', handler);
```

8. Check in the release build to the forked release repo and update our u6 repo to use it.

9. Push the cherry-picked changes to our forked source repo.
