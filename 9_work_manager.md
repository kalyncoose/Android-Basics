# Work Manager
Video: https://www.youtube.com/watch?v=A2JetouoNSc&list=PLQkwcJG4YTCSVDhww92llY3CAnc_vUhsm&index=9
- Work Manager offers the ability to execute tasks and functionality reliably: https://developer.android.com/topic/libraries/architecture/workmanager
	- Tasks that the user does not need to be directly aware of
	- Can still show a notification if desired, but not required like a foreground service
	- Examples: app needs to sync data in the background periodically, upload data to a server on a slow connection (user can close app and not stop the upload), etc.
- Example: Background service that compresses images, app that renders them once they're compressed
	- Use a background service because it is unpredictable to compress images since they vary data and size and may hit a desired compression ratio at different timings
	- This background service will need permission access to the file system
- A "Worker" is a process that Work Manager controls and runs
- To use Work Manager in a project:
	- `implementation "androidx.work:work-runtime-ktx:2.8.1"`
- To process image data, Coil is recommended:
	- `implementation "io.coil-kt:coil-compose:2.4.0"`
- To convert information from a work request to observable Compose state:
	- `implementation "androidx.compose.runtime:runtime-livedata:1.4.3"`
- For the example app, we would setup an `<intent-filter>` to allow sending an image
- A "Worker" is just a class:
  ```kotlin
  import androidx.work.CoroutineWorker
  
  class PhotoCompressionWorker(
	  private val appContext: Context,
	  private val params: WorkerParameters
  ): CoroutineWorker(appContext, params) {
	  // Logic for your worker to run
	  override suspend fun doWork(): Result {
		  // Makes sure this runs in an optimized thread
		  return withContext(Dispatch.IO) {
			  val stringUri = params.inputData.getString(KEY_CONTENT_URI)
			  val compressionThresholdInBytes = params.inputData.getLong(
				  KEY_COMPRESSION_THRESHOLD,
				  0L // default value
			  )
			  val uri = Uri.parse(stringUri)
			  val bytes = appContext.contentResolver.openInputStream(uri)?.use {
				  it.readBytes()
			  } ?: return@withContext Result.failure()
			  
			  // Convert the byte array to a bitmap
			  var bitmap.BitmapFactory.decodeByteArray(bytes, 0, bytes.size)
			  
			  // Compress image until desired size, retain highest quality
			  var outputBytes: ByteArray
			  var quality = 100
			  do {
				  val outputStream = ByteArrayOutputStream()
				  outputStream.use { outputStream ->
					  bitmap.compress(Bitmap.CompressFormat.JPEG, quality, outputStream)
					  outputBytes = outputStream.toByteArray()
					  // Decrease quality by 10% until image is desired size
					  quality -= (quality * 0.1).roundToInt()
				  }
			  } while(outputBytes.size > compressionThresholdInBytes && quality > 5)
			  
			  // Save compressed image to file storage
			  val file = File(appContext.cacheDir, "${params.id}.jpg")
			  file.writeBytes(outputBytes)
			  
			  // Return success with the image path
			  return Result.success(
				  workDataOf(
					  KEY_RESULT_PATH to file.absolutePath
				  )
			  )
		  }
		  
	  }
	  
	  companion object {
		  const val KEY_CONTENT_URI = "KEY_CONTENT_URI"
		  const val KEY_COMPRESSION_THRESHOLD = "KEY_COMPRESSION_THRESHOLD"
		  const val KEY_RESULT_PATH = "KEY_RESULT_PATH"
	  }
  }
	```
	- There are different types of workers, but we'll just use a `CoroutineWorker`
	- Override the `doWork()` function to implement your worker's logic
	- Return a `Result` to let the worker know if the processing was successful, failed, or should run later
	- Worker has an incoming data limit (e.g. 10 KB)
		- So instead of sending raw image data over bytes you can send the `uri` address to download and process
	- Every worker has a unique ID when running, which is available at `params.id`
	- You can use `setForeground()` if you want to show a notification while the service runs
- Now to utilize the `PhotoCompressionWorker` class:
	- Set `android:launchMode="SINGLE_TOP"` in your `<activity>` in `AndroidManifest`
	- Use the `override onNewIntent()` in your app:
	  ```kotlin
	  // Get the work manager when the app starts
	  private lateinit var workManager: WorkManager
	  override fun onCreate(...) {
		  super.onCreate(...)
		  // Set the work manager instance
		  workManager = WorkManager.getInstance(applicationContext)
	  }
	  // ...

	  // Run this logic on a new intent
	  override fun onNewIntent(intent: Intent?) {
		  super.onNewIntent(intent)
		  val uri = if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
			  intent?.getParcelableExtra(Intent.EXTRA_STREAM, Uri::class.java)
		  } else { // older version does not require uri class
			  intent?.getParcelableExtra(Intent.EXTRA_STREAM)
		  } ?: return // no uri
		  
		  // Setup the work request
		  val request = OneTimeWorkRequestBuilder<PhotoCompressionWorker>()
		  .setInputData(
			  workDataOf(
				  PhotoCompressionWorker.KEY_CONTENT_URI to uri.toString(),
				  PhotoCompressionWorker.KEY_COMPRESSION_THRESHOLD to 1024*20L // 20KB
			  )
		  )
		  .setConstraints(Constraints(
			  requiresStorageNotLow = true
		  ))
		  .build()
		  
	  // Queue the work request
	  workManager.enqueue(request)
	  }
		```
	- Can use a `PeriodicWorkRequestBuilder` if you need the request to run on time intervals
	- Can use `setConstraints` on the work request for things like `requiresCharging`, `requiresDeviceIdle`, `requiresBatteryNotLow`, `requiresStorageNotLow`, etc.
	- Can define multiple workers, can chain the workers so that one executes and passes data to another worker, set back-off criteria, initial delay, schedule, so there's much more available patterns
	- When queueing work requests you can use `enqueue()`, `enqueueUniqueWork()` to ensure only one request runs at a time, and `enqueueUniquePeriodicWork()` to schedule requests
- Now we can create a `ViewModel` to bridge the data of the resulting compressed image:
  ```kotlin
  class PhotoViewModel: ViewModel() {
	  var uncompressedUri: Uri? by mutableStateOf(null)
		  private set
	  var compressedBitmap: Bitmap? by mutableStateOf(null)
		  private set
	  var workId: UUID? by mutableStateOf(null)
		  private set
	  
	  fun updateUncompressedUri(uri: Uri?) {
		  uncompressedUri = uri
	  }
	  fun updateCompressedBitmap(bmp: Bitmap?) {
		  compressedBitmap = bmp
	  }
	  fun updateWorkId(uuid: UUID?) {
		  workId = uuid
	  }
  }
	```
- And use the `PhotoViewModel` in our app's Compose:
  ```kotlin
  class MainActivity : ComponentActivity() {
	  private val viewModel by viewModels<PhotoViewModel>()
	  
	  // override fun onCreate...
	  setContent {
		  // Get the latest worker's data
		  val workerResult = viewModel.workId?.let { id ->
			  workManager.getWorkInfoByIdLiveData(id).observeAsState().value
		  }
		  // Runs when outputData changes
		  LaunchedEffect(key1 = workerResult?.outputData) {
			  if(workerResult?.outputData != null) {
				  // Get the image filepath from the successful worker
				  val filepath = workerResult.outputData.getString(
					  PhotoCompressionWorker.KEY_RESULT_PATH
				  )
				  // Load the image file into viewModel
				  filepath?.let {
					  val bitmap = BitmapFactory.decodeFile(it)
					  viewModel.updateCompressedBitmap(bitmap)
				  }
			  }
		  }
		  // ...
		  Column(
			  modifier = Modifier.fillMaxSize(),
			  verticalArrangement = Arrangement.Center,
			  horizontalArrangement = Alignment.CenterHorizontally
		  ) {
			  viewModel.uncompressedUri?.let {
				  Text(text = "Uncompressed photo:")
				  AsyncImage(model = it, contentDescription = null)
			  }
			  Spacer(modifier = Modifier.height(16.dp))
			  viewModel.compressedBitmap?.let {
				  Text(text = "Compressed photo:")
				  Image(bitmap = it.asImageBitmap(), contentDescription = null)
			  }
		  }
	  }
	  
	  
	  // Use the new viewModel inside onNewIntent
	  override fun onNewIntent(intent: Intent?) {
		  // val uri...
		  viewModel.updateUncompressedUri(uri)
		  // val request...
		  viewModel.updateWorkId(request.id)
	  }
  }
	```
	- `getWorkInfoByIdLiveData()` is lifecycle aware and helps prevent leaks
