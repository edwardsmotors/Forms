# Edwards Motors Kia PBV enquiry form — Microsoft 365 / SharePoint

## What is included
- `index.html`: anonymous customer form, preserving the supplied branding and questions.
- `staff.html`: Microsoft 365 sign-in, centrally loaded enquiries, Excel copy, CSV export, share/email fallback.
- `api/`: Azure Functions that create and retrieve SharePoint List items through Microsoft Graph.
- `staticwebapp.config.json`: Azure Static Web Apps routing and authentication rules.

**This is a deployment-ready starter, not an already-hosted service.** Microsoft 365 administrator configuration, deployment, and end-to-end testing are required. No actual customer data is included.

## 1. Create the SharePoint List
Create a SharePoint List called **Kia PBV Enquiries** with these **single line of text** columns (unless otherwise noted):

| Display/internal column name | Purpose |
| --- | --- |
| Title | Customer name (built-in column) |
| FirstName | First name |
| Surname | Surname |
| Address | Street address |
| Town | Town |
| County | County |
| Postcode | Postcode |
| Phone | Phone number |
| Email | Email address |
| InterestedIn | Vehicle type |
| PurchaseTimescale | When buying |
| PartExchange | Part exchange |
| Notes | Multiline plain text |
| MarketingConsent | Yes / No (single line text) |

Create the columns with these exact names as the **internal names**. If SharePoint changes an internal name, adjust `names` in `api/src/functions/enquiries.js`. SharePoint automatically stores Created timestamp. Restrict list permissions to authorised sales staff; disable external sharing of the list.

## 2. Register the Graph application
In Microsoft Entra ID, register a **single-tenant** application for the backend. Create a client secret (prefer a managed identity or certificate for production). Grant **Microsoft Graph application `Sites.Selected`** permission with admin consent. Grant that app **write** access to the specific SharePoint site using the Microsoft Graph site-permissions endpoint (an administrator must do this). Avoid tenant-wide `Sites.ReadWrite.All` where possible.

Collect `TENANT_ID`, `CLIENT_ID`, `CLIENT_SECRET`, `SITE_ID`, and `LIST_ID`. Keep secrets in Azure app settings, never in HTML or Git. Use Key Vault where possible.

## 3. Deploy to Azure Static Web Apps
Create an Azure Static Web App (Standard plan is recommended for custom Entra authentication) from a private GitHub or Azure DevOps repository containing this folder. Set app location `/`, API location `api`, output location empty. Add the five environment variables above to the backend application settings.

Configure **custom Microsoft Entra ID authentication** for Azure Static Web Apps with your organisation's single-tenant Entra application. The provider must be named `aad` for the built-in login route `/.auth/login/aad`. Restrict sign-in to the Edwards Motors tenant and authorised sales staff. The backend also checks the tenant claim on every staff data request. Set appropriate Entra enterprise application assignment rules for staff access.

**Important:** The included `staticwebapp.config.json` supplies route protection, but custom Entra authentication settings and the identity provider client secret must be configured in Azure before launch. Do not rely on the original four-digit PIN.

## 4. Verify before the show
1. Open the customer URL in an incognito browser without signing in. Submit a **fictional test** enquiry.
2. Confirm the record appears in the SharePoint List. If SharePoint is unavailable, the form must show an error, not a success message.
3. Confirm `/staff.html` redirects unauthenticated visitors to Microsoft sign-in.
4. Confirm an authorised Edwards Motors account can see the test enquiry, copy it into Excel, and download CSV.
5. Confirm an unauthorised Microsoft account and an unauthenticated direct API request cannot retrieve enquiries.
6. Test the share button on an iPad and desktop. On browsers without native file sharing, it downloads a CSV to attach to Outlook.
7. Configure retention, access review, privacy notice, and marketing-consent handling to match your organisation's UK GDPR policies.
8. Add rate limiting/bot protection at the edge (e.g. Azure Front Door WAF or equivalent) before publicly advertising the anonymous form. Anonymous submissions must be protected against abuse.

## Notes
- The customer form is anonymous; staff access requires Microsoft 365.
- Leads are never cached in `localStorage`.
- No bulk deletion function is included, to avoid accidental loss of centrally held customer information.
- The app currently retrieves up to 5,000 records; for larger events, implement server-side filtering/pagination.
- CSV exported by staff contains personal data. Send it only to authorised recipients.
