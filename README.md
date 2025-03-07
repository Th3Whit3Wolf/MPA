# MPA

Some notes I'm taking while working on an MPA

## Performance
How to optimize page performance
### Images

#### Lazy Loading

By default, all images on a page are loaded as soon as the user opens the page. Setting the [`loading`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#attr-loading) attribute to `lazy` can defer fetching images and iframes until the user scrolls near the element.

- [DebugBear | Image Lazy Loading](https://www.debugbear.com/blog/image-lazy-loading#the-problem-resource-prioritization)

##### How do you lazy load a background image?

The `loading=lazy` attribute doesn't work for background images, so you'll need to use JavaScript to [defer an off-screen background image](https://www.debugbear.com/blog/defer-offscreen-images). You can check if the image is near the viewport and then either modify the `style` attribute directly or add a class with a `background-image` defined in a CSS file.

You can use the [Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) to check whether the image is in the viewport.

##### Asynchronous Decoding
Because the browser loads all the resources (text, images) on a web page synchronously before rendering the page, it would take longer to decode heavy or multiple images on a page. This would delay the rendering of the page, causing the UI to freeze until it is complete.

Several factors (image size, format, etc.) contribute to the time it takes to display an image. Still, the time it takes to decode an image is very important in determining when to load the image once it has been downloaded.

To control image decoding use the `decoding` attribute of the HTML `<img>` element and set it’s value to `async`.

This way, image decoding happens asynchronously (off the main thread), leaving the main thread free so the browser can continue rendering the other content of the page. It defers the image decoding until later and displays it when it is done.

##### Show Performance-Friendly Placeholders for Deferred Images
Slower devices or networks still need some kind of visual feedback until the images are rendered to the screen. It's a good practice to use a low quality image placeholder until the proper image is downloaded

##### Notes
- Don't forget to [add the `width` and `height` attributes to the `<img>` element](https://www.debugbear.com/docs/image-elements-do-not-have-explicit-width-and-height), as lazy loaded images may push other content elements aside if their dimensions are not defined, which can result in unexpected layout shifts and harm your [Cumulative Layout Shift](https://www.debugbear.com/docs/metrics/cumulative-layout-shift) (CLS) scores.
- Don't lazy loading above-the-fold images. Doing so can be especially harmful if you defer images that are candidates for the [Largest Contentful Paint element](https://www.debugbear.com/docs/largest-contentful-paint-element) on the page _(Lighthouse even has a flag that warns users [not to lazy load the LCP image](https://www.debugbear.com/docs/lcp-lazily-loaded))_.

#### Use Next-Gen Formats
AVIF and WebP are image formats that have superior compression and quality characteristics compared to their older JPEG and PNG counterparts. Encoding your images in these formats rather than JPEG or PNG means that they will load faster and consume less cellular data.

- AVIF has less browser compatibility but has better compression than Webp
- WebP is still much better than JPEG and PNG
- JPEG compresses better than PNG

[CanIUse | WebP](https://caniuse.com/webp)
[CanIUse | avif](https://caniuse.com/avif)

#### Make Images Responsive
Responsive images allow you to tell the browser that when they request an image they can choose from a predefined list and pick the image best for their user's device. You can find a more detailed explanation [here](https://www.debugbear.com/blog/responsive-images#what-are-responsive-images)

The big wins from this are reduced network usage (potentially better load times) and better LCP (Largest Contentful Paint

Here are a few responsive image generators you can use to automate making responsive images
- [Responsive Image Breakpoints Generator](https://www.responsivebreakpoints.com/)
- [John Franey's Responsive Image Generator](https://johnfraney.ca/tools/responsive-image-generator/)
- [Felix Rieseberg's Responsive Images Generator](https://github.com/felixrieseberg/responsive-images-generator)

#### Image Optimizations

There are many tools you can use to shrink images.  for shrinking PNGs

| Format | Tool                                                                                                                                                                                                          |
| :----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| PNG    | [pngquant](https://pngquant.org/) (lossy) [rdopng](https://github.com/richgel999/rdopng) (lossy), [OxiPNG](https://github.com/shssoichiro/oxipng)(lossless), [Pngcrush](https://pmt.sourceforge.io/pngcrush/) |
| JPEG   | [MozJPEG \| jpegtran](https://github.com/mozilla/mozjpeg)(lossless), [MozJPEG \| cijpeg](https://github.com/mozilla/mozjpeg) (lossy, [jpegli](https://github.com/libjxl/libjxl/tree/main/lib/jpegli) (lossy)  |
| GIF    | [Gifsicle](https://www.lcdf.org/gifsicle/)                                                                                                                                                                    |
| SVG    | [SVGO](https://github.com/svg/svgo)                                                                                                                                                                           |
Formats like webp and avif can be created from PNG & JPEG formats with a reduced quality level to produce smaller files.

#### Preload

You can [preload your LCP image](https://www.debugbear.com/blog/preload-largest-contentful-paint-image#how-to-preload-your-lcp-image)  by  adding a preload tag to the HTML header with the image `href` and the `fetchpriority=”high”` attribute.

### CSS

Here are the easy ones:

- Minify your CSS
- Reduce unneeded CSS
	- If a style/styles are only being used on one page then they should be inlined
- Avoid long CSS selectors
- Avoid using `@import`

#### Don't embed images or fonts in your CSS code

While it's possible to use data URLs and Base64 encoding to embed images and other resources in your CSS you should avoid it where you can.

Image files are often large and take more time to download. Normally they are not render-blocking, but if they are embedded in a stylesheet they do block render and content takes longer to appear for users.

Loading CSS asynchronously means loading styles without blocking rendering. Generally this is undesirable as you don't want visitors to see your website without any styles. Users would experience a flash of unstyled content and when the styles are loaded the page layout would change.

However, it can be useful for less important styles, for example for a widget that's shown far down the page.

Here's an example from [DebugBear](https://www.debugbear.com/blog/css-page-speed#loading-css-asynchronously)

```html
<link  rel="stylesheet"  href="styles.css"  media="print"  onload="this.media='all'"/>
```

#### Consider Async Loading
### JavaScript

Always minify javascript.

#### Async and Defer
For non-critical javascript you can [Improve Page Speed With Async And Defer](https://www.debugbear.com/blog/async-vs-defer) 

The difference between `async`  and `defer` is:
- Async - When the script with the async keyword is read the browser will fetch the resource and continue to parse the HTML. Once the javascript resource has been fetched then the browser will stop parsing the HTML (if it is still doing that) and execute the javascript before continuing
- Defer - When the script with the async keyword is read the browser will fetch the resource and continue to parse the HTML. The javascript resource will **NOT** be executed until after the HTML is finished parsing.

#### Partytown

[Partytown](https://partytown.builder.io/) is a lazy-loaded library to help relocate resource intensive scripts into a [web worker](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API), and off of the [main thread](https://developer.mozilla.org/en-US/docs/Glossary/Main_thread). Its goal is to help speed up sites by dedicating the main thread to your code, and offloading third-party scripts to a web worker.
#### Articles
- [Mastering JavaScript high performance in V8](https://marcradziwill.com/blog/mastering-javascript-high-performance/#ic)

### Navigation

#### Prefetch or Prerender URL

Prefetching will download the resource while prerendering will [provide instant page navigations](https://developer.chrome.com/docs/web-platform/prerender-pages). The catch is you can only prerender **1** url per page load.

[This](https://macarthur.me/posts/best-ish-practices-for-dynamically-prefetching-and-prerendering-with-javascript/) is a good article for best practices. I am currently prefetching when a user hovers over a link and prerendering on mousedown. While this probably isn't enough time to prerender the page it does atleast reliably start the download process sooner

##### Chrome Speculations
You can read more about the Chrome's Speculation API [here](https://www.debugbear.com/blog/speculation-rules) but basically it allows the chrome browser to preload & prerender in a more sophisticated manner

##### Interesting Tools
These tools seem like they maybe able to help more accurately predict what the user will click on
- [premonish](https://github.com/mathisonian/premonish)
- [Guess.js](https://guess-js.github.io/docs)
#### View Transitions

View Transitions don't technically speed up navigation but they sure do make them feels faster. They provide an animation between the current webpage and the one the next page the user visits, giving the feels of an SPA (with a lot less bytes).

For more on view transitions and how to use them checkout this [DebugBear](https://www.debugbear.com/blog/view-transitions-spa-without-framework) article.


### Compression

All text-based assets (i.e. HTML, CSS, JavaScript, SVG, etc) can benefit from compression

For all static resources you should use Zopfli (or gzip) and Brotli, both with max compression settings.

For dynamic resources you should be able to set your reverse proxy (Nginx, Apache, Caddy, etc) to compress on the fly. Be careful not to raise the compression levels too high as it will reduce the response time of the server.

### Minification

I've said it before and I'll say it again, minify your CSS, JavaScript, and HTML
There are many tools to do this but I'll name a few

| Asset      | Tool                                                                                                                                                                                |
| :--------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CSS        | [Lightning CSS](https://lightningcss.dev/) it's the fastest CSS minifier and produces the smallest CSS file. If you use [tailwindcss v4](https://tailwindcss.com/) it ships with it |
| JavaScript | [SWC](https://swc.rs/) it's very fast, does a good job, and it's configuration is fairly simple given how many options there are to tweak                                           |
| HTML       | [minify-html](https://github.com/wilsonzlin/minify-html) very fast (maybe the fastest) HTML minifier with great (maybe the best) HTML size reduction.                               |


## UI

### Animations
[This article](https://stuartcusack.ie/development-blog/tailwind-scroll-animations-using-an-intersection-observer) show a clever way to use the [Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) with [tailwindcss](https://tailwindcss.com/) to set class before & after an element is in view,

## SEO

### Basics
- Create a sitemap and upload

The following information comes from [Semrush](https://www.semrush.com/blog/seo-basics/)
#### Title Tags
Google uses title tags to understand your content's topic. But, more importantly, compelling titles convince people to click through to your site.

Do the following to write effective title tags:

- Stay between 50-60 characters
- Include a target keyword
- Clearly describe the page
- Write a unique title tag for each

#### Meta Descriptions

[Meta descriptions](https://www.semrush.com/blog/meta-description/) are short summaries of a page that can appear as a snippet below the title on SERPs.

While meta descriptions don't directly affect rankings, they can influence [click-through rates](https://www.semrush.com/blog/click-through-rate/).

To write effective meta descriptions, do the following:

- Stick to 105 characters or fewer
- Include the target keyword naturally
- Highlight the page's value 
- Include a clear CTA
- Write a unique meta description for each page

#### Heading Tags
Heading tags are HTML tags on pages that organize your content into sections and help Google understand the structure of your pages. 

It’s best to use headings and subheadings (H2-H6) to segment topics in a hierarchical and logical way.

Here are some other tips to use heading tags effectively:

- Include one [H1](https://www.semrush.com/blog/h1-tag/) tag per page
- Use your target keyword in the H1
- Structure main topics with H2s and subtopics with H3s and smaller
- Keep headings descriptive and clear

#### Page URLs
To optimize your URLs:

- Keep them descriptive but concise
- Use your primary keyword
- Separate words with hyphens
- Remove unnecessary words (“a,” “the,” “and.” etc.)
- Avoid special characters
#### Images

Images can make your content more engaging and bring in traffic through Google image searches.

But search engines can't "see" images. So, you need to make sure they can understand what’s being shown.

To optimize images for search:

- Use descriptive file names (mountain-hiking-gear.jpg—not IMG001.jpg)
- Add relevant [alt text](https://www.semrush.com/blog/alt-text/) to describe each image (include your keyword when it makes sense)
- Compress files to improve loading speed with tools like [TinyPNG](https://tinypng.com/)
- Choose the right file format (WebP is often best, but you can also use JPG for photos and PNG for graphics)

#### Help the Crawlers
A [robots.txt](https://www.semrush.com/blog/beginners-guide-robots-txt/) file tells search engines which of your pages they should and shouldn't crawl, which helps search engines prioritize the most important pages.

#### Optimize Your Core Web Vitals
Google’s [Core Web Vitals](https://www.semrush.com/blog/core-web-vitals/) are a set of metrics that indicate how user-friendly your site.

They include the following:

- **Largest Contentful Paint (LCP)**: Measures how long it takes for the largest element on your page to load
- **Interaction to Next Paint (INP)**: Assesses how quickly a webpage responds to user interactions
- **Cumulative Layout Shift (CLS)**: Measures much the elements on a page unexpectedly shift as the page loads

#### Use HTTPS

[HTTPS](https://www.semrush.com/blog/http-vs-https/) is a protocol that lets web browsers communicate with web servers in a secure way (using encryption) and is a [ranking factor](https://webmasters.googleblog.com/2014/08/https-as-ranking-signal.html).

Migrating from HTTP to HTTPS is straightforward:

- Purchase an SSL certificate (many hosts offer them for free)
- Set up [301 redirects](https://www.semrush.com/blog/301-redirects/) from HTTP to HTTPS
- Update any links and assets that are still using HTTP
- Update your property in Google Search Console and submit a new sitemap

#### Mobile SEO

Google uses [mobile-first indexing](https://www.semrush.com/blog/mobile-first-indexiing/), meaning it prioritizes the mobile version of your site’s content. 

So, if your site isn’t optimized for [mobile SEO](https://www.semrush.com/blog/mobile-seo/), it could perform poorly in search results.

Here are a few ways to make sure your website is mobile-friendly: 

- Prioritize quick load times
- Use a responsive website layout that automatically adapts to the user’s screen
- Optimize title tags and meta descriptions for mobile SERPs
- Keep paragraphs short and use plenty of white space
- Avoid intrusive pop-ups
- Make all your buttons and text accessible and readable
### Enriched Search Results

- Use [structured data](https://developers.google.com/search/docs/appearance/structured-data/intro-structured-data) to get [Enriched search results](https://developers.google.com/search/docs/appearance/enriched-search-results) when you come up in google search

#### OpenGraph
Use the [Open Graph Protocol](https://ogp.me/) and [Twitter Cards](https://www.seotesteronline.com/blog/seo-social/twitter-cards/)

### Acknowledgments
I took these notes after reading articles by: 

- [Cloudinary](https://cloudinary.com)
- [DebugBear](https://www.debugbear.com)
- [web.dev](https://web.dev)
