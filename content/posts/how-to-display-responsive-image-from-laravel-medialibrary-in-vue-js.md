---
title: How to display responsive images from Laravel-medialibrary in Vue.js
date: 2022-05-29
summary: It's easy to do in a Blade file, but what about a Vue component?
---

## Introduction

[Spatie](https://spatie.be/) created an extremely useful library for working with any type of media in Laravel, called [`Laravel-medialibrary`](https://github.com/spatie/laravel-medialibrary). It allows you to associate all sorts of files with Eloquent models and organize them into collections. It also has a simple fluent API that makes it effortless and pleasant to use.

Additionally, the package supports different filesystems (like Amazon S3) and can manipulate all uploaded images or pdfs according to the specified configuration. It's possible to resize, reformat, apply special effects, and much more. Check out the [documentation](https://spatie.be/docs/laravel-medialibrary) to learn about everything it can do for you.

However, the feature I like most about this library is that it can automatically create responsive variants of images. The moment you upload an image, the media library will create resized versions of the original image suitable for different screen sizes, as well as a tiny blurred one used for progressive loading (which will be displayed before the full image is ready).

What's even better, the package has support for generating a necessary HTML markup, automatically creating an `img` tag and specifying `src`, `srcset`, and `sizes` attributes for you.

To display your image in a Blade file, all you have to do is output the `Media` object:

```html
<h1>My blog post</h1>

{{ $post->getFirstMedia() }}
```

It's very convenient, isn't it? But what about Vue? In a component's template, we can't just output the object like in a Blade file.

Luckily, there is a way to make responsive images work in Vue.

## Solution

The `Media` object implements the `Htmlable` interface, defined as follows:

```php
interface Htmlable
{
    /**
     * Get content as a string of HTML.
     *
     * @return string
     */
    public function toHtml();
}
```

Internally, the Blade compiler will use the `toHtml()` method when it encounters a `Media` object while rendering the view.

This method is public, so nothing stops us from using it ourselves.

In a controller that's responsible for providing data used by your Vue component, simply return the result of the `toHtml()` method:

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;

class PostController extends Controller
{
    public function show(Post $post)
    {
        return [
			'title' => $post->title,
			'content' => $post->content,
			'img' => $post->getFirstMedia()->toHtml()
		];
    }
}
```

Then, inside your Vue component, output the image using the `v-html` directive by providing the data received from the controller:

```vue
<template>
	<div v-html="img"></div>
</template>
```

And that's it! Now, your image should be correctly displayed based on the screen size.

## Summary

If you've been trying to take advantage of the simplicity of responsive images provided by the media library inside a Vue component, I hope this post saved you some time. If you have some questions, feel free to reach me on [Twitter](https://twitter.com/ClouddJR/).