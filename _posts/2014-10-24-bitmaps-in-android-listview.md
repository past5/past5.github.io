---
layout: post
title: "Bitmaps in Android ListView"
tagline: "loading bitmaps to display pictures while maintaining smooth Android ListView performance"
description: "Android tutorial on how to load bitmaps with LRUCache and AsyncTask to display pictures while maintaining smooth ListView performance"
author: Vsevolod Geraskin
category: [tutorials]
tags: [android, java, code]
excerpt: "A simple task such as displaying your gallery photos in a ListView component can lead to a significant loss of performance in your Android application, to the point where it would render the application
unusable.  Worse yet, the application might run out of memory and crash while loading bitmaps.  Just think about user quickly scrolling through a list of hundreds of photos each the size of 3-10 megabytes. 
Thus, to display pictures in ListView is not straightforward and requires several performance considerations."
---
{% include JB/setup %}

#### Photocalypse
A simple task such as displaying your gallery photos in a ListView component can lead to a significant loss of performance in your Android application, to the point where it would render the application
unusable.  Worse yet, the application might run out of memory and crash while loading bitmaps.  Just think about user quickly scrolling through a list of hundreds of photos each the size of 3-10 megabytes. 
Thus, to display pictures in ListView is not straightforward and requires several performance considerations.

#### Caching Bitmaps
It doesn't make sense for us to keep default ListView behavior which continuously frees up and loads our bitmaps when our users scroll back and forth through the list. 
Caching allows us to store bitmaps in memory and quickly retrieve them on consecutive views while scrolling.  Thankfully, Android provides us with 
[LRUCache](http://developer.android.com/training/displaying-bitmaps/cache-bitmap.html) that helps us cache our bitmaps.  Thus, we can create a fairly simple class for our LRUCache.

{% highlight Java %}
import android.graphics.Bitmap;
import android.support.v4.util.LruCache;

public class BitmapCache {
	private static LruCache<Integer, Bitmap> MemoryCache = null;
	
	public static void InitBitmapCache() {
		// Get max available VM memory, exceeding this amount will throw an
		// OutOfMemory exception. Stored in kilobytes as LruCache takes an
		// int in its constructor
	
		if (MemoryCache==null) {
			final int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);
			// use 1/4th of the available memory for this memory cache.
			
			final int cacheSize = maxMemory / 4;
			MemoryCache = new LruCache<Integer, Bitmap>(cacheSize) {c
				@Override
				protected int sizeOf(Integer key, Bitmap bitmap) {
					// The cache size will be measured in kilobytes rather than
					// number of items.
					return bitmap.getByteCount() / 1024;
				}
			};
		}
	}
	
	public static void addBitmapToMemoryCache(Integer key, Bitmap bitmap) {
		MemoryCache.put(key, bitmap);
	}

	public static Bitmap getBitmapFromMemCache(Integer key) {
		return MemoryCache.get(key);
	}
}
{% endhighlight %}

The above code allows creation of LruCache with a quarter of available memory, and adding and retrieving bitmaps by key value, which is a position of an image in our ListView.

#### Resampling Bitmaps
Loading hundreds of full-sized image bitmaps would quickly cause our application to crash from lack of memory.  Oftentimes, displaying image at fraction of the original size in a list is sufficient.
The functions below help us resize the image and resample the bitmap before using it further.
 
{% highlight Java %}
//resample Bitmap to prevent out-of-memory crashes
private Bitmap decodeSampledBitmapFromString(int reqWidth, int reqHeight) {
	Bitmap bitmap;
	
	//decode File and set inSampleSize
	final BitmapFactory.Options options = new BitmapFactory.Options();
	options.inJustDecodeBounds = true;
	BitmapFactory.decodeFile(m_photoPath, options);
	options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);
	
	// decode File with inSampleSize set
	options.inJustDecodeBounds = false;
	bitmap = BitmapFactory.decodeFile(m_photoPath, options);
	return bitmap;
}

//calculate bitmap sample sizes
private int calculateInSampleSize(BitmapFactory.Options options, int reqWidth, int reqHeight) {
	final int height = options.outHeight;
	final int width = options.outWidth;
	int inSampleSize = 1;
	
	if (height > reqHeight || width > reqWidth) {
		if (width > height) {
			inSampleSize = Math.round((float) height / (float) reqHeight);
		} else {
			inSampleSize = Math.round((float) width / (float) reqWidth);
		}
	}
	return inSampleSize;
}
{% endhighlight %}

Basically, `calculateInSampleSize` takes original image width or height and divides them by the target width or height. Returned inSampleSize then gives us a divisor to create processed bitmaps that are a 
fraction of the original size.  Finally, `decodeSampledBitmapFromString` reads our bitmap from provided image path and returns resized smaller bitmap.

#### Loading Asynchronously
Initially, we want to load all our bitmaps in a separate thread or process, so we do not freeze our UI thread.  To achieve that, we can use 
[AsyncTask](http://developer.android.com/training/improving-layouts/smooth-scrolling.html).  In our case, this is also where we want to re-sample our bitmaps and place the images into cache. Therefore,
our AsyncTask would look like this:

 
{% highlight Java %}
public class ImageLoaderTask extends AsyncTask<Integer, String, Bitmap> {
	...
	
	@Override
	protected Bitmap doInBackground(Integer... params) {
		//re-sample sample image in the background to 200x200
		Bitmap bitmap = decodeSampledBitmapFromString(200,200);
	}
	
	//set photoView and holder
	protected void onPostExecute(Bitmap bitmap) {
		if (bitmap != null) {
			BitmapCache.addBitmapToMemoryCache(m_position, bitmap);
			
			// to do: set imageView 
		}
	}
	
	private Bitmap decodeSampledBitmapFromString(int reqWidth, int reqHeight) {
		...
	}
	private int calculateInSampleSize(BitmapFactory.Options options, int reqWidth, int reqHeight) {
		...
	}

}
{% endhighlight %}

#### Putting it all together
In the main activity, we would initialize our bitmap cache in `onCreate()` method:

{% highlight Java %}
protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	setContentView(R.layout.activity_main);
	
	...
	
	//initialize bitmap cache
	BitmapCache.InitBitmapCache();
}
{% endhighlight %}

Then, in our ListView adapter, we would add `loadBitmap()` function to either set ImageViews bitmaps from cache or to start a new AsyncTask.

{% highlight Java %}
public class GalleryAdapter extends BaseAdapter {
	...
	
	@Override
	public View getView(int position, View view, ViewGroup parent) {
		...
		loadBitmap(photo, position, holder);
	}
	
	// load Bitmap either from our cache or asynchronously
	public void loadBitmap(ImageView photo, Integer position, ViewHolder holder) {
		Bitmap bitmap = null;
		bitmap = BitmapCache.getBitmapFromMemCache(position);
		
		if (bitmap != null) {
			photo.setImageBitmap(bitmap);
		} else {
			new ImageLoaderTask(photo, m_uris.get(position).getPath(),
								m_adapter, position, holder).executeOnExecutor(
								AsyncTask.THREAD_POOL_EXECUTOR, (Integer[]) null);
		}
	}
{% endhighlight %}

Also, we could also do other things to optimize performance, such as implementing [ViewHolder Pattern](http://developer.android.com/training/improving-layouts/smooth-scrolling.html) to recycle views 
in a static class.   In our example, ViewHolder will hold two things: ImageView and its position in ListView.

{% highlight Java %}
// ViewHolder class
public static class ViewHolder {
	public ImageView picture;
	public int position;
}
{% endhighlight %}

Finally, Android has a limit of 128 simultaneous AsyncTasks and gives an error if we exceed that limit.  This error is entirely possible when loading a thousand bitmaps in a ListView. 
One way to prevent the error from happening and to manage memory is to implement a static class to keep and control current AsyncTask count.

{% highlight Java %}
//static class to count number of concurrent asynctasks
public final class SyncCounter {
	private static int i = 0;

	public static synchronized void inc() {
		i++;
	}

	public static synchronized void dec() {
		i--;
	}

	public static synchronized int current() {
		return i;
	}
} 
{% endhighlight %}

Our AsyncTask class could then increment the count in the AsyncTask constructor and decrement in `onPostExecute()` method.  We would check the current count against arbitrary maximum 
prior to launching `new ImageLoaderTask(...)` in List Adapter `loadBitmap(...)` method.

#### Complete Source

And, of course, all of this might not make sense without a working code, which you can access on GitHub in my [gallery-in-listview repository](https://github.com/past5/gallery-in-listview).


