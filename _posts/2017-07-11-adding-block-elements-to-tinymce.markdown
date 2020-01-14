---
layout: post
title:  "Adding Elements to the TinyMCE Format Dropdown"
date:   2017-07-11 14:59:53 -0700
categories: wordpress
---
# Adding Elements to the TinyMCE Format Dropdown

On a recent migration project, one of the requirements was to add a few blockquote styles to the TinyMCE dropdown list to match the editorial process of the old CMS. Wordpress [provides a filter](https://codex.wordpress.org/TinyMCE_Custom_Styles) to add a secondary or tetriary dropdown, but if you only have a couple additional elements, it seems like bad UX to seperate it from the original dropdown.

Wordpress does provide an argument for `block_formats` when initializing TinyMCE, thus [relying on the defaults](https://www.tinymce.com/docs/configure/content-formatting/#block_formats). By adding this argument to the `tiny_mce_before_init` filter, we can add in our extra elements*:

\* Note: since we're overriding the defaults, we need to include the original elements (p, h1-h6..etc)

```php
function mce_formats( $init ) {

	$formats = array(
		'p'          => __( 'Paragraph', 'text-domain' ),
		'h1'         => __( 'Heading 1', 'text-domain' ),
		'h2'         => __( 'Heading 2', 'text-domain' ),
		// ... h3 - h6 ...
		'pre'        => __( 'Preformatted', 'text-domain' ),
		'pullquote-1' => __( 'Pull Quote 1', 'text-domain' ),
		'pullquote-2' => __( 'Pull Quote 2', 'text-domain' ),
		'pullquote-3' => __( 'Pull Quote 3', 'text-domain' ),
	);

	// concat array elements to string
	array_walk( $formats, function ( $key, $val ) use ( &$block_formats ) {
		$block_formats .= esc_attr( $key ) . '=' . esc_attr( $val ) . ';';
	}, $block_formats = '' );

	$init['block_formats'] = $block_formats;

	return $init;
}
add_filter( 'tiny_mce_before_init', __NAMESPACE__ . '\\mce_formats' );
```

![tinymce dropdown with extra elements](http://i.imgur.com/82N36mH.png)

Now that we have the elements in the dropdown, we need to register them with TinyMCE via the `tinymce.activeEditor.formatter.register()` method. [(docs)](https://www.tinymce.com/docs/configure/content-formatting/)

```javascript
jQuery( document ).on( 'tinymce-editor-init', function( event, editor ) {
    // register the formats
    tinymce.activeEditor.formatter.register('pullquote-1', {
        block : 'blockquote',
        classes : 'blockquote-1',
    });
    tinymce.activeEditor.formatter.register('pullquote-2', {
        block : 'blockquote',
        classes : 'blockquote-2',
    });
    tinymce.activeEditor.formatter.register('pullquote-3', {
        block : 'blockquote',
        classes : 'blockquote-3',
    });
});
```

And there you have it.

![html inserted with our new tinymce buttons](http://i.imgur.com/JqiUab2.png)



