Phonegap Parse.com Plugin
=========================

Phonegap 3.0.0 plugin for Parse.com push service

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
phonegap local plugin add https://github.com/benjie/phonegap-parse-plugin


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
Phonegap > 3.0.0
