WebImageLoader
==============

WebImageLoader is a library designed to take to hassle out of handling images on the web. It has the following features:

* Images are downloaded on a background thread pool and saved to disk and memory.
* Disk and memory cache size is configurable and can even be reconfigured on the fly.
* Separate thread do load images back from disk after being cached, reducing I/O bottlenecks on most phones.
* Reusing requests when the same image is requested multiple times.
* Respects cache-control and expires headers and will refetch images when they expire (using conditional get).
* Support image transformations which are also cached to disk and memory.
* Support to do synchronous fetches while still taking advantage of the cache. 
* Easy setup without singletons.
* Compatible with API level 7 and up.
* No additional dependencies.

Usage
=====

Use the builder to build an ImageLoader that suits your needs.

```java
// Get memory class of this device, exceeding this amount will throw an
// OutOfMemory exception.
ActivityManager am = (ActivityManager) getSystemService(Context.ACTIVITY_SERVICE);
int memClass = am.getMemoryClass();

// Use part of the available memory for memory cache.
final int memoryCacheSize = 1024 * 1024 * memClass / 8;

File cacheDir = new File(getExternalCacheDir(), "images");
ImageLoader imageLoader = new ImageLoader.Builder()
        .enableDiskCache(cacheDir, 10 * 1024 * 1024)
        .enableMemoryCache(memoryCacheSize);
```

Or use the provided Applications class for convienience and reasonable defaults (which you can override!)

```xml
<application android:name="com.webimageloader.ext.ImageLoaderApplication">
    <!-- Your app -->
</application>
```

Retrive your loader like this.

```java
ImageLoader imageLoader = ImageLoaderApplication.getLoader(context);
```

Loading images
--------------

Loading images is simple if you want it to be.

```java
// This will show a nice fadein when the image has loaded
new ImageHelper(context, imageLoader)
        .load(imageView, "http://example.com/image.png");
```

You can also use specify a loading and failure image

```java
new ImageHelper(context, imageLoader)
        .setLoadingResource(R.drawable.loading)
        .setErrorResource(R.drawable.error)
        .load(imageView, "http://example.com/image.png");
```

Loading images can also be done more explicit if needed.

```java
Bitmap b = loader.load(imageView, "http://example.com/image.png", new Listener<ImageView>() {
    @Override
    public void onSuccess(ImageView v, Bitmap b) {
        // Everything went well
        v.setImageBitmap(b);
    }

    @Override
    public void onError(ImageView v, Throwable t) {
        // Something went wrong
        Log.d("MyApp", "Failed to load image", t);
    }
});

// Did we get an image immediately?
if (b != null) {
    imageView.setImageBitmap(b);
}
```

Transformations
---------------

You can transform (and cache!) the images you get.

```java
final int width = 100;
final int height = 100;
        
Transformation t = new SimpleTransformation() {
    @Override
    public String getIdentifier() {
        // Pass a unique identifier for caching
        return "scale-" + width + "x" + height;
    }
    
    @Override
    public Bitmap transform(Bitmap b) {
        return Bitmap.createScaledBitmap(b, width, height, true);
    }
};

new ImageHelper(this, imageLoader)
        .load(imageView, "http://example.com/image.png", t);
```

Developed By
============

* Alexander Blom - <me@alexanderblom.se>