Cordova Parse.com Plugin
=========================

Cordova 3.x plugin for Parse.com push service

Using [Parse.com's](http://parse.com) REST API for push requires the installation id, which isn't available in JS

This plugin exposes the four native Android and iOS API push services to JS:
* <a href="https://www.parse.com/docs/android/api/com/parse/ParseInstallation.html#getInstallationId()">getInstallationId</a>
* <a href="https://www.parse.com/docs/android/api/com/parse/PushService.html#getSubscriptions(android.content.Context)">getSubscriptions</a>
* <a href="https://www.parse.com/docs/android/api/com/parse/PushService.html#subscribe(android.content.Context, java.lang.String, java.lang.Class, int)">subscribe</a>
* <a href="https://www.parse.com/docs/android/api/com/parse/PushService.html#unsubscribe(android.content.Context, java.lang.String)">unsubscribe</a>

The iOS part also exposes a service to set the badge, both locally and remotely on Parse:
* <a href="https://www.parse.com/docs/ios/api/Classes/PFInstallation.html#//api/name/badge">PFInstallation badge</a>

Installation
------------
cordova plugin add https://github.com/azicchetti/phonegap-parse-plugin.git

In order to use the plugin on Android, you've to properly setup the AndroidManifest.xml and do a couple of other simple things:

* follow <a href="https://parse.com/apps/quickstart#parse_push/android/existing">this guide</a> to modify the AndroidManifest. If you update the Parse SDK you may have to add more things to the manifest.
You'll probably have to add these lines before the closing `</application>` tag:

```
	<service android:name="com.parse.PushService" />
        <receiver android:name="com.parse.ParseBroadcastReceiver">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
                <action android:name="android.intent.action.USER_PRESENT" />
            </intent-filter>
        </receiver>
	<receiver android:name="com.parse.GcmBroadcastReceiver"
	    android:permission="com.google.android.c2dm.permission.SEND">
	  <intent-filter>
	    <action android:name="com.google.android.c2dm.intent.RECEIVE" />
	    <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
	 
	    <!--
	      IMPORTANT: Change "com.parse.starter" to match your app's package name.
	    -->
	    <category android:name="com.parse.starter" />
	  </intent-filter>
	</receiver>
```

* make sure you have the following permissions set in the manifest:

```
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="android.permission.VIBRATE" />
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
<uses-permission android:name="android.permission.GET_ACCOUNTS" />
<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
 
<!--
  IMPORTANT: Change "com.parse.starter.permission.C2D_MESSAGE" in the lines below
  to match your app's package name + ".permission.C2D_MESSAGE".
-->
<permission android:protectionLevel="signature"
    android:name="com.parse.starter.permission.C2D_MESSAGE" />
<uses-permission android:name="com.parse.starter.permission.C2D_MESSAGE" />
```

* to avoid the dreaded error:
	"java.lang.RuntimeException: Unable to start receiver com.parse.ParseBroadcastReceiver: java.lang.RuntimeException: applicationContext is null. You must call Parse.initialize(context, applicationId, clientKey) before using the Parse library."
that'll make the application close unexpectedly, you have to edit the `<application>` tag in the AndroidManifest.xml and add `android:name="MyMainApp"` (you can change the MyMainApp name to something more meaningful):

```
<application android:hardwareAccelerated="true" android:icon="@drawable/icon" android:label="@string/app_name" android:name="MyMainApp">
```

* last step! You have to create the MyMainApp.java file into the proper directory. If your package is, for instance, "com.parse.starter", the file will reside into com/parse/starter/MyMainApp.java and its content will be (remember, you can find the keys you need into the settings page of your app on Parse):

```
package com.parse.starter;

import android.app.Application;
import com.parse.Parse;

public class MyMainApp extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        Parse.initialize(this, "your application id", "your client key");
    }
}
```

Usage
-----
```
<script type="text/javascript>
	parsePlugin.initialize(appId, clientKey, function() {
		alert('success');
	}, function(e) {
		alert('error');
	});
  
	parsePlugin.getInstallationId(function(id) {
		alert(id);
	}, function(e) {
		alert('error');
	});
	
	parsePlugin.getSubscriptions(function(subscriptions) {
		alert(subscriptions);
	}, function(e) {
		alert('error');
	});
	
	parsePlugin.subscribe('SampleChannel', function() {
		alert('OK');
	}, function(e) {
		alert('error');
	});
	
	parsePlugin.unsubscribe('SampleChannel', function(msg) {
		alert('OK');
	}, function(e) {
		alert('error');
	});

	parsePlugin.setInstallationBadge(5);
</script>
```


Notes
-----

The plugin uses the new iOS 8 api to register for remote notifications, but it's still compatible
with previous iOS versions.

This plugin won't build any notification interface for you on iOS or Android when a push is received.
Please use to the push plugin for cordova if you want this feature.
I'm planning to implement this behavior once I find some spare time. 

On iOS, however, an alert is still shown when a notification is received and your app is inactive
(in background). To mimic this behavior when the app is in foreground, just uncomment the handlePush
call in src/ios/CDVParsePlugin.m

```
- (void)swizzled_application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo
{
    // Call existing method
    [self swizzled_application:application didReceiveRemoteNotification:userInfo];
    [PFPush handlePush:userInfo];
}
```

Compatibility
-------------
Cordova > 3.0.0
