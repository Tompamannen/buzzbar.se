# Google Search Console Setup Guide for buzzbar.se

## Steps to verify your domain:

1. **Go to Google Search Console**
   - Visit: https://search.google.com/search-console

2. **Add Property**
   - Click "Add Property"
   - Choose "Domain" (recommended) or "URL prefix"
   - Enter: buzzbar.se

3. **Verify Ownership**
   
   ### Option A: HTML file upload (Easiest)
   - Google will give you an HTML verification file (e.g., `google1234567890abcdef.html`)
   - Download it and upload to your website root: `/Users/oliverwiseman/Documents/buzzbar.se/`
   - Make sure it's accessible at: `https://buzzbar.se/google1234567890abcdef.html`
   
   ### Option B: DNS verification (Recommended for domains)
   - Add a TXT record to your DNS settings
   - Google will provide the TXT record value
   - Add it through your domain registrar (wherever buzzbar.se is registered)
   
   ### Option C: HTML meta tag (Already prepared)
   - Google will give you a meta tag like: `<meta name="google-site-verification" content="abc123..." />`
   - Add it to the `<head>` section of index.html (after line 8)

4. **Submit Sitemap**
   - After verification, go to "Sitemaps" in the left menu
   - Submit: `https://buzzbar.se/sitemap.xml`
   - This helps Google crawl and index your pages

5. **Monitor Performance**
   - Check "Performance" tab for search impressions and clicks
   - Review "Coverage" to ensure all pages are indexed
   - Fix any issues reported in "Coverage" or "Mobile Usability"

## Files Already Created:
- ✅ `robots.txt` - Tells search engines what to crawl
- ✅ `sitemap.xml` - Lists all pages for search engines
- ✅ SEO meta tags added to index.html

## Next Steps:
1. Complete Google Search Console verification
2. Submit the sitemap
3. Request indexing for main pages
4. Monitor performance weekly
