# Product Recognition + Pricing Plan (Next.js + Netlify)

This doc tailors the MVP approach to a **Next.js** app deployed on **Netlify**, using **Netlify Functions** for server-side integrations.

## 1) Recognition approach (barcode-first with cloud fallback)

**Recommendation:** Start with **barcode/QR scanning** in the browser. Use a **cloud vision API** only when the barcode path fails or is unavailable. Defer on-device ML (TensorFlow.js) until demand is proven.

**Why in this stack:**
- Barcode scanning is fast, privacy-friendly, and stable in a browser.
- Cloud vision is easier to integrate through Netlify Functions without shipping heavy ML to clients.
- On-device ML adds bundle size and performance complexity for a minimal MVP.

**Implementation notes (Next.js + Netlify):**
- Client: Use a barcode scanning library that works with a camera stream (e.g., `@zxing/browser` or `html5-qrcode`).
- Server: Create a Netlify Function (e.g., `netlify/functions/vision-lookup.js`) that forwards image payloads to a cloud vision API (Google Vision or AWS Rekognition).

## 2) Minimum viable recognition input

**MVP input:** **UPC/EAN from barcode**.

**Fallback input:** **Image → inferred product name** (cloud vision). Keep this as “unverified” until validated.

**Why:** UPC/EAN is a stable canonical identifier, enabling consistent price lookups and de-duplication.

## 3) Price sources

**Preferred order:**
1. **Retailer APIs** (stable, compliant, maintainable)
2. **Community-submitted entries** (coverage gap filler)
3. **Scraping** (only if explicitly allowed)

**Implementation notes (Next.js + Netlify):**
- Create a provider interface that supports multiple sources.
- Use Netlify Functions to proxy API calls and protect API keys.
- Add a lightweight admin/moderation queue for community-submitted prices.

## 4) Product identifier format

**Primary identifier:** `upc_ean` (string)

**Optional identifiers:**
- `sku` (retailer-specific)
- `normalized_name` (for human readability/fallback)

**Normalization rules:**
- Store UPC/EAN as a **string** to preserve leading zeros.
- Validate length (UPC-A 12 digits, EAN-13 13 digits).
- Track source and confidence on any inferred name.

## Suggested data model (minimal)

```json
{
  "product_id": "012345678905",
  "id_type": "upc_ean",
  "normalized_name": "Sample Product 12oz",
  "sku": "ABC-123",
  "source": "api",
  "source_ref": "retailer_api_name",
  "price": 3.99,
  "currency": "USD",
  "last_verified_at": "2024-01-01T00:00:00Z"
}
```

## Recommended MVP flow

1. User scans barcode.
2. Client sends UPC/EAN to a Netlify Function.
3. Function queries retailer API(s) and community database.
4. If no match and user opts in, allow image upload to vision endpoint.
5. Save any community entry with moderation/verification status.
