# BENTRADE Site — Setup & Configuration Guide

## Files in this release
| File | Status |
|------|--------|
| `index.html` | ✅ Fixed |
| `product.html` | ✅ Fixed |
| `checkout.html` | ✅ Fixed |
| `thankyou.html` | ✅ Fixed (verification gate) |

---

## Bugs Fixed

### 1. Images not square
**Problem:** Product card images had no fixed aspect ratio — they stretched or squished depending on the source image dimensions.

**Fix:** Applied `aspect-ratio: 1/1` + `object-fit: cover` to every image container on both `index.html` (homepage cards) and `product.html` (main image + thumbnails). Images now always render as a perfect square, cropping from the center.

```css
/* homepage card gallery */
.card-gallery {
  aspect-ratio: 1/1;
  overflow: hidden;
}
.card-gallery img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* product page main image */
.main-img-wrap {
  aspect-ratio: 1/1;
  overflow: hidden;
}
.main-img-wrap img {
  position: absolute;
  inset: 0;
  width: 100%; height: 100%;
  object-fit: cover;
}

/* product page thumbnails */
.thumb {
  width: 72px;
  height: 72px;     /* explicit equal W+H = square */
  object-fit: cover;
}
```

---

### 2. Single shared PayPal link for all products
**Problem:** All products pointed to the same single PayPal link, making it impossible to track which product was purchased or to use product-specific payment pages.

**Fix:** Each product now has its own unique PayPal link defined in two places:

**`product.html` — `PRODUCTS` object:**
```js
'zentra':       { paypal: 'https://www.paypal.com/ncp/payment/XXXX1', ... },
'golden-guard': { paypal: 'https://www.paypal.com/ncp/payment/XXXX2', ... },
'gold-strike':  { paypal: 'https://www.paypal.com/ncp/payment/HYH9HMAKX5HE8', ... }, // ✓ real
// etc.
```

**`checkout.html` — `PAYPAL_LINKS` object:**
```js
const PAYPAL_LINKS = {
  'zentra':       'https://www.paypal.com/ncp/payment/XXXX1',
  'golden-guard': 'https://www.paypal.com/ncp/payment/XXXX2',
  'gold-strike':  'https://www.paypal.com/ncp/payment/HYH9HMAKX5HE8', // ✓ real
  // etc.
};
```

If a link is still a placeholder (`XXXX`), the checkout page shows a clear error banner and disables the payment buttons automatically.

---

### 3. No payment verification before download
**Problem:** `thankyou.html` showed the download link to anyone who navigated to the page — no check that they actually paid.

**Fix:** A verification gate now blocks the download link until the customer enters their PayPal Transaction ID. Three verification tiers are available:

#### Tier A — Format validation only (default, zero setup)
PayPal Transaction IDs are always exactly **17 uppercase alphanumeric characters** (e.g. `7RJ35724XC123456P`). The gate validates the format. Stops casual URL-guessing.

#### Tier B — Known TX ID list (manual, more secure)
Add real Transaction IDs to the set in `thankyou.html`:
```js
const VERIFIED_TX_IDS = new Set([
  'HYH9HMAKX5HE81234',
  'ANOTHER_TX_ID_HERE',
]);
```
Only those exact IDs will unlock the download.

#### Tier C — Server-side API verification (most secure, for production)
Uncomment the `fetch('/api/verify-payment', ...)` block in `thankyou.html` and deploy a serverless function that calls the [PayPal Orders API](https://developer.paypal.com/docs/api/orders/v2/#orders_get):
```
GET https://api-m.paypal.com/v2/checkout/orders/{txId}
Authorization: Bearer {access_token}
```
Return `{ valid: true }` if `status === "COMPLETED"` and `purchase_units[0].amount.value` matches the expected price.

---

## Setup Checklist

### Step 1 — Add your real PayPal links
For each product, create a PayPal.com payment link:
1. Log in to paypal.com → **Pay & Get Paid** → **Payment Links** → **Create payment link**
2. Set the amount, title, and currency
3. Copy the resulting URL (format: `https://www.paypal.com/ncp/payment/XXXXXXXXXXXXXXXXX`)
4. Paste it into **both** `product.html` and `checkout.html` for that product ID

### Step 2 — Add your download files
In `thankyou.html`, replace each `YOUR_XXX_FILE_ID` with a real Google Drive file ID:
```js
const DOWNLOAD_LINKS = {
  'zentra':      'https://drive.google.com/file/d/REAL_FILE_ID/view',
  'gold-strike': 'https://drive.google.com/file/d/REAL_FILE_ID/view',
  // etc.
};
```
To get a Drive file ID: upload the `.ex4`/`.ex5` file → right-click → **Share** → **Copy link** → the ID is the long string between `/d/` and `/view`.

Make sure the file sharing is set to **"Anyone with the link can view"**.

### Step 3 — Add product images
Place images in the `images/` folder (create it next to `index.html`):
```
images/
  zentra-1.jpg
  zentra-2.jpg
  golden-guard-1.jpg
  golden-algo-1.jpg
  prime-1.jpg
  forex-strike-1.jpg
  bitrocket-1.jpg
```
In `product.html`, update each product's `imgs` array:
```js
'zentra': { imgs: ['images/zentra-1.jpg', 'images/zentra-2.jpg'], ... }
```
In `index.html`, replace each `<!-- <img src="images/xxx.jpg"> -->` comment with a real `<img>` tag inside `.card-gallery`.

`GOLD-Strike.png` is already wired up in both files since it exists in the repo root.

### Step 4 — (Optional) Enable server-side payment verification
For production security, deploy a serverless function and uncomment the `fetch('/api/verify-payment')` block in `thankyou.html`. 

Recommended platforms: **Netlify Functions**, **Vercel**, **Cloudflare Workers**, or **Firebase Functions**.

---

## URL Flow

```
index.html
  └─ Buy button ──────────────────────────────────────────────────────┐
  └─ Details button → product.html?id=gold-strike                     │
                          └─ Buy Now → checkout.html?id=...&price=... ┘
                                           └─ PayPal.com (payment)
                                               └─ thankyou.html?id=...
                                                     └─ [TX ID gate]
                                                         └─ Download link
```

---

## Other Bugs Noticed & Fixed

| # | Issue | Fix |
|---|-------|-----|
| 1 | `imgs:[GOLD-Strike.png]` — missing quotes in JS array | Fixed to `imgs:['GOLD-Strike.png']` |
| 2 | Language preference not persisted across pages | Cookie + localStorage fallback added to all pages |
| 3 | `product.html` built checkout URL without `lang` param | Added `&lang=` so checkout page inherits language |
| 4 | No validation on PayPal links — broken `#` links silently failed | Error banner + button disable when link is placeholder |
| 5 | `thankyou.html` showed download to anyone visiting the page | Payment verification gate added |
| 6 | `<form>` tags present in original — incompatible with some renderers | Converted all to `button onClick` handlers |
| 7 | Direction flash on page load (RTL→LTR flicker) | `dir` set synchronously before DOMContentLoaded |
