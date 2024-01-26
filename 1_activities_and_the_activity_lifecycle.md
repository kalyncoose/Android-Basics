# Activities & the Activity Lifecycle
Video: https://www.youtube.com/watch?v=SJw3Nu_h8kk&list=PLQkwcJG4YTCSVDhww92llY3CAnc_vUhsm&index=1&pp=iAQB
- Activity is a container for one or multiple screens in your app
	- Not just for screens, any unit of your app that your users can interact with
- `MainActivity`
	- The main entry point for your app
- Brief History
	- Used to be every screen was an activity
	- Then, when fragments were popular, screens were bundled together in one activity (like Profile, Profile Edit, etc.)
	- Now, with Jetpack Compose, you generally only need the `MainActivity` and the everything else is written in Kotlin and Compose
- Lifecycle: https://developer.android.com/guide/components/activities/activity-lifecycle
	- `onCreate()`: when the activity is first created
		- Good for initializing variables, setting up stuff
		- User does not see anything yet
		- More edge cases when this occurs due to advanced scenarios
	- `onStart()`: when the activity is starting
		- Good for splash screen/loading screen
		- User can see the app but cannot interact with it yet
	- `onResume()`: when the activity is interactable
		- User is fully in the app and can interact with it
	- `onPause()`: when another activity takes over user's focus
		- When another app comes into the foreground 
		- Can even be a dialog pop-up, not only activities
		- Good idea to save important data here as `onStop()` and `onDestroy()` aren't always guaranteed (edge cases)
	- `onStop()`: when the activity is no longer visible
		- When app is in the background, not actively being used
		- App process can be killed if system needs memory
	- `onRestart()`: when the activity was stopped but came back into focus
		- When the user brings the app back from the background into the foreground
	- `onDestroy()`: when the activity is being shutdown by the system

 ![activity_lifecycle](https://github.com/kalyncoose/Android-Basics/assets/65800415/5793d14f-98de-4c6b-b791-77d64f814aa2)

