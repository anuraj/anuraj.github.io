---
layout: post
title: "Adding Chrome push notifications to web applications"
subtitle: "This post is about how to create and consume Chrome push notifications in Web Applications. Push notifications helps application developers to build more engaging apps for the users."
date: 2020-05-07 00:00:00
categories: [Desktop Notification,Push Notification]
tags: [Desktop Notification,Push Notification]
author: "Anuraj"
---
This post is about how to create and consume Chrome push notifications in Web Applications. Push notifications helps application developers to build more engaging apps for the users.

In this post I am using Chrome Desktop notification service to build push notifications. So first we need to build an Firebase Project. Once you create a Project, click on the Project settings. And create a Web App. Once you created the Web App, you will be able to see Firebase SDK Config. 

![Firebase SDK Config]({{ site.url }}/assets/images/2020/05/firebase_sdk_config.png)

You need this code to configure your app to receive the Push Notifications. 

Next click on the "Cloud Messaging" tab, and you need to create a Server Key, if there is none. The Server key is required to send the notification to client(s) from the Web Application.

Use the following code in the `_layout.cshtml`, which initializes the Firebase SDK.

{% highlight HTML %}
{% raw %}
<script src="https://www.gstatic.com/firebasejs/7.13.2/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/7.13.2/firebase-messaging.js"></script>
<script src="https://www.gstatic.com/firebasejs/7.13.2/firebase-analytics.js"></script>
<script>
    var firebaseConfig = {
        apiKey: "AIzaSyBBOuO1fUYzxKnTPMJYqSG_QUKb8mWtmZA",
        authDomain: "pushdemoapp-helloworld.firebaseapp.com",
        databaseURL: "https://pushdemoapp-helloworld.firebaseio.com",
        projectId: "pushdemoapp-helloworld",
        storageBucket: "pushdemoapp-helloworld.appspot.com",
        messagingSenderId: "285444955521",
        appId: "1:285444955521:web:59c7c6ead00bdd39"
    };
    firebase.initializeApp(firebaseConfig);
</script>
{% endraw %}
{% endhighlight %}

For applications to show browser notifications, first you need to request for permission. Here is the code which will request for the permission and get the token which helps receive the notification.

{% highlight Javascript %}
const messaging = firebase.messaging();
messaging.requestPermission().then(m => {
    messaging.getToken().then(t => {
        console.log(t);
    });
});
{% endhighlight %}

Right now I am configured this code in a button click, when user clicks on the button it will show a prompt like this.

![Firebase SDK Config]({{ site.url }}/assets/images/2020/05/notification_request.png)

This token need to be send to server and store it in the database.

Apps can manage push notifications in two ways, first notifications coming when user is using the application, second user is using the browser but not the application. So for the first scenario, users may not need to show the Desktop notification, instead it is better to display the in app, and if the user is not using the application, service worker can be used to display the notification.

So here is code which helps you to consume the code when the app is in Foreground - means user is using the application.

{% highlight Javascript %}
const messaging = firebase.messaging();
messaging.onMessage((payload) => {
    var obj = JSON.parse(payload.data.notification);
    console.log(obj);
});
{% endhighlight %}

I wrote this code after the initialize app inside the `_layout.cshtml`.

Next you need the serviceworker script, Firebase SDK expects a JavaScript file in the root directory with the name `firebase-messaging-sw.js`. Create a script file inside the `wwwroot` folder and put the following code.

{% highlight Javascript %}
importScripts('https://www.gstatic.com/firebasejs/7.12.0/firebase-app.js');
importScripts('https://www.gstatic.com/firebasejs/7.12.0/firebase-messaging.js');

var firebaseConfig = {
    apiKey: "AIzaSyBBOuO1fUYzxKnTPMJYqSG_QUKb8mWtmZA",
    authDomain: "pushdemoapp-helloworld.firebaseapp.com",
    databaseURL: "https://pushdemoapp-helloworld.firebaseio.com",
    projectId: "pushdemoapp-helloworld",
    storageBucket: "pushdemoapp-helloworld.appspot.com",
    messagingSenderId: "285444955521",
    appId: "1:285444955521:web:59c7c6ead00bdd39"
};

firebase.initializeApp(firebaseConfig);

const messaging = firebase.messaging();

messaging.setBackgroundMessageHandler(function (payload) {
    console.log('[firebase-messaging-sw.js] Received background message ', payload);
    var obj = JSON.parse(payload.data.notification);
    const notificationTitle = obj.title;
    const notificationOptions = {
        body: obj.body,
        image: obj.image,
    };

    return self.registration.showNotification(notificationTitle,
        notificationOptions);
});
{% endhighlight %}

Now we are ready to run the application. If you're not added the service worker, the `messaging.getToken()` method might fail.

Let's run the application and click on `Enable Notification` button. From the console, copy the token. This token have to store it in the database against the user.

Next use Postman to send notification. Open Postman, Select HTTP method as `POST` and use the following as the URL to send the request - `https://fcm.googleapis.com/fcm/send`. And in the headers select `Authorization`. And use value like `key=YOUR_SERVER_KEY`. And HTTP body like this.

{% highlight Javascript %}
{
    "data": {
        "notification": {
            "title": "New Blog post : Introduction to Microsoft DevSkim",
            "body": "This post is about Microsoft DevSkim. DevSkim is a framework of IDE extensions and Language analyzers that provide inline security analysis in the dev environment as the developer writes code.",
            "image": "https://dotnetthoughts.net/assets/images/2020/03/devskim_visualstudio.png",
            "url": "https://dotnetthoughts.net/introduction-to-devskim/",
            "icon":"https://dotnetthoughts.net/favicon.ico"
        }
    },
    "to": "TOKEN_FROM_BROWSER_CONSOLE"
}
{% endhighlight %}

In the body, the `data` and `to` properties are mandatory. Here is the notification displayed as Desktop Notification.

![Notification Display]({{ site.url }}/assets/images/2020/05/notification_display.png)

Now we have implemented Chrome Desktop notification with web application. We can enhance introducing topic subscriptions, so that instead of sending the data to individual tokens, we can send it topics. So which ever users subscribed for that topic the message will be delivering.

Happy Programming :)