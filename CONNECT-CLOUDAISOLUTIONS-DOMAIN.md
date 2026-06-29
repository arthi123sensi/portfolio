# Connect cloudaisolutions.org to Netlify

Congratulations on purchasing **cloudaisolutions.org**! Let's connect it to your Netlify site.

## Step-by-Step Guide

### Step 1: Login to Netlify

1. Go to [https://app.netlify.com](https://app.netlify.com)
2. Login with your GitHub account
3. Click on your site: `deft-licorice-b8a79a`

### Step 2: Add Custom Domain

1. Click **Domain settings** (in the navigation)
2. Click **Add custom domain**
3. Enter: `cloudaisolutions.org`
4. Click **Verify**
5. Click **Add domain**

### Step 3: Add www subdomain

1. Click **Add domain alias**
2. Enter: `www.cloudaisolutions.org`
3. Click **Save**

### Step 4: Configure DNS at Domain Registrar

Now you need to add DNS records where you bought the domain.

#### If you bought from Namecheap:

1. Login to [Namecheap.com](https://namecheap.com)
2. Go to **Domain List**
3. Click **Manage** next to cloudaisolutions.org
4. Click **Advanced DNS** tab
5. Delete all existing records
6. Add these new records:

**Primary Domain (cloudaisolutions.org):**
```
Type: A Record
Host: @
Value: 75.2.60.5
TTL: Automatic
```

**WWW Subdomain (www.cloudaisolutions.org):**
```
Type: CNAME Record
Host: www
Value: deft-licorice-b8a79a.netlify.app
TTL: Automatic
```

7. Click **Save All Changes**

#### If you bought from GoDaddy:

1. Login to [GoDaddy.com](https://godaddy.com)
2. Go to **My Products**
3. Click **DNS** next to cloudaisolutions.org
4. Add these records:

```
Type: A
Name: @
Value: 75.2.60.5

Type: CNAME
Name: www
Value: deft-licorice-b8a79a.netlify.app
```

#### If you bought from another registrar:

The DNS records are the same everywhere:
- **A Record:** @ pointing to `75.2.60.5`
- **CNAME Record:** www pointing to `deft-licorice-b8a79a.netlify.app`

### Step 5: Wait for DNS Propagation

- **Time:** Usually 15 minutes to 24 hours
- **Average:** 2-4 hours
- **Check status:** [https://whatsmydns.net](https://whatsmydns.net)

Enter `cloudaisolutions.org` and check if it shows `75.2.60.5`

### Step 6: Enable HTTPS (Automatic)

1. Once DNS propagates, go back to Netlify
2. Click **Domain settings**
3. Scroll to **HTTPS**
4. Wait 30-60 minutes
5. SSL certificate will be auto-provisioned
6. **HTTPS** will be enabled automatically ✅

### Step 7: Set Primary Domain

1. In Netlify Domain settings
2. Find `cloudaisolutions.org`
3. Click **Options** → **Set as primary domain**
4. This makes it the default URL

## What to Expect

### After DNS Propagates:

✅ `cloudaisolutions.org` → Your site
✅ `www.cloudaisolutions.org` → Your site
✅ `https://cloudaisolutions.org` → Secure version
✅ `https://www.cloudaisolutions.org` → Secure version
❌ `deft-licorice-b8a79a.netlify.app` → Still works (redirects to main domain)

## Troubleshooting

### Domain not working after 24 hours?

1. Check DNS records are correct:
   - A record @ → 75.2.60.5
   - CNAME www → deft-licorice-b8a79a.netlify.app

2. Use [https://dnschecker.org](https://dnschecker.org) to verify propagation globally

3. Clear browser cache (Ctrl+Shift+Delete)

4. Try incognito/private mode

5. Check Netlify shows "Domain verified" ✅

### HTTPS not working?

1. Wait 30-60 minutes after DNS propagates
2. Check Netlify Domain settings shows "HTTPS: Active"
3. Click "Verify DNS configuration" in Netlify
4. Netlify auto-renews certificates, no action needed

### Getting SSL errors?

1. Make sure DNS is fully propagated
2. Don't use SSL certificate from your registrar (Netlify provides it)
3. Wait a bit longer - SSL provisioning can take up to 24 hours

### Still showing Netlify URL?

Make sure you set cloudaisolutions.org as the **primary domain** in Netlify settings.

## DNS Configuration Summary

| Record Type | Host | Value | Purpose |
|-------------|------|-------|---------|
| A | @ | 75.2.60.5 | Main domain |
| CNAME | www | deft-licorice-b8a79a.netlify.app | WWW subdomain |

## After Setup Complete

Your site will be accessible at:
- ✅ https://cloudaisolutions.org (primary)
- ✅ https://www.cloudaisolutions.org (redirects to primary)
- ✅ SSL/HTTPS enabled automatically
- ✅ Professional domain for your business!

## Update References

Once domain is live, update these:

1. **Business cards** - Use cloudaisolutions.org
2. **LinkedIn** - Update website field
3. **Email signature** - Add website link
4. **GitHub profile** - Update website
5. **Social media** - Update bio links

## Professional Email

Now that you have the domain, set up professional email:

### Recommended: Zoho Mail (FREE)
1. Go to [https://www.zoho.com/mail](https://www.zoho.com/mail)
2. Sign up for free plan
3. Verify domain ownership (add TXT record)
4. Create: `info@cloudaisolutions.org`
5. Create: `support@cloudaisolutions.org`

### Alternative: Google Workspace ($6/month)
- More professional
- Better integration
- Gmail interface
- Get: `info@cloudaisolutions.org`

## Timeline

| Step | When | Status |
|------|------|--------|
| Domain purchased | Now | ✅ Done |
| Added to Netlify | Today | ⏳ Pending |
| DNS configured | Today | ⏳ Pending |
| DNS propagated | 2-24 hours | ⏳ Waiting |
| SSL enabled | +1-2 hours after DNS | ⏳ Automatic |
| Site live | 3-26 hours total | ⏳ Pending |

## Quick Checklist

- [ ] Added domain in Netlify
- [ ] Added www alias in Netlify
- [ ] Added A record at registrar
- [ ] Added CNAME record at registrar
- [ ] Waited for DNS propagation (check whatsmydns.net)
- [ ] Verified HTTPS is working
- [ ] Set as primary domain
- [ ] Updated sitemap.xml with new URL
- [ ] Updated social media links

## Need Help?

- **DNS not working:** Check records at registrar match exactly
- **Netlify issues:** Check [status.netlify.com](https://status.netlify.com)
- **Domain issues:** Contact your domain registrar support

---

**Once complete, your professional corporate site will be live at:**
## 🌐 https://cloudaisolutions.org ✨

Perfect for business cards, LinkedIn, and professional communications!
