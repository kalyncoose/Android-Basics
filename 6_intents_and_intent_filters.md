# Intents & Intent Filters
Video: https://www.youtube.com/watch?v=2hIY1xuImuQ&list=PLQkwcJG4YTCSVDhww92llY3CAnc_vUhsm&index=6
- An Intent is like an envelope that declares your intentions to another Android component, usually an activity (but could but other things like services, via broadcasts, etc.): https://developer.android.com/guide/components/intents-filters
- Two types of Intents
	- Explicit - targeted towards one specific app
	- Implicit - specify an action and have Android shows which apps can be used
- Android Manifest: defines what your app accepts or should do according to intents it receives
	- `AndroidManifest.xml`
- Creating an intent:
	- `Intent(...)`
	- `action` - used for intents against other apps
		- Example: launching the YouTube app
		  ```kotlin
		  Intent(Intent.ACTION_MAIN).also {
			  it.package = "com.google.android.youtube"
			  try {
				  startActivity(it)
			  } catch (e: ActivityNotFoundException) {
				  e.printStackTrack()
			  }
		  }
			```
		- `Intent.ACTION_MAIN` is a type of action, for example to start the `MainActivity` of the YouTube app
		- Use the `adb` to quickly see package names of installed apps on the device:
			- `adb shell`
			- `pm list packages` or `pm list packages | grep youtube`
	- `packageContext` - used for intents against your own app
		- Example: launching a second activity from a first activity
		  ```kotlin
		  Intent(applicationContext, SecondActivity::class.java).also {
			  startActivity(it)
		  }
			```
	- Implicit intent example: send an email
	  ```kotlin
	  val intent = Intent(Intent.ACTION_SEND).apply {
		  type = "text/plain"
		  putExtra(Intent.EXTRA_EMAIL, arrayOf("test@test.com"))
		  putExtra(Intent.EXTRA_SUBJECT, arrayOf("This is my subject"))
		  putExtra(Intent.EXTRA_TEXT, "This is the content of my email")
	  }
	  if (intent.resolveActivity(packageManager) != null) {
		  startActivity(intent)
	  }
		```
		- The `resolveActivity` function requires you to update the `<queries>` block in the `AndroidManifest` to specify what you need the intent for, example:
		  ```xml
		  <queries>
			  <intent>
				  <action android:name="android.intent.action.SEND" />
				  <data android:mimeType="text/plain" />
			  </intent>
		  </queries>
			```
- Use `intent.` to access info about an incoming intent during an activity
- Intent filters allow you to configure your app to receive specific types of incoming intents
	- Example: your app can receive an image share request
		- Update `AndroidManifest` so the activity can receive the intent:
		  ```xml
		  <activity>
			  <intent-filter>
				  <action android:name="action.intent.action.SEND" />
				  <category android:name="android.intent.category.DEFAULT" />
				  <data android:mimeType="image/*" />
			  </intent-filter>
		  </activity>
			```
	- Usually the other app will try to create a new instance of your app
	- Instead of using `intent.` at the top of your activity, change the launch mode in your `AndroidManifest`:
	  ```xml
	  <activity android:launchMode="singleTop">
	  </activity>
		```
	- This now means it will re-use an existing app instance that is already running
	- So instead of using `intent.` you can use the `onNewIntent` function:
	  ```kotlin
	  override fun onNewIntent(intent: Intent?) {
		  super.onNewIntent(intent)
		  val uri = intent?.getParcelableExtra(Intent.EXTRA_STREAM, Uri::class.java)
		  // ^- can use Android Studio to automatically create a verson check
	  }
		```
	- Now the `uri` needs to be accessible to the UI, so you can create a `ViewModel` to transfer this data to Compose (like an `Image` component)
	- Advanced: use an image dependency manager like Coil (image loading/caching)
		- `implementation "io.coil-kt:coil-compose:2.4.0"`
		- Commonly used in Compose projects
		- Simplifies converting `uri` from raw bytes to useable `Image` component, otherwise you would need to manually do that
		- Usage example: `AsyncImage(model = viewModel.uri, ...)`
