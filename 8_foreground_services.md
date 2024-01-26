# Foreground Services
Video: https://www.youtube.com/watch?v=YZL-_XJSClc&list=PLQkwcJG4YTCSVDhww92llY3CAnc_vUhsm&index=8
- A service is an Android component like an activity but it does not have an interface and it runs in the background
- There are types of services
	- Background services: The user is not aware that the service is running and that your app is doing something
		- Cannot be guaranteed run indefinitely because the OS may decide to destroy the service for memory
		- Work Manager handles these services
	- Foreground services: The user is aware that the app is running (e.g. live notification like a music player): https://developer.android.com/develop/background-work/services/foreground-services
		- Can be guaranteed that they run because the user sees a notification (the OS knows the user wants the service to keep running)
- A foreground service is just a class:
  ```kotlin
  import android.app.Service
  
  class RunningService: Service() {
	  override fun onBind(intent: Intent?): IBinder? {
		  return null
	  }
	  
	  override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
		  when(intent?.action) {
			  Actions.START.toString() -> start()
			  Actions.STOP.toString() -> stopSelf()
		  }
		  return super.onStartCommand(intent, flags, startId)
	  }
	  
	  private fun start() {
		  // Example of an exercise app persistent notification
		  val notification = NotificationCompat.Builder(this, "running_channel")
		  .setSmallIcon(R.drawable.ic_launcher_foreground)
		  .setContentTitle("Run is active")
		  .setContentText("elapsed time: 00:50")
		  .build()
		  startForeground(1, notification)
	  }
	  
	  enum class Actions {
		  START,
		  STOP
	  }
  }
	```
	- `IBinder` is used to create a "bound" service
		- You can have one active instance and multiple components can connect to that single instance, communicate to each other, receive events, etc.
		- Useful if you want to have multiple apps communicate to each other
		- Downside: you cannot use Work Manager if you use this pattern
		- Return `null` from `onBind` to disable this option
	- `onStartCommand` is triggered when another Android component sends an intent to this running service
		- Can process the intent with whatever intent action you want, e.g. `START`, `STOP`
		- You can use `stopSelf()` to have the service automatically stop itself (without writing the stop method yourself)
		- You can write your own `start()` function to have the logic your service should run
	- **Important**: To make this a foreground service, a persistent notification is needed
		- To do this, use `startForeground()` which takes an `id` and `notification`
			- `id` needs to be at least `1`
			- Notification can be updated whenever
		- Use `NotificationCompat.Builder()` to build a custom notification
			- Needs `context` from the service class itself or elsewhere
				- Using `this` means the notification only lives as long as the service does
			- Needs a notification `channelId` string which is like a category for notifications
				- This is where you can setup notification categories that users can later disable in settings
		- The `channelId` must be created on the app's `onCreate` before this notification channel works:
		  ```kotlin
		  import android.app.Application
		  
		  class RunningApp: Application() {
			  override fun onCreate() {
				  super.onCreate()
				  // Channels came in Android Oreo so you can add a version check
				  if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
					  val channel = NotificationChannel(
						  "running_channel", // channelId
						  "Running Notifications" // Label to be shown in settings
						  NotificationManager.IMPORTANCE_HIGH // notification rank
					  )
					  val notificationManager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
					  notificationManager.createNotificationChannel(channel)
				  }
			  }
		  }
			```
		- Then you need to define a service in your `AndroidManifest`:
		  ```xml
		  <manifest>
		  <!-- ... -->
		  <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
		  <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
			  <application>
			  <!-- ... -->
			  <service android:name=".RunningService" />
			  </application>
		  <!-- ... -->
			```
		- Service may need additional permissions defined as well (e.g. camera, location, mic)
		- Need to request permission for `POST_NOTIFICATION` in your app's `onCreate`
		- Need to attach the `.RunningService` to your `<application>` in `AndroidManifest`
		- Now you can use buttons to start/stop the foreground service via intents:
		  ```kotlin
		  // Start button
		  Button(onClick = {
			  Intent(applicationContext, RunningService::class.java).also {
				  it.action = RunningService.Actions.START.toString()
				  startService(it)
			  }
		  }) {
			  Text(text = "Start run")
		  }
		  
		  // Stop button
		  Button(onClick = {
			  Intent(applicationContext, RunningService::class.java).also {
				  it.action = RunningService.Actions.STOP.toString()
				  startService(it) // also used to send the STOP intent
			  }
		  }) {
			  Text(text = "Stop run")
		  }
			```
- There are different types of foreground services:
	- Use the `android:foregroundServiceType=""` property on `<service>`
		- Examples: `camera`, `connectedDevice`, `dataSync`, `location`, `mediaPlayback`, `mediaProjection`, `microphone`, `phoneCall`, etc.
		- Required when you want to use these permissions (default foreground service is restricted from using these unless you specify them)
- As long as your foreground service is active, then your app is also active
	- Closing the app does not stop the foreground service, instead it is running in the background
	- So you don't need to specify all the code in the service class, all other code in the app will still be alive and available to run
		- Suggestion is to use the service just for communication and updating the notification, keep the service code minimal (use separate classes if you wanted to track a user's time elapsed or location, etc.)
