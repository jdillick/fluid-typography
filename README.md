# Fluid Typography

This Fluid Typography starter kit quickly sets you up to create dynamically-resizing, responsive websites (using the vw, or viewport width, CSS unit).

## Getting Started

* Add the _fluid-type.scss SASS file to your project.
* Prior to using _fluid-type.scss mixins and function, set layout widths in your _variables.scss if different from the defaults.

````

$layouts: (
    'desktop': 1250px,
    'tablet': 640px,
    'mobile': 320px
);

// Account for difference between 100vw and 100% when scroll bars are included on page.
// Center and trims the offset caused in most browsers by scroll bars when true
$body-width-fix: true;

// set to true to limit maximum scale up for larger screens (default false)
$use-maximum-scaling-width: true;
$maximum-scaling-width: 1600px;

// set the base font size in pixels that will be used for non-screen media (default 10px)
// in general, make this the same value in px that you intend to use as your root font size for rem based sizing.

i.e. body { font-size: 1.6rem; } // 16px for print style
$print-base-font-size: 10px;
````

## Usage

### Quick Setup

#### Proportionally Perfect

For each layout, to make rem values that scale perfectly to match your layout.

```
// in your scss
// rem values scale 1:1 with screen size to be proportional to layout
html {
  @include vw-base;
}
```

#### Limited Up-scale

For each layout, make rem values proportionally perfect at the ideal width, and then
scale gradually until the next layout.

```
// in your scss
// rem values are proportional to layout at minimum (at the layout width)
// and scale up slowly to 120% maximum before next layout
html {
  @include base-font-locks;
}
```

### Pass Layouts to Javascript Environment

Use the bp-to-json mixin to package the layouts map into your body:before psuedo element as parsable JSON.

In your scss:

```
body {
  @include bp-to-json;
}
```

In your javascript:

```
// Get a list of layouts that came from css
const layouts = JSON.parse(JSON.parse(window.getComputedStyle(document.body, ':before').getPropertyValue('content')));

// Now you can run js media queries on this
let isMobile = window.matchMedia(`(max-width: ${layouts.tablet - 1}px)`).matches;
```

## TLDR;

There are a couple different ways you can use VWs in your project.

Default (recommended): Apply the vw-base mixin to the root <html> element. Then, in your CSS, set your style measurements (font-size, padding, margin, width, height, position, etc.) for all other elements using the px2rem() function. Simply measure your dimmensions and values in pixels in Photoshop, and enter the pixel value into the px2rem() function. Your value will be converted to REMs, which are similar to EMs except that they are relative to the root element and will not be affected by the parent element's font-size.

````
html {
  // root font-sizes, in vw's
  @include vw-base;
}

body {
  // the true default font-size, in rem's (should be 16px when viewed at exact layout width, will be larger than 16 for
  // screens larger than the layout width, but proportional to the width)
  font-size: px2rem(16);
}

h1 {
  font-size: px2rem(48); // 48 pixels when viewed at ideal layout width
  margin: px2rem(30) 0;
}
````

Option 2: ("font locks")

````
html {
  // root font-sizes, in calculated scale
  // scale root font size from 10px proportional to layout up to 12px
  // proportional to layout
  @include base-font-locks(10, 12);
}

body {
  // the true default font-size, in rem's (should be 16px when viewed at exact layout width, will be larger than 16 for
  // screens larger than the layout width, no more than 120% size)
  font-size: px2rem(16); // from 16px physical up to 19.2px before breaking to next layout
}

h1 {
  font-size: px2rem(48); // 48 pixels when viewed at ideal layout width
  margin: px2rem(30) 0;
}
````

Option 3: (individual component control) To affect individual components only, apply the vw-base mixin to the component container element. Then, use ems and the px2em() function to resize all component elements. This is a little more complicated because EMs are relative to the font-size of the context in which they are used, so you will have to track the current sizing context each time you set the font-size.

````
header {
	@include vw-base;

	h1 {
		font-size: px2em(48);
		margin: px2em(30, 48) 0;
	}

	h2 {
		font-size: px2em(36);
	}
}
````

### Setting minimum and maximum font sizes

To avoid text that gets too small or expands too large, you can set min and max font sizes using the font-size-breakpoint() function.

Example:

````
h2 {
	font-size: px2em(16);

	// when the text scales down to 12px, set a fixed font-size of 12px on desktop
	@media screen and (max-width: font-size-breakpoint(12px, $desktop-layout, 16px)) and (min-width: $desktop-min) {
		font-size: 12px;
	}

	// when the text scales up to 20px, set a fixed font-size of 20px on tablet
	@media screen and (max-width: $tablet-max) and (min-width: font-size-breakpoint(20px, $tablet-layout, 16px)) {
		font-size: 20px;
	}
}
````

### VW Quirks & Fixes

#### Body Width Bug Fix

In many browsers, 100vw is not equivalent to 100% on the <html> element--the scroll bars are taken into account for % but not for vws. The relatively easy fix below forces <html> to be 100vw wide and is included, by default, in the vw-base mixin. If 100vw is larger than 100% it applies a negative margin to center the <html> element. The only caveat is that a little bit of your page will be trimmed on the sides.

````
html {
	@media screen {
		width: 100vw;
		margin-left: calc((100% - 100vw) / 2);
		overflow-x: hidden;
	}
}
````

In your scss variables file, you an make this adjustment automatically by setting $body-width-fix to true. Set this to false if you are ok with the layouts being slightly truncated by the right-side scroll-bar (i.e. if this is already accounted for in the design with left/right margins for instance). Warning: If your design is edge to edge and there is visible content touching the edge, using this approach may cut off your content slightly.

To adjust for scroll bars.
```
$body-width-fix: true;
```

#### Print Stylesheets

VWs are interpreted as 0px when you print which renders elements invisible. To avoid undesirable styling, keep all vw styles within @media screen blocks.

Style for desktop first, then use @media **screen** to override your styles for tablet and mobile.

### Browser Compatibility
IE9+, Firefox, Chrome, Safari, iOS, Android 4.4+
