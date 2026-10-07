# Click2Check: Contract Setup Form

**Live demo:** https://click2checkdev.github.io/Contract-setup-demo/

An online version of the *Company information for contract set up* spreadsheet. A client fills in the form and
presses **Send**. The page builds their End User Licence Agreement (.docx) with every highlighted section filled in,
plus a company information sheet (.xlsx), and emails them:

- **To the Click2Check team:** the contract and the company information sheet, with reply-to set to the client.
- **To the client:** a copy of their contract.

**Demo mode:** no email is actually sent. After Send, the page shows the two emails it would send, and the
attachments can be opened. See *Connecting real email* below.

Prices are deliberately left out. Every fee in the contract comes out as `£ ________` for Click2Check staff to
complete before the contract goes for signature, so this public repo holds no pricing.

## Files

| File | Purpose |
|---|---|
| `index.html` | The form, contract and spreadsheet generation, and Send |
| `template.docx` | The contract with each client detail and price replaced by a `{{placeholder}}` |
| `template.js` | `template.docx` embedded as base64 (lets the page work from any host, even double-clicked) |
| `tools/build_template.py` | Rebuilds `template.docx` + `template.js` from the source contract |

## Field mapping

| Contract text | Placeholder | Filled from |
|---|---|---|
| Date the terms are dated | `contract_date` | Date the form is sent |
| Client company name (×2, incl. signature block) | `company_name` | Company name |
| Abbreviation in brackets (×2) | `short_name` | Short name (auto from initials, editable) |
| "trading as" name | `trading_name` | Trading name (defaults to company name) |
| Company number | `company_reg` | Company registration number |
| Registered office address | `registered_address` | Building, street, city, county, postcode |
| Company telephone number | `phone` | Head office phone number |
| Signatory ("Signed by …") | `director_name` | Director's name |
| Fee sheet: monthly fee, report price, credit allocation, set-up fee, set-up charge, total | `monthly_fee`, `report_price`, `subscription_amount`, `setup_fee`, `setup_charged`, `total_usage` | Left blank for staff |
| Administration charge example: credit file fee, admin fee, total | `credit_file_fee`, `admin_fee`, `example_total` | Left blank for staff |

The other spreadsheet fields aren't in the contract: legal status, FCA number, DA/AR, network, contacts,
solutions, future modules and users. They go into the company information .xlsx.

Left unchanged in the contract: the renewal worked example (£10 / £100), the £1,000 liability cap, the 20%
admin-charge cap and the Quilter wording ("Qulter Price", "No cost to Quilter Adviser"). If this form will be
used for non-Quilter clients, the Quilter lines should become placeholders too.

## Connecting real email

GitHub Pages can't send email, so Send posts to an email endpoint. In `index.html`, set:

```js
const CONFIG = {
  SUBMIT_URL: "https://…",                 // null = demo mode
  TEAM_EMAIL: "onboarding@click2check.com", // where the team's copy goes
};
```

The page POSTs JSON as `text/plain` (no CORS preflight):
`{ data: {...form fields}, emails: [{ to, replyTo, subject, body, attachments: [{ name, base64 }] }] }`.
The endpoint only has to send each email in `emails`. Options:

- **Google Apps Script** (free, no DNS access needed): a `doPost(e)` that parses the JSON and calls
  `MailApp.sendEmail` with `Utilities.newBlob(Utilities.base64Decode(a.base64), null, a.name)` attachments.
  Deploy it as a web app with access set to *Anyone*.
- **Microsoft Power Automate:** an "HTTP request received" flow that sends from the Microsoft 365 mailbox
  (needs a Premium licence).
- **A serverless function** (Netlify/Vercel/Cloudflare) with an email API such as Resend or SendGrid. This needs
  DNS access to send from @click2check.com.

Optional later step: send the contract to DocuSign or Adobe Sign for e-signature instead of by email.

## Running locally

Open `index.html` in a browser (needs internet for the CDN libraries), or run:

    python -m http.server 8765 --directory contract-form

## If the contract wording changes

Edit the source .docx, keeping the yellow highlights where they are, then run this from the repo folder:

    python tools/build_template.py "path/to/new contract.docx"

After rebuilding, bump the `?v=` number on the `template.js` script tag in `index.html` so browsers fetch the
new template.

The script checks that every section it replaces is highlighted, or holds a price, in the source. If the layout
has moved, it stops with an error rather than silently producing a broken template.

## Going live on click2check.com

click2check.com runs on **Squarespace**, which can't host these files. Keep the form on GitHub Pages and then
either point a subdomain such as `onboarding.click2check.com` at it, or link or embed it from a Squarespace page:
`<iframe src="https://click2checkdev.github.io/Contract-setup-demo/" style="width:100%;height:2200px;border:0"></iframe>`
