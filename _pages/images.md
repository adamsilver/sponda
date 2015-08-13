---
layout: default
title: Image standards and conventions
id: images
permalink: /images/
---

# Images

## Inline images

Inline images are images that, on their own, provide value in their content e.g. star rating.

This is because the value of the star rating is valuable to the user regardless if they are sited or not.

The "alt" text is perfect because it is read out by screen readers and is available in the event that the image doesn't load.

**This works in *all* browsers.**

## Decorative images

Decorative images are images that compliment the *visual* design of the page or component.

An example would be up and down arrows that compliment links and buttons to show that clicking the button would expand or collapse.

Another example would be the restaurant cuisine, delivery, collection, address and time icons.

Without them, all the information would be there for the user in text. They are there to compliment the visual design and scanability of some piece of content.

In these cases, background CSS images should be used.

**This works in *all* browsers.**

## Naming conventions and File Organisation

* All files should live under path/to/images/

* All images should be named in lowerCamelCase.

* All images should be named on what they are, not what they represent.

Good:

up_arrow.png

Bad:

bg_collapsed.png