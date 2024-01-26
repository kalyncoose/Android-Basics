# Content Providers
Video: https://www.youtube.com/watch?v=IVHZpTyVOxU&list=PLQkwcJG4YTCSVDhww92llY3CAnc_vUhsm&index=11
- A Content Provider provides content from either the Android system itself or between Android components like other applications, services, or activities: https://developer.android.com/guide/topics/providers/content-provider-basics
	- Example: A custom phone app that loads the existing contacts from the Android system using a content provider or a custom calendar app that loads the existing calendar events
	- A content provider is a database-like mechanism to obtain data from various sources
	- Most commonly used content provider on Android is the `MediaStore` content provider, which gives you access to all kinds of images and video files
		- e.g. `MediaStore.Images`, `MediaStore.Video`, `MediaStore.Audio`
- Example app that uses a content provider to query and obtain images from the gallery:
  ```kotlin
  class MainActivity : ComponentActivity() {
	  private val viewModel by viewModels<ImageViewModel>()
  
	  override fun onCreate(...) {
		  super.onCreate(...)
		  // Request permission to read image gallery
		  ActivityCompat.requestPermissions(
			  this,
			  arraryOf(Manifest.permissions.READ_MEDIA_IMAGES), // API >= 33
			  0
		  )
		  // Define specific columns we want from the provider (like a SQL "select")
		  val projection = arrayOf(
			  MediaStore.Images.Media._ID,
			  MediaStore.Images.Media.DISPLAY_NAME,
		  )
		  
		  // Get a timestamp of yesterday
		  val millisYesterday = Calendar.getInstance().apply {
			  add(Calendar.DAY_OF_YEAR, -1)
		  }.timeInMillis
		  
		  // Define a condition for the query (like a SQL "where")
		  // ? is a selection arg (NOTE: don't put $ vars, that is injection vulnerable)
		  val selection = "${MediaStore.Images.Media.DATE_TAKEN} >= ?"
		  val selectionArgs = arrayOf(millisYesterday).toString()
		  val sortOrder = "${MediaStore.Images.Media.DATE_TAKEN} DESC"
		  
		  // Query the external image storage provider
		  contentResolver.query(
			  MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
			  projection,
			  selection,
			  selectionArgs,
			  sortOrder
		  )?.use { cursor -> 
			  // cursor is used to iterate over a large dataset
			  val idColumn = cursor.getColumnIndex(MediaStore.Images.Media._ID)
			  val nameColumn = cursor.getColumnIndex(
				  MediaStore.Images.Media.DISPLAY_NAME
			  )
			  
			  // Iterate over query results and store images
			  val images = mutableListOf<Image>()
			  while(cursor.moveToNext()) {
				  val id = cursor.getLong(idColumn)
				  val name = cursor.getString(nameColumn)
				  val uri = ContentUris.withAppendedId(
					  MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
					  id
				  )
				  images.add(Image(id, name, uri))
			  }
			  // Send the image list to the ImageViewModel
			  viewModel.updateImages(images)
		  }
		  // ...
		  setContent {
			  LazyColumn(
				  modifier = Modifier.fillMaxSize()
			  ) {
				  items(viewModel.images) { image ->
					  Column(
						  modifier = Modifier.fillMaxWidth(),
						  horizontalAlignment = CenterHorizontally,
					  ) {
						  // implementation "io.coil-kt:coil-compose:2.4.0"
						  AsyncImage(
							  model = image.uri,
							  contentDescription = null
						  )
						  Text(text = image.name)
					  }
				  }
			  }
		  }
	  }
	  
	  
	  // ...
	  data class Image(
		  val id: Long,
		  val name: String,
		  val uri: Uri
	  )
  }
	```
- Now we can create a `ViewModel` to manage the image results from the query:
  ```kotlin
  class ImageViewModel: ViewModel() {
	  var images by mutableStateOf(emptyList<Image>())
		  private set
		  
	  fun updateImages(images: List<Image>) {
		  this.images = images
	  }
  }
	```
- Use the new `ImageViewModel` in the `MainActivity`
- Add permissions request in `AndroidManifest`:
  ```xml
  <manifest>
	  <uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
	  <!-- ... -->
  </manifest>
	```
- Note that in general permission requests are fairly complex and involve varying Android API levels that you need to account for
- Advanced scenario: create a custom content provider for your app so other apps can access your app's internal data storage (files, database values, etc.):
	- Example:
	  ```kotlin
	  import android.content.ContentProvider
	  
	  class MyContentProvider: ContentProvider() {
		  override fun onCreate(): Boolean {
			  // logic...
		  }
		  override fun query(
			  // args...
		  ): Cursor? {
			  // logic...
		  }
		  // override fun getType, insert, delete, update, etc.
	  }
		```
	- Then you would need to register the provider in your `AndroidManifest`:
	  ```xml
	  <application>
		  <provider
			  android:authorities="..."
			  android:name="MyContentProvider"
		  />
		  <!-- ... -->
	  </application>
		```
