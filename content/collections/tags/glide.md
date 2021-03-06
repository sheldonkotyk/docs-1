---
title: Glide
overview: Manipulate Asset images on the fly using the wonderful [Glide](http://glide.thephpleague.com/) library.
parameters:
  -
    name: src|id
    type: string
    description: The ID of the asset when _not_ using the shorthand syntax. (Use the shorthand syntax if you can, it's nicer.)
  -
    name: path
    type: string
    description: The path to an image, if you don't want to use assets. This should be relative to your webroot, eg. `/assets/photo.jpg`
  -
    name: field
    type: tag part
    description: 'The name of the field containing the asset ID or image path when using the shorthand syntax. This is not actually a parameter, but part of the tag itself. For example, `{{ glide:hero_image }}`.'    
  -
    name: filename
    type: string
    description: >
      Optionally prepends a filename to the end of the URL. See [_Vanity Filenames_](#vanity-filenames).
  -
    name: width
    type: integer
    description: Width in pixels
  -
    name: height
    type: integer
    description: Height in pixels
  -
    name: square
    type: integer
    description: >
      A shortcut for setting `width` and
      `height` to the same value.
  -
    name: fit
    type: string
    description: >
      How the image should fit into the target
      dimensions. Options are `contain`,
      `max`, `fill`, `stretch`, `crop`, `crop-x-y`,
      and `crop_focal`. See the [_Focal Crop_](#focal-crop) section
      for more details.
  -
    name: crop
    type: string
    description: 'Crops the image to specific dimensions prior to any other resize operations. Required format: `width,height,x,y`'
  -
    name: orient
    type: mixed
    description: >
      Rotates the image. Accepts `auto`, `0`,
      `90`, `180` or `270`. Default is `auto`.
      The `auto` option uses Exif data to
      automatically orient images correctly.
  -
    name: brightness
    type: integer
    description: >
      Adjusts the image brightness. Use values
      between `-100` and `+100`, where `0`
      represents no change.
  -
    name: contrast
    type: integer
    description: >
      Adjusts the image contrast. Use values
      between `-100` and `+100`, where `0`
      represents no change.
  -
    name: gamma
    type: integer
    description: >
      Adjusts the image gamma. Use values
      between `0.1` and `9.99`.
  -
    name: sharpen
    type: integer
    description: >
      Sharpen the image. Use values between
      `0` and `100`.
  -
    name: blur
    type: integer
    description: >
      Adds a blur effect to the image. Use
      values between `0` and `100`.
  -
    name: pixelate
    type: integer
    description: >
      Applies a pixelation effect to the
      image. Use values between `0` and
      `1000`.
  -
    name: filter
    type: string
    description: >
      Applies a filter effect to the image.
      Accepts `greyscale` or `sepia`.
  -
    name: quality
    type: integer *90*
    description: >
      Adjusts the quality of the image. Use values between `0` and `100`.
id: b70a3d9a-6605-446e-b278-de99ba561fe0
---
## Example {#example}

### Referencing an Asset

We have an image's asset ID saved in the YAML, and we want to resize it to 300x200, fit it inside the area by cropping it, and apply a sepia filter.

``` language-yaml
image: "380dc8d9-481c-4d18-9162-ecd5688f98a8"
```

```
{{ glide src="{image}" width="300" height="200" fit="crop" filter="sepia" }}

<!-- shorthand syntax: -->
{{ glide:image width="300" height="200" fit="crop" filter="sepia" }}
```

``` .language-output
/img/id/380dc8d9-481c-4d18-9162-ecd5688f98a8?w=300&h=200&fit=crop&filt=sepia&s=3982hf983f2mf90r23
```

### Referencing a list of assets

If you have a list of asset IDs, you may want to loop over them and generate images for each one.
You can do this using the [Assets](/reference/tags/assets) tag:

``` .language-yaml
images:
  - a09b7fa0-2749-11e6-bdf4-0800200c9a66
  - a26a9a50-2749-11e6-bdf4-0800200c9a66
```

Make sure to reference `id` on the Glide tag, since when inside the `assets` loop, that would output
the ID of the asset:

```
{{ assets:images }}
  {{ glide:id width="300" height="200" }}
{{ /assets:images }}
```

``` .language-output
/img/id/a09b7fa0-2749-11e6-bdf4-0800200c9a66?w=300&h=200&s=3982hf983f2mf90r23
/img/id/a26a9a50-2749-11e6-bdf4-0800200c9a66?w=300&h=200&s=3982hf983f2mf90r23
```

### Referencing a path

If you don't want to use assets and would just like to point to an image, you can do that too.

``` language-yaml
image: "/assets/image.jpg"
```

```
{{ glide path="{image}" width="300" height="200" fit="crop" filter="sepia" }}

<!-- shorthand syntax: -->
{{ glide:image width="300" height="200" fit="crop" filter="sepia" }}
```

``` .language-output
/img/assets/image.jpg?w=300&h=200&fit=crop&filt=sepia&s=3982hf983f2mf90r23
```

### Complex sources/paths

If you need the path to your image to be generated with another tag, you can't just drop that into the `path` parameter.
It would likely be confusing to read your templates – there could be parameters in parameters, oh my.

Instead, use the `glide` tag as a tag pair. The contents of the tag will be used as the `path`.

```
{{ glide width="300" height="200" fit="crop" filter="sepia" }}
  {{ theme:img src="photo.jpg" }}
{{ /glide }}
```

``` .language-output
/img/site/themes/your-theme/img/photo.jpg?w=300&h=200&fit=crop&filt=sepia&s=3982hf983f2mf90r23
```


## Focal Crop {#focal-crop}

When using the `fit` parameter, you may choose to crop to a focal point. You may specify the
two offset percentages: `crop-x%-y%`.

For example, `fit="crop-75-50"` would crop the image and make sure that the point at 75% across
and 50% down would be the focal point.

Rather than specifying the offsets, you may use `crop_focal` to use the asset's saved focal point.

A Statamic image asset can be assigned a percentage based focal point. You can do this by editing an
asset and defining the focal point using the UI. Or, you may add `focus: x-y` to the asset's metadata.

When using `crop_focal` and an asset doesn't have a focal point set, it will crop from the center.

If you are specifying an image by its path, and not by asset, the default `fit` parameter will be `crop`.

Note: All Glide generated images are cropped at their focal point, unless you disable the _Auto Crop_
setting. This happens even when you don't specify a `fit` parameter. You may override this behavior
per-image/tag by specifying the `fit` parameter as described above.


## Vanity Filenames {#vanity-filenames}

The Glide URL is not really pretty, nor SEO friendly. The URL contains the ID of the image asset. For example: `/img/cbffccd0-ae6f-11e5-a837-0800200c9a66?w=20`.

Specifying `filename="bacon.jpg"` would change the URL to `/img/cbffccd0-ae6f-11e5-a837-0800200c9a66/bacon.jpg?w=20`.

The additional segment is purely cosmetic in terms of Statamic functionality (hence 'vanity'). However now if you were to right click and save the image, it would name it `bacon.jpg`. It would also be more recognizable to search engines as a photo of bacon rather than a photo of a `cbffccd0-ae6f-11e5-a837-0800200c9a66`.

The vanity filename option is only available when referencing assets. When using paths, the filename will already be the filename.


## Serving Cached Images Directly {#serving-cached-images}

Glide brings you some pretty nifty on-the-fly URL manipulations. The default behaviour of the Glide tag is to simply output a URL. When that URL is
visited, Glide analyses the URL and manipulates an image. However, if you have a lot of assets in your site and a lot of them on a page, time for each
request can soon start to add up.

It is possible to "reverse" this behavior and to simply generate static images. Your server will load the images directly instead of handing the work
over to Glide each time. If you are familiar with Statamic v1, this can be thought of similar to the "Transform" tag.

In `Configure > Settings > Assets`, or `site/settings/assets.yaml`:

``` .language-yaml
# Enable or disable the feature
image_manipulation_cached: true

# The folder containing the manipulated images.
# If you're running above webroot, this might be something like public/img
image_manipulation_cached_path: img

# The URL to the folder
image_manipulation_route: img
```
