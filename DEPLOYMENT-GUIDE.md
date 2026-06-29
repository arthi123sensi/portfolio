# CloudXAI Deployment Guide

## Step 1: Register Domain
1. Go to [Namecheap](https://www.namecheap.com) or [GoDaddy](https://www.godaddy.com)
2. Search for `cloudxai.io`
3. Purchase domain (~$10-15/year for .io)

## Step 2: Choose Hosting Platform

### Option A: Netlify (Recommended - Easiest)
**Pros:** Free, auto-deploy from Git, custom domain, HTTPS, CDN
**Cons:** None for static sites

1. Go to [Netlify.com](https://netlify.com)
2. Sign up with GitHub
3. Click "New site from Git"
4. Select your repository
5. Build settings:
   - Build command: (leave empty)
   - Publish directory: `/` or `portfolio`
6. Click "Deploy site"
7. Add custom domain:
   - Go to Domain settings
   - Add `cloudxai.io`
   - Update DNS records at your registrar

**DNS Settings for Netlify:**
```
Type: A
Name: @
Value: 75.2.60.5

Type: CNAME
Name: www
Value: your-site.netlify.app
```

### Option B: Vercel (Alternative)
**Pros:** Free, fast, great for static sites
**Cons:** None

1. Go to [Vercel.com](https://vercel.com)
2. Sign up with GitHub
3. Import your repository
4. Deploy
5. Add custom domain in settings

### Option C: AWS S3 + CloudFront
**Pros:** Professional, scalable
**Cons:** Slightly complex, costs ~$1-2/month

1. Create S3 bucket named `cloudxai.io`
2. Enable static website hosting
3. Upload files (index.html, sitemap.xml, robots.txt)
4. Create CloudFront distribution
5. Point domain to CloudFront
6. Add SSL certificate (free with ACM)

### Option D: GitHub Pages (Free but limited)
**Pros:** Free, simple
**Cons:** No custom domain HTTPS easily, basic features

1. Push code to GitHub
2. Go to Settings → Pages
3. Select branch
4. Add custom domain `cloudxai.io`

## Step 3: DNS Configuration
Once domain is registered and hosting is chosen:

1. Go to your domain registrar (Namecheap/GoDaddy)
2. Find DNS settings
3. Add records provided by your hosting platform
4. Wait 24-48 hours for propagation (usually faster)

## Step 4: Google Search Console Setup

1. Go to [Google Search Console](https://search.google.com/search-console)
2. Add property: `cloudxai.io`
3. Verify ownership (DNS or HTML file method)
4. Submit sitemap: `https://cloudxai.io/sitemap.xml`
5. Request indexing for main page

## Step 5: Google Analytics Setup

1. Go to [Google Analytics](https://analytics.google.com)
2. Create account for CloudXAI
3. Get tracking code
4. Add to your HTML before `</head>`:

```html
<!-- Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');
</script>
```

## Step 6: Social Media Optimization

### Create OG Image (for social sharing)
1. Use [Canva](https://canva.com) or Figma
2. Size: 1200x630px
3. Include: CloudXAI logo, tagline, key visual
4. Save as `og-image.jpg`
5. Upload to root directory
6. Already linked in HTML meta tags

### Update Social Profiles
- **LinkedIn:** Add cloudxai.io to website field
- **GitHub:** Add to profile bio and README
- **Twitter:** Update bio with link

## Step 7: Post-Launch Checklist

### Day 1:
- [ ] Test site on mobile and desktop
- [ ] Verify all links work
- [ ] Test contact forms/modals
- [ ] Check page load speed (use PageSpeed Insights)
- [ ] Verify HTTPS is working
- [ ] Submit to Google Search Console
- [ ] Setup Google Analytics

### Week 1:
- [ ] Post announcement on LinkedIn
- [ ] Share on Twitter
- [ ] Submit to 5 directories (see SEO-STRATEGY.md)
- [ ] Write first blog post
- [ ] Monitor Google Analytics

### Month 1:
- [ ] Check Google Search Console for issues
- [ ] Review analytics data
- [ ] Update content based on feedback
- [ ] Build 5-10 backlinks
- [ ] Create 2 open-source projects

## Monitoring & Maintenance

### Weekly:
- Check Google Analytics for traffic
- Monitor Google Search Console for errors
- Respond to any contact form submissions

### Monthly:
- Update portfolio with new projects
- Write and publish 1-2 blog posts
- Check and fix any broken links
- Review and improve SEO
- Update sitemap if needed

## Troubleshooting

### Site not loading?
- Check DNS propagation: [whatsmydns.net](https://whatsmydns.net)
- Clear browser cache
- Try incognito mode
- Check hosting platform status

### Not appearing in Google?
- Wait 2-4 weeks after launch
- Verify Search Console ownership
- Check robots.txt isn't blocking
- Ensure sitemap is submitted
- Build more backlinks

### Slow loading?
- Compress images (use TinyPNG)
- Minify CSS/JS
- Enable CDN
- Use WebP images

## Cost Breakdown

| Item | Cost | Frequency |
|------|------|-----------|
| Domain (.io) | $12-15 | Yearly |
| Netlify Hosting | $0 | Free |
| SSL Certificate | $0 | Free (included) |
| Google Workspace (optional) | $6 | Monthly |
| **Total** | **~$15-20** | **First year** |

## Recommended: Professional Email

Setup email@cloudxai.io using:
- **Google Workspace** ($6/month) - Recommended
- **Zoho Mail** (Free for 1 user)
- **ProtonMail** (Free tier available)

## Next Steps

1. Deploy site to Netlify/Vercel
2. Configure custom domain
3. Submit to Google Search Console
4. Start content creation (blogs)
5. Build backlinks
6. Monitor and optimize

Need help with any step? Let me know!
