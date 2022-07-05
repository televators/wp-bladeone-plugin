# WP Blade(One)

WordPress plugin that provides Laravel Blade (via BladeOne) templating, using the default WordPress template hierarchy.
The actual Blade templating comes from [BladeOne](https://github.com/EFTEC/BladeOne/), which is a standalone, single-file port of Laravel's Blade, without dependencies. It's also actually kept up to date, which makes it pretty unique and helpful in the seemingly niche world of WordPress + Blade.

__NOTE:__ This is a fork of [WP Blade(One)](https://github.com/dievardump/wp-bladeone-plugin) by [@dievardump](https://github.com/dievardump) made by me over the weekend because I've got multiple deadlines and with the WordPress 6.0 update, the original is completely broken. I make no guarantees as to its usability, stability, or longevity.

## 0 - Requirements
- [Composer](https://getcomposer.org)
- [Composer Installers](https://github.com/composer/installers)
- [WordPress](https://wordpress.org)
- PHP >=7.3
  - largely due to dependencies `tightenco/collect` and `eftec/bladeone` but you should be on at least 7.3 anyways in 2022.
- Basic ability to use the command line

## 1 - Installation

Get [composer/installers](https://github.com/composer/installers) installed and configured, so Composer knows how to install this plugin in the right WordPress directory.

Then, require this package with Composer:
```
$ composer require televators/wp-bladeone-plugin
```
and activate it in the WordPress Plugins page of your site.

## 2 - Configuration

There are 3 constants that you can define to configure WP BladeOne.

- `WP_BLADEONE_VIEWS`
  - Where BladeOne will look for the view files.
  - If you use [WP Blade(One) Starter Template](https://github.com/dievardump/wp-bladeone-theme) or want `WP BladeOne` to work with WordPress hierarchy, it should be your theme directory.
  - If you just want to manually render some templates using `wp_bladeone()->run( $view_name, $data )` then you can set another path (see Usage below).
  - Defaults to `get_stylesheet_directory()`.

- `WP_BLADEONE_CACHE`

  - Where BladeOne will have the right to write the views cached files. Usually somewhere in `WP_CONTENT_DIR` as other directories are often not writable.
  - Defaults to `WP_CONTENT_DIR . '/cache/.wp-bladeone-cache'`.

- `WP_BLADEONE_MODE`

  - Configures how BladeOne manages the rendering of the views, including caching.
  - See [BladeOne](https://github.com/EFTEC/BladeOne/) to know what the different modes are and what they do.
  - Defaults to `\eftec\bladeone\BladeOne::MODE_AUTO`

## 3 - Usage

There are two ways to use this Plugin in your templates.

### 3.1 Using Hierarchy

This plugin will hook on some WordPress actions so that when WordPress goes to render a template file, it will first check if a Blade version is available, i.e., before rendering `index.php` it will first check for `index.blade.php`.

This allows you to create full themes working entirely with Blade syntax (see [WP Blade(One) Starter Template](https://github.com/dievardump/wp-bladeone-theme) as en example).

__NOTE:__ (from Televators) For some reason, I could never get `theme-name/footer.blade.php` or `theme-name/header.blade.php` to work, even when mirroring divardump's theme example. Will dig into it later, hopefully. In the mean time, if some pieces aren't being picked up as `*.blade.php` then just use the vanilla `*.php` version. I've never had a problem with proper page, single, or archive templates.

### 3.2 Using `wp_bladeone()->run( $view_name, $data )`

If you don't want to use the WordPress hierarchy hooks, you can set `WP_BLADEONE_VIEWS` to `/views`, or any other directory, and store your Blade templates there.

Then, when you want to use a Blade view in your theme, just use `wp_bladeone()->run( $view_name, $data )`.

_`wp-config.php`_
```php
// Define Blade template directory
define( 'WP_BLADEONE_VIEWS', __DIR__ . '/../views' );
```

_`wp-content/themes/your-theme/index.php`_
```php
<?php
// This will look for WP_BLADEONE_VIEWS . '/index.blade.php' and pass a variable $className with value 'index'.
echo wp_bladeone()->run( 'index', ['className' => 'index'] );
?>
```

Assuming `WP_BLADEONE_VIEWS` is set to a `views` folder in your theme:

_`views/index.blade.php`_
```php
@extends( 'layout' )

@section( 'title' )
  <h1>{{ get_the_title() }}</h1>
@endsection

@section('content')
	@php( the_content() )
@endsection
```

_`views/layout.blade.php`_
```php
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>Document</title>
</head>
<body>
  <article class="{{ $className ?? ''}}">
		@section('title')
			<h1>Default title</h1>
		@show

		@section('content')
			<p>Default content</p>
		@show
  </article>
</body>
</html>
```

And voilà!

## 4 - How does it actually work?

When deciding which template file is required to display a given page or post, WordPress goes through a [Template Hierarchy](https://developer.wordpress.org/themes/basics/template-hierarchy/), taking the first available template.
As with most things, WordPress provides actions for developers to modify the core behavior of this Template Hierarchy, and this plugin only modifies the behavior by prioritizing files ending with `.blade.php` over `.php`.

For example, let's say you try to load a normal page with the slug `/example/` and ID `10`. WordPress will look, in order, for:
```php
[
	'page-exemple.php',
	'page-10.php',
	'page.php',
	'singular.php',
	'index.php'
]
```
This plugin will just modify this so wordpress will look for
```php
[
	'page-exemple.blade.php',
	'page-exemple.php',
	'page-10.blade.php',
	'page-10.php',
	'page.blade.php',
	'page.php',
	'singular.blade.php',
	'singular.php',
	'index.blade.php',
	'index.php'
]
```

Then, if the first existing file ends with `.blade.php`, it will be rendered through BladeOne.

## 5 - Why not use Sage by Roots?

I just wish to be able to use Blade's syntax, nothing else. Sage brings a lot of things that I have absolutely no use for since my stack is totally different than theirs.

---

## The Future
_From Televators_

I have been using dievardump's plugin, which this is a fork of, for a while now and, at the time of writing, have several sites in progress with tight deadlines. When I updated one of them to WordPress version 6.0, seemingly random parts of Blade templates started breaking down and things got ultrajank.

There's not much movement or community around WordPress + Blade, and every other plugin/starter theme with Blade integration is trying to do a lot more than just adding Blade to WP. So, I am just forking this project and kicking the core dependencies (eftech/bladeone and tightenco/collect) up a few major versions and hoping it works-ish. Also fixing typos and tweaking comments/docs for better readability and consistency.

I'd love to take this, get it fully up to date, and keep it updated going forward. BladeOne provides the real muscle and it _is_ kept up to date. So, this one shouldn't be too bad. I've just never created my own Composer package or WP plugin aimed at the public so I don't completely know what I'm doing. Any help, advice, tips, feedback, or collaboration would be &nbsp; ___s i c c___.

<3
