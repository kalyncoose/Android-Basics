# Android Context
Video: https://www.youtube.com/watch?v=YdnM2ZvrIFM&list=PLQkwcJG4YTCSVDhww92llY3CAnc_vUhsm&index=4
- Context is used all the time in Android apps: https://developer.android.com/reference/android/content/Context
- A Context in Android Studio is just an instance of a class
	- Comes from `android.content`
	- Is a bridge for your Android app to the rest of the Android operating system
	- Always needed when your app needs to communicate to the system or other apps
	- Examples: 
		- when accessing resources like images
		- writing to the file system database
		- launching other apps
- Context is basically just a super class:
	- Has a sub class of `ContextWrapper` which has sub classes of `Activity`, `Service`, `Application`
  <img width="300" alt="00s1L" src="https://github.com/kalyncoose/Android-Basics/assets/65800415/b6d2de67-3465-4a87-93a3-fc1cf3d11aa2">
- Each context has a lifetime (e.g. an activity is context, when it is destroyed so is its context)
- Contexts can lead to memory leaks or freezes (be careful with these)
	- Android Studio has can show a yellow warning
- Application context outlives activities (as long as app is alive)
	- Does not mean you can replace Activity context with it always
	- Is not connected to apps UI, because its running even if app is in background
- Activity context is needed when you want to request device permissions for example
	- Example: request permission with `ActivityCompat.requestPermissions(activityContext...)`
	- Connects to UI, e.g. permission dialog pops up over the current activity
