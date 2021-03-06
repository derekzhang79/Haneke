Haneke
======
<img src="https://raw.github.com/hpique/Haneke/master/screenshot.png" width="75%" "alt="Screenshot" />

[![Build Status](https://travis-ci.org/hpique/Haneke.png)](https://travis-ci.org/hpique/Haneke)

A lightweight zero-config image cache for iOS. 

Haneke resizes images and caches the result on memory and disk. Everything is done in background, allowing for fast, responsive scrolling. Asking Haneke to load, resize, cache and display an *appropriately sized image* is as simple as:

```objective-c
[imageView hnk_setImageFromURL:url];
```

_Really._

##Features

* First-level memory cache using `NSCache`.
* Second-level LRU disk cache using the file system.
* Zero-config `UIImageView` category to use the cache, optimized for `UITableView` and `UICollectionView` cell reuse.
* Asynchronous and synchronous image retrieval.
* Background image resizing and file reading.
* Custom image transformations before and after resizing.
* Thread-safe.
* Automatic cache eviction on memory warnings or disk capacity reached.
* Preloading images from the disk cache into memory on startup.

##Add Haneke to your project

1. Add the [Haneke](https://github.com/hpique/Haneke/tree/master/Haneke) folder to your project.
2. Profit!

##UIImageView category

Haneke provides convenience methods for `UIImageView` with optimizations for `UITableView` and `UICollectionView` cell reuse. Images will be resized appropriately and cached in a shared cache.

```objective-c
// Setting a remote image
[imageView hnk_setImageFromURL:url];

// Setting a local image
[imageView hnk_setImageFromFile:path];

// Setting an image manually. Requires you to provide a key.
[imageView hnk_setImage:image withKey:key];
```

The above lines take care of:

1. If cached, retrieving an appropriately sized image (based on the `bounds` and `contentMode` of the `UIImageView`) from the memory or disk cache. Disk access is performed in background.
2. If not cached, loading the original image from web/disk/memory and producing an appropriately sized image, both in background. Remote images will be retrieved from the shared `NSURLCache` if available.
3. Setting the image and animating the change if appropriate.
4. Or doing nothing if the `UIImageView` was reused during any of the above steps.
5. Caching the resulting image.
6. If needed, evicting the least recently used images in the cache.


##Cache formats

The cache behavior can be customized by defining cache formats. Each image view has a default format and you can also define your own formats. A format is uniquely identified by its name.

### UIImageView format

Each image view has a default format created on demand. The default format is configured as follows:

* Size matches the `bounds` of the image view.
* Images will be scaled based on the `contentMode` of the the image view.
* Images can be upscaled if they're smaller than the image view.
* High compression quality.
* No preloading.
* Up to 10MB of disk cache.

Modifying this default format is discouraged. Instead, you can set your own custom format like this:

```objective-c
HNKCacheFormat *format = [[HNKCacheFormat alloc] initWithName:@"thumbnail"];
format.size = CGSizeMake(320, 240);
format.scaleMode = HNKScaleModeAspectFill;
format.compressionQuality = 0.5;
format.diskCapacity = 1 * 1024 * 1024; // 1MB
format.preloadPolicy = HNKPreloadPolicyLastSession;
imageView.hnk_cacheFormat = format;
```

The image view category will take care of registering the format in the shared cache.

### Disk cache

A format can have disk cache by setting the `diskCapacity` property with a value greater than 0. Haneke will take care of evicting the least recently used images of the format from the disk cache when the disk capacity is surpassed.

### Preload policy

When registering a format, Haneke will load none, some or all images cached on disk into the memory cache based on the preload policy of the format. The available preload policies are:

* `HNKPreloadPolicyNone`: No images will be preloaded.
* `HNKPreloadPolicyLastSession`: Only images from the last session will be preloaded.
* `HNKPreloadPolicyAll`: All images will be preloaded.

If an image of the corresponding format is requested before preloading finishes, Haneke will cancel preloading to give priority to the request. To make the most of this feature it's recommended to register formats on startup.

Preloading only applies to formats that have disk cache.

### Pre and post resize blocks

Formats can have blocks that will be called before and after the original image is resized: `preResizeBlock` and `postResizeBlock` respectively. Both receive a key and the image up to the corresponding stage. For example:

```objective-c
format.postResizeBlock = ^UIImage* (NSString *key, UIImage *image) {
    UIImage *roundedImage = [image imageByRoundingCorners];
    return roundedImage;
};
```

These blocks will be called only if the requested image is not found in the cache. They will be executed in background when using the image view category or the asynchronous methods of the cache directly.


##Requirements

Haneke requires iOS 7.0 or above and ARC. 

##Roadmap

Haneke is in initial development and its public API should not be considered stable.

##License

 Copyright 2014 Hermes Pique (@hpique)
 
 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at
 
 http://www.apache.org/licenses/LICENSE-2.0
 
 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.