# Broadcasts & Broadcast Receivers
Video: https://www.youtube.com/watch?v=HDVyFsFUuVg&list=PLQkwcJG4YTCSVDhww92llY3CAnc_vUhsm&index=7
- A Broadcast is similar to an intent, but instead of sending an intent to one app, you send it to many apps: https://developer.android.com/develop/background-work/background-tasks/broadcasts
	- Apps that receive a broadcast can silently handle it and not open an activity
	- Broadcasts are system-wide events that your app can send and receive
	- Events can be sent by the android system itself, your app, or other apps
	- Your app can register a Broadcast Receiver which can be triggered on incoming events
	- A Broadcast Receiver is just a class: `android.content.BroadcastReceiver`
	- Example: android device is fully booted up, sends a "boot completed" broadcast to all apps, your app can receive it and start a background sync, etc.
	- Example: your music player app can receive a broadcast from the phone app and pause the music so the user can focus on the incoming phone call
- Dynamic Receiver - receivers that only run while your app is running
- Reacting to android system broadcasts:
	- Example: react to when airplane mode is toggled
	  ```kotlin
	  class AirPlaneModeReceiver: BroadcastReceiver() {
		  override fun onReceive(context: Context?, intent: Intent?) {
			if (intent?.action == Intent.ACTION_AIRPLANE_MODE_CHANGED) {
				val isTurnedOn = Settings.Global.getInt(
					context?.contentResolver,
					Settings.Global.AIRPLANE_MODE_ON
				) != 0
				println("Is airplane mode enabled? $isTurnedOn")
			}
		  }
	  }

	// now register the receiver in your activity
	private val airPlaneModeReceiver = AirPlaneModeReceiver()
	registerReceiver(
		airPlaneModeReceiver,
		IntentFilter(Intent.ACTION_AIRPLANE_MODE_CHANGED)
	)
	
	// unregister the receiver when its not needed anymore (like onDestroy)
	override fun onDestroy() {
		super.onDestroy()
		unregisterReceiver(airPlaneModeReceiver)
	}
		```
- Static Receiver - used when your app is not running and needs to receive a broadcast
	- Has more restrictions, like the "device is booted" intent
	- Can drain device's battery life so that's why there are restrictions
	- To define a static receiver, add a `<receiver>` block to your `AndroidManifest`:
	  ```xml
		  <receiver android:name=".ExampleReceiverClass">
			  <intent-filter>
				  // ...
			  </intent-filter>
		  </receiver>
		```
- Sending a Broadcast
	- Example: Send a broadcast when a button is pressed in Compose
	  ```kotlin
	  Button(onClick = {
		  sendBroadcast(
			  Intent("TEST_ACTION")
			  // Sends custom action "TEST_ACTION" to all other apps
			  // Configure the intent further for other scenarios if desired
		  )
	  }) {
		  Text(text = "Send broadcast")
	  }
		```
- Receive a custom action in a broadcast:
  ```kotlin
  class TestReceiver: BroadcastReceiver() {
	  override fun onReceive(context: Context?, intent: Intent?) {
		if (intent?.action == "TEST_ACTION") {
			println("Received test intent!")
		}
	  }
  }
  // then register your TestReceiver in the app, etc.
	```
