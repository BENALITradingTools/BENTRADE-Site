# BENTRADE — Setup Guide

## Files
- `index.html`    → Main store page
- `product.html`  → Product detail page (uses ?id=zentra etc.)
- `checkout.html` → Payment page (PayPal)
- `thankyou.html` → Post-payment download page
- `images/`       → Put product screenshots here

---

## STEP 1 — Add your logo
In `index.html`, find:
```html
<span class="logo-placeholder">BT</span>
```
Replace with:
```html
<img src="logo.png" alt="BENTRADE">
```
Then upload `logo.png` to the same folder.

---

## STEP 2 — Add your PayPal username
In **both** `checkout.html` and `product.html`, find:
```js
const PAYPAL_ME = 'YOUR_PAYPAL_ME_USERNAME';
```
Replace with your real PayPal.me username (e.g. `bentrade`).
Create your PayPal.me link at: https://www.paypal.com/paypalme/

### Optional: Real PayPal Checkout buttons
1. Go to https://developer.paypal.com/dashboard/
2. Create an app → get your Client ID
3. In `checkout.html`, uncomment the `<script src="paypal sdk...">` line
4. Replace `YOUR_PAYPAL_CLIENT_ID`
5. Uncomment the `paypal.Buttons({...})` block

---

## STEP 3 — Add download links
Upload your EA files (.ex4 / .ex5) to Google Drive.
For each file: Right-click → Share → "Anyone with the link" → Copy link.

In `thankyou.html`, find `DOWNLOAD_LINKS` and replace each placeholder:
```js
'zentra': 'https://drive.google.com/file/d/YOUR_FILE_ID/view',
```

---

## STEP 4 — Add product images
Create a folder called `images/` next to the HTML files.
Add screenshots named like: `zentra-1.jpg`, `zentra-2.jpg`, etc.

In `product.html`, find the product and update `imgs:[]` to:
```js
imgs: ['images/zentra-1.jpg', 'images/zentra-2.jpg', 'images/zentra-3.jpg']
```

---

## STEP 5 — Deploy to GitHub Pages

1. Go to https://github.com → Create new repository (name it `bentrade` or anything)
2. Upload all files: index.html, product.html, checkout.html, thankyou.html, images/
3. Go to Settings → Pages → Source: Deploy from branch → main → / (root) → Save
4. Your site will be live at: `https://YOUR_USERNAME.github.io/bentrade/`

Done! Real site, real payments, real downloads.
