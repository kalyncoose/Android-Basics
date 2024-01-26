# URIs (Unique Resource Identifier)
Video: https://www.youtube.com/watch?v=4Ob0plBL084&list=PLQkwcJG4YTCSVDhww92llY3CAnc_vUhsm&index=10
- A `uri` could be a path to a file, resource in the `res` folder, image or video, etc.
- There are four types of URIs:
	- Note that `android.net.Uri` combines all four types of URIs: https://developer.android.com/reference/android/net/Uri
	- A **resource URI** is one that points to a resource in `res`
		- Example: 
		  ```kotlin
		  val uri = Uri.parse("android.resource://$packageName/drawable/example")
		  val exampleBytes = contentResolver.openInputStream(uri)?.use { 
			  it.readBytes() 
		  }
			```
	- A **file URI** identifies a specific file
		- Each app has its own internal file storage
		- External storage in Android is the more shared area multiple apps can use
			- This requires a special permission for your app to use
		- Example: 
		  ```kotlin
		  val file = File(filesDir, "example.jpg")
		  FileOutputStream(file).use { 
			  it.write(exampleBytes)
		  }
		  val fileUri = file.toUri()
			```
			- The path of where the file is saved on the system
			- `filesDir` was used for internal app storage (from app `context`), so this file path is not accessible to other apps
	- A **content URI** accesses external storage data (e.g. image from image gallery)
		- Example: 
		  ```kotlin
		  val pickImage = rememberLauncherForActivityResult(
			  contract = ActivityResultContracts.GetContent(),
			  onResult = { contentUri ->
				  println(contentUri)
			  }
		  )
		  Button(onClick = {
			  pickImage.launch("image/*")
		  }) {
			  Text(text = "Pick image")
		  }
			```
		- By picking the file it is converted into a `contentUri` which is granted temporary permission for the app to use
		- If you save the `contentUri` to the device, it is not guaranteed that your app will continue to have permissions for that `contentUri`
			- Therefore it is not a reliable file path for long-term usage
			- Recommended pattern is to read the data and make a copy to save in your app's internal storage
	- A **data URI** stores encoded data inside the URI itself
		- Example:
		  ```kotlin
		  val dataUri = Uri.parse("data:text/plain;charset=UTF-8,Hello%20World")
			```
		- Not as common as the other three URI types
		- Usually the data is `base64` encoded
- Overall lesson is to be careful to acknowledge what type of URI you are working with at all times because there are different behaviors for each type
	- If you use them wrongly, you risk introducing bugs into your app's behavior
