---
layout: default
title: CSS standards and conventions
id: css
permalink: /css/
---

# CSS

## Writing maintainable CSS...

[Maintainablecss.com](http://maintainablecss.com)

## Syntax

* Use soft tabs.

* When grouping selectors, keep individual selectors to a single line. Include one space before the opening brace of declaration blocks for legibility. Place closing braces of declaration blocks on a new line.

Code:

	selector,
	selectorN {

	}

* Include one space after : for each declaration. Place closing braces of declaration blocks on a new line. End all declarations with a semi-colon. The last declaration's is optional, but your code is more error prone without it.

Code:

	selector {
		rule: value;
		rule2: value;
	}

* Comma-separated property values should include a space after each comma (e.g., box-shadow).

Code:

	selector {

	}

* Don't prefix property values or color parameters with a leading zero (e.g., .5 instead of 0.5).

* Lowercase all hex values, e.g., #fff. Lowercase letters are much easier to discern when scanning a document as they tend to have more unique shapes.

* Use shorthand hex values where available, e.g., #fff instead of #ffffff.

* Quote attribute values in selectors, e.g., input[type="text"]. They’re only optional in some cases, and it’s a good practice for consistency.

* Avoid specifying units for zero values, e.g., margin: 0; instead of margin: 0px;.

Code:

	/* Bad CSS */
	.selector, .selector-secondary, .selector[type=text] {
	  padding:15px;
	  margin:0px 0px 15px;
	  background-color:rgba(0, 0, 0, 0.5);
	  box-shadow:0px 1px 2px #CCC,inset 0 1px 0 #FFFFFF
	}

	/* Good CSS */
	.selector,
	.selector-secondary,
	.selector[type="text"] {
	  padding: 15px;
	  margin-bottom: 15px;
	  background-color: rgba(0,0,0,.5);
	  box-shadow: 0 1px 2px #ccc, inset 0 1px 0 #fff;
	}

## Conventions and style

### Split files up by what they are, not where they live

Sometimes components live in many places or get moved around. If they are named based on what they are, then little or no reorganising needs to take place later.

### Don't use @import

Compared to <link>s, @import is slower, adds extra page requests, and can cause other unforeseen problems. Avoid them and instead opt for an alternate approach:

* Use multiple <link> elements

### Prefixed properties

Don't use vendor prefixes unless absolutely necessary and beneficial to the product. They are prefixed for a reason. Instead be happy that they degrade gracefully all on their own.

### Shorthand notation

Strive to limit use of shorthand declarations to instances where you must explicitly set all the available values.

	/* good */
	selector {
		padding-left: 10px;
	}

	/* bad */
	selector {
		padding: 0 0 10px 0;
	}

Likewise:

	/* good */
	selector {
		padding: 10px 10px 0 10px;
	}

	/* bad */
	selector {
		padding-left: 10px;
		padding-right: 10px;
		padding-top: 10px;
	}

Common overused shorthand properties include:

* padding
* margin
* font
* background
* border
* border-radius

Often times we don't need to set all the values a shorthand property represents. For example, HTML headings only set top and bottom margin, so when necessary, only override those two values. Excessive use of shorthand properties often leads to sloppier code with unnecessary overrides and unintended side effects.


### Selectors

* Use classes over generic element tag for optimum rendering performance and cross-browser support; avoid using attribute selectors (e.g., [class^="..."]). Browser performance is known to be impacted by these and cross-browser compatibility becomes an issue.

	/* good */
	.textInput {

	}

	/* bad */
	input[type="text"] {

	}

## Bundling and Minification and more

All files should be bundled and revved. Bundle file names should be in the following format:

	<name of bundle>.<the version>.css

e.g. globalModules.123.css

## Architecture

### Base

Base rules are the defaults. They are almost exclusively single element selectors but it could include attribute selectors, pseudo-class selectors, child selectors or sibling selectors. Essentially, a base style says that wherever this element is on the page, it should look like this.

Developers should be *very* careful when editing the base styles and it should be imperative that any change to these styles be communicated and reviewed by the other developers.

Examples of base rules:

	html,
	body {
		margin: 0;
	}

	a {
		color: #000;
	}

	a img {
		border: none;
	}

As there aren't many state classes just add these to base.css. An example of a state class:

	.hide {
		display: none;
	}

### Clearfix

Littering the HTML DOM with *clearfix* classes is not ideal because it is not semantic. Read more under HTML/Semantic HTML.md. It talks about Responsive Web Design issues too.

The clean way to solve this is by having a dedicated file called clearfix.css.

Within this file is one ruleset with comma delimited (opt-in) selectors.

	/* Inside clearfix.css */
	.someSelector:after,
	.another:after,
	.andSoOnEtc:after {
		clear:both;
		content:".";
		display:block;
		height:0;
		visibility:hidden;
	}

### Fixing IE

It is then likely that various versions of IE will need a conditional style, within the relevant conditional IE stylesheet for hasLayout issues.

	/* e.g fixing IE7 */

	#whatever,
	selector,
	.andSoOnEtc {
		zoom: 1
	}

And for IE6...considering the effort for this is minimal...

	/* e.g fixing IE6 */

	#whatever,
	selector,
	.andSoOnEtc {
		height: 1%;
	}

### Colour

Reusing colours is straightforward. Take an example where two different paragraphs and one heading specific to a module require the same colour.

	.someModule p,
	.someOtherModule p,
	.basketModule h2 {
		color: #ff0000;
	}

### Internet Explorer conditional CSS

In order to provide essential CSS fixes to IE9 and below there should be a file per each browser that we support.

Do not group these files together with a conditional comment such as "IE9 and below".

	<!--[if IE 6]><link rel="stylesheet" href="/css/ie6.css" type="text/css" media="all"><![endif]-->

	<!--[if IE 7]><link rel="stylesheet" href="/css/ie7.css" type="text/css" media="all"><![endif]-->

	<!--[if IE 8]><link rel="stylesheet" href="/css/ie8.css" type="text/css" media="all"><![endif]-->

	<!--[if IE 9]><link rel="stylesheet" href="/css/ie9.css" type="text/css" media="all"><![endif]-->

The benefits include:

* Each problem browser will only have 1 extra HTTP request that is as lightweight and specific as possible

* Dropping support for a version if/when decided is straightforward. Delete the line in the HTML, delete the file.

* CSS fixes are encapsulated within the CSS file and not via HTML calss littering.

* CSS hacks are not required because the conditional CSS is loaded last and specifically targeted towards the browser version.
