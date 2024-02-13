# Push notification guide:

### 1-  What is push notification:
It’s a term that combines two separate technologies ***Push*** & ***Notification*** when : 
#### - Push for the technology for sending messages from your server to users even when they're not actively using your website.
#### - Notification: for  the technology that displays the pushed information on the user's device.
So in this collection ”push notifications” we mean the combination of pushing a message followed by displaying it as a notification.
### 2- Why use push notifications?
- For users, push notifications are a way to receive timely, relevant, and precise information.

- For you (a website owner), push notifications are a way to increase user engagement.

### 3 - Integration with PWA: 
03 technologies in order to implements  with PWA: 
 - ***Push API:*** gives web applications the ability to receive messages pushed to them from a server.
 - ***Notification API:*** allows web pages to control the display of system notifications to the end user.
 - ***Service worker:***  javascript files that act as proxies between web browsers and web servers.

### 4 - Steps:
 - ***First make sure all the 03 main technologies are supported in the browser:***

```js
if (!("Notification" in window)) {
  // Notification API isn't supported
}
if (!("serviceWorker" in navigator)) {
  // Service Workers aren't supported
  return;
}
if (!("PushManager" in window)) {
  // Push API isn't supported
  return;
}

```
- ***Register a service worker:***  After registering a service worker the browser will give it access to service worker APIs including Push.

```js
if ("serviceWorker" in navigator) {
  navigator.serviceWorker.register("/sw.js").then(
    (registration) => {
      console.log("Service worker registration succeeded:", registration);
    },
    (error) => {
      console.error(`Service worker registration failed: ${error}`);
    },
  );
} else {
  console.error("Service workers are not supported.");
}
```
- ***Request Permission:*** Provide a method for the user to grant permission with a gesture, such as clicking or tapping a button, to achieve that here is an example:

```js
function requestPermission() {
  return new Promise((resolve, reject) => {
    const permissionResult = Notification.requestPermission((result) => {
      resolve(result);
    });
    if (permissionResult) {
      permissionResult.then(resolve, reject);
    }
  }).then((permissionResult) => {
    if (permissionResult !== "granted") {
      throw new Error("Permission denied");
    }

    subscribeUserToPush();
  });
}
```
- ***Subscribe the user:*** After getting user permission we can now subscribe the user to the Push API.
```js
async function subscribeUserToPush() {
  const registration = await registerServiceWorker();

  const subscribeOptions = {
    userVisibleOnly: true,
    applicationServerKey: PUBLIC_VAPID_KEY,
  };

  const pushSubscription = await registration.pushManager.subscribe(
    subscribeOptions
  );

  axios
    .post("/api/subscription", pushSubscription)
    .then((response) => {
      console.log(response);
    })
    .catch((error) => console.log(error));
}
```
   **userVisibleOnly:** A boolean that must be set to true indicating that the user will be notified every time a push message is sent.
   **applicationServerKey:** This is a public key generated in a pair along with a private key and are unique to your app. Application server keys otherwise knowns as VAPID keys are used by a push service to identify the application subscribing a user and ensure that the same application is messaging that user.

- ***Send messages from server:***
The server side shoud work on:

 1. Receive user push notification registrations.

 2. Store the push notification subscription endpoint and encryption keys together with the user’s account information.

 3. Determine which users should receive a push notification when important events occur.

 4. Prepare and send push notifications to the users you identify.

- ***Display Notification to the user:***

In the Service worker we shoud add a push event listener that listens for the push event. then display the pushed message using the notification API of the browser

```js
self.addEventListener('push', (event) => {
    let pushMessageJSON = event.data.json();
    event.waitUntil(self.registration.showNotification(pushMessageJSON.title, {
        body: pushMessageJSON.body,
        tag: pushMessageJSON.tag,
        actions: [{
            action: pushMessageJSON.actionURL,
            title: pushMessageJSON.actionTitle,
        }]
    }));
}
```
### 5 - Web push notification on iOS:
 - Starting from iOS 16.4, websites are now able to start sending iOS push notifications.
 - The new support follow the web standars, ***no site changes required*** if your website already imlement push notification.
 - In iOS users need to add your the website to their iOS home screen, ***showing intent*** This can be done by creating a shortcut for the web page using the **Add to Home Screen** button in the share screen.
 - when the user subscribe, the notifications will then be shown in iOS Notification Center and Lock Screen.

### 6 - Designing a PWA For iOS:
Unlike the other platform, iOS does not use the icons specified in the Web App Manifest, instead it using HTML tags with additional metadata to set the home and the splash icons for differents iOS devices
#### 6 - 1: custom icon: 
In order to add custom icon for PWA in iOS, this the following html tag:

```html
<link rel="apple-touch-icon" href="touch-icon-iphone.png">
<link rel="apple-touch-icon" sizes="152x152" href="touch-icon-ipad.png">
<link rel="apple-touch-icon" sizes="180x180" href="touch-icon-iphone-retina.png">
<link rel="apple-touch-icon" sizes="167x167" href="touch-icon-ipad-retina.png">
```
in this example we add tags for different devices iOS.

#### 6 - 1: Custom splash screen: 
The following exmaple tags sets the head element of PWA website to support custom splash screens for the different iOS devices.
```html
<!-- iPhone Xs Max (1242px x 2688px) --> 
<link rel="apple-touch-startup-image" media="(device-width: 414px) and (device-height: 896px) and (-webkit-device-pixel-ratio: 3)" href="/apple-launch-1242x2688.png">
<!-- iPhone Xr (828px x 1792px) --> 
<link rel="apple-touch-startup-image" media="(device-width: 414px) and (device-height: 896px) and (-webkit-device-pixel-ratio: 2)" href="/apple-launch-828x1792.png">
<!-- iPhone X, Xs (1125px x 2436px) --> 
<link rel="apple-touch-startup-image" media="(device-width: 375px) and (device-height: 812px) and (-webkit-device-pixel-ratio: 3)" href="/apple-launch-1125x2436.png">
<!-- iPhone 8 Plus, 7 Plus, 6s Plus, 6 Plus (1242px x 2208px) --> 
<link rel="apple-touch-startup-image" media="(device-width: 414px) and (device-height: 736px) and (-webkit-device-pixel-ratio: 3)" href="/apple-launch-1242x2208.png">
<!-- iPhone 8, 7, 6s, 6 (750px x 1334px) --> 
<link rel="apple-touch-startup-image" media="(device-width: 375px) and (device-height: 667px) and (-webkit-device-pixel-ratio: 2)" href="/apple-launch-750x1334.png">
<!-- iPad Pro 12.9" (2048px x 2732px) --> 
<link rel="apple-touch-startup-image" media="(device-width: 1024px) and (device-height: 1366px) and (-webkit-device-pixel-ratio: 2)" href="/apple-launch-2048x2732.png">
<!-- iPad Pro 11” (1668px x 2388px) --> 
<link rel="apple-touch-startup-image" media="(device-width: 834px) and (device-height: 1194px) and (-webkit-device-pixel-ratio: 2)" href="/apple-launch-1668x2388.png">
<!-- iPad Pro 10.5" (1668px x 2224px) --> 
<link rel="apple-touch-startup-image" media="(device-width: 834px) and (device-height: 1112px) and (-webkit-device-pixel-ratio: 2)" href="/apple-launch-1668x2224.png">
<!-- iPad Mini, Air (1536px x 2048px) --> 
<link rel="apple-touch-startup-image" media="(device-width: 768px) and (device-height: 1024px) and (-webkit-device-pixel-ratio: 2)" href="/apple-launch-1536x2048.png">
```
