# Resources & Qualifiers
Video: https://www.youtube.com/watch?v=vj1ZdUfPlJM&list=PLQkwcJG4YTCSVDhww92llY3CAnc_vUhsm&index=5
- Resources refer to things your app needs, examples: https://developer.android.com/guide/topics/resources/providing-resources
	- pictures
	- vector graphics
	- localized strings
- Resources are stored under the `res` folder
	- `drawable` - everything visual, pictures/vector graphics (vectors convert to xml)
	- `mipmap` - stores the app launcher icons in various sizes
	- `values` - store xml files for defining things like colors, strings, themes
	- `xml` - rules files, advanced configuration stuff
- To use a resource in Compose: `val drawable = resources.getDrawable(R.drawable.kermit)`
	- Every resource has an `id` that is an integer
	- Use the `R` from your package name
		- The other `R` are resources from other installed packages
	- More commonly, use a resource directly in a Compose component, like a drawable resource in an `Image` component:
		- `Image(painter = painterResource(id = R.drawable.kermit))`
	- Example of color resource:
		- `val color = colorResource(id = R.color.purple)`
		- `getColor(R.color.purple)`
- Qualifiers: https://developer.android.com/guide/topics/resources/providing-resources#BestMatch
	- Ensures resources are only used in specific configurations
		- Example: `(v24)` is a qualifier, meaning the device needs to run Android API 24 to be able to use the resource
	- Can create custom qualifications on resources by creating a `Resource File`
		- More qualifiers available like country code, locale, layout direction, night mode, etc.
		- Can combine multiple qualifiers at the same time
	- Resolution qualifiers of the same resource so that the icons look good in all resolutions
		- e.g. `hdpi`, `mdpi`, `xhdpi`, `xxhdpi`, `xxxhdpi`, `anydpi-v26`
		- Common for the app's launcher icon `ic_launcher.webp`