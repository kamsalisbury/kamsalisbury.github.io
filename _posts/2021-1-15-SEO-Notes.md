Search engine optimization (SEO) is about improving the quality and quantity of website traffic to a website from search engines. The value of SEO is agressively debated in marketing but maybe ignored in other industries. Using SEO will help your website appear in web searches. Applying SEO well depends on what type of web searches or social media you want the website to appear in.

Level
1. Add SEO HTML code (search 'SEO HTML Code').

```
In the head code section;
<title>Your Website Title</title>
<meta name="description" content="Description including your brand name">
<meta name="og:title" property="og:title" content="The website's Open Graph Title for FB and other sites using Open Graph">
<meta name="robots" content="index, follow">
<link rel="canonical" href="https://www.yourwebsite.tld/">
<meta name="twitter:card" content="The website summary for Twitter shares">
<meta name="viewport" content="width=device-width, initial-scale=1">

In the body code section;
<h1>Use only one H1 tag per a page, other sections of the same page use H2-6 tags</h1>
<img src="url" alt="Don't omit img tag alt text unless you want the image to not show up in searches or blind people not to be able to know what the image is.">
<a href="http://somerandomwebsite.com/" rel="nofollow">your anchor text to another website but that site is not affiliated with your site</a>
```

2. Get seen in search engines. Register the website main URL with [Bing Webmaster Tools](https://www.bing.com/webmasters) and [Google Search Console](https://search.google.com/search-console).

3. There are multiple websites that will [evaluate a URL for speed](https://developers.google.com/speed/pagespeed/insights/) or [compliance to a code standard](http://validator.w3.org/) or even [security](https://securityheaders.com/). Which tools used (if any) are goal dependent.

4. Add tracking javascript (Which I do not recommend unless you are developing an app, or require grainular telemetry for marketing). Start with [Google Analytics](https://analytics.google.com/analytics) and add additional tracking code if needed for specific telemetry or tracking. The more javascript you add to the website, the slower it will execute.
