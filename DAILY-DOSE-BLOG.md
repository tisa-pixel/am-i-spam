# Daily Dose - Build Post: Am I Spam?
**For conversionisourgame.com**
**Created:** December 11, 2025

---

## 1. BUILD TITLE
How I Built a Phone Number Reputation Checker to Stop My Calls from Being Marked as Spam

---

## 2. THE PROBLEM
You're running outbound calling campaigns - sales calls, appointment reminders, cold outreach - and your numbers keep getting flagged as spam. Prospects aren't picking up because their phone shows "Scam Likely" before they even hear your pitch.

Most businesses don't know their phone numbers are flagged until it's too late - after weeks of terrible connect rates and wasted ad spend. The tools that DO check number reputation cost $500+/month and are built for enterprise call centers, not small teams.

You just want to know: **Am I spam?** before you launch campaigns.

---

## 3. THE SOLUTION
Built a self-service phone number reputation checker that tells you if your DIDs (Direct Inward Dialing numbers) are flagged as spam before you use them.

Check single numbers instantly or upload a CSV with hundreds of numbers for bulk analysis. The app hits IPQualityScore and NumVerify APIs to get fraud scores, STIR/SHAKEN attestation levels, carrier flagging status, and risk assessments. Everything exports to CSV so you can share with your team or VoIP provider.

Built with Next.js 15, deployed on Vercel. The whole thing costs $0 in infrastructure (free tier) and ~$0.01 per number check via the APIs.

---

## 4. WATCH ME BUILD IT
[YouTube embed code - TBD]

Watch the full walkthrough on YouTube where I break down the API integrations, bulk processing logic, and deployment.

---

## 5. WHAT YOU'LL LEARN
- How to integrate IPQualityScore API for phone fraud detection
- How to build bulk CSV processing in Next.js with real-time progress updates
- How to use server-side API routes to secure API keys
- Understanding STIR/SHAKEN attestation levels (A, B, C)
- Building type-safe APIs with TypeScript
- Implementing toast notifications for user feedback
- Exporting data to CSV from the browser
- Using React Hot Toast for beautiful notifications
- Batch processing patterns to avoid rate limits

---

## 6. BUILD DETAILS

### 6.1 Time Investment
| Who | Time Required |
|-----|---------------|
| **If You Hire a Dev** | 10-12 hours ($1,000-$1,800) |
| **If You Build It (with this guide)** | 4-5 hours |

### 6.2 Cost Breakdown
| Approach | Cost |
|----------|------|
| **Developer Rate** | $100-150/hour |
| **Estimated Dev Cost** | $1,000-$1,800 |
| **DIY Cost (Your Time)** | 4-5 hours + API usage fees |
| **IPQualityScore API** | Free 5,000 credits/month, then ~$0.01/lookup |
| **NumVerify API** | Free 250/month, then $10/month for 1,000 |
| **Hosting (Vercel)** | Free |

**Total Monthly Cost (100 checks/month):** $0 (within free tiers)
**Total Monthly Cost (1,000 checks/month):** ~$10-15

---

## 7. TECH STACK
🔧 **Tools Used:**
- **Next.js 15** (React framework with App Router)
- **TypeScript** (Type safety for APIs and data)
- **Tailwind CSS 4** (Styling)
- **Turbopack** (Ultra-fast dev server - replaces webpack)
- **IPQualityScore API** (Fraud scoring, carrier info, spam detection)
- **NumVerify API** (Phone validation, carrier lookup)
- **PapaParse** (CSV parsing)
- **React Hot Toast** (Beautiful notifications)
- **Lucide React** (Icons)
- **Vercel** (Hosting + serverless functions)

---

## 8. STEP-BY-STEP BREAKDOWN

### 1. **Set Up Next.js Project**

Start by creating a new Next.js 15 project with TypeScript and Tailwind CSS. The CLI wizard walks you through the setup in under a minute. Once the base project is ready, install the additional packages you'll need: PapaParse for CSV processing, React Hot Toast for notifications, Lucide React for icons, and Axios for API calls.

**Why Next.js 15?**
- App Router for modern routing
- Server-side API routes (keep API keys secure)
- Built-in TypeScript support
- Turbopack for blazing fast dev experience

---

### 2. **Get Your API Keys**

**IPQualityScore (Recommended):**
- Sign up at [IPQualityScore](https://www.ipqualityscore.com/)
- Free 5,000 credits/month
- Get API key from dashboard
- Provides fraud scores, carrier info, line type

**NumVerify (Optional backup):**
- Sign up at [NumVerify](https://numverify.com/)
- Free 250 requests/month
- Basic validation and carrier lookup

Create a `.env.local` file in your project root to store these API keys securely. This file is automatically ignored by Git, so your credentials never get committed to your repository. Next.js loads these environment variables server-side, keeping them hidden from browser code.

---

### 3. **Define TypeScript Types**

Before writing any business logic, define the data structures you'll be working with. Create a types file that describes what a reputation check result looks like - including the phone number, validation status, carrier info, location data, and all the reputation metrics like spam score and risk level.

This upfront work pays off immediately: your IDE provides autocomplete, catches typos, and warns you when you're accessing properties that don't exist. When dealing with complex API responses from multiple providers, TypeScript prevents countless runtime bugs.

---

### 4. **Build the API Route for Single Number Validation**

Create a server-side API route that handles phone validation requests. This route receives a phone number from the frontend, calls the IPQualityScore API with your secret key, and transforms the raw response into your standardized format. The API key never leaves the server, so it can't be stolen from browser dev tools.

The route also handles error cases gracefully - missing phone numbers, API failures, and invalid responses all return appropriate error messages. This gives your frontend predictable responses to work with.

---

### 5. **Build Phone Validator Client Library**

Create a client-side library that handles all the phone validation logic your components will need. This class formats phone numbers into a consistent E.164 format (adding country codes, stripping formatting characters), then makes the API calls to your server-side routes.

By encapsulating this logic in a single place, your components stay clean and focused on UI. If you ever need to change how phone numbers are formatted or add caching, you only update one file.

---

### 6. **Build Single Number Form Component**

Create a simple form component that lets users type in a phone number and submit it for checking. The component manages its own input state, handles form submission, and shows loading feedback while the API call is in progress. Disabled states prevent double-submissions when users get impatient.

Keep this component focused on input only - it doesn't know anything about results or how they're displayed. That separation makes it reusable and easy to test.

---

### 7. **Build CSV Upload Component**

Create a drag-and-drop style file upload component for bulk processing. When a user selects a CSV file, PapaParse reads and parses it client-side, extracting all the phone numbers from whatever column structure the file has. Toast notifications give immediate feedback about how many numbers were found.

This component handles the messy reality of user-uploaded files - empty rows, mixed data types, different column formats. It extracts what it can and passes clean data upstream.

---

### 8. **Build Results Table with Export**

Create a results table component that displays all the reputation data in a scannable format. Color-coded risk badges make it easy to spot problem numbers at a glance - green for low risk, yellow for medium, red for high. The table handles both single results and bulk results gracefully.

Include an export button right in the header so users can download their results instantly. This is often the whole point - getting a report they can share with their team or VoIP provider.

---

### 9. **Implement CSV Export**

Create a utility function that transforms your results array into a downloadable CSV file. Map each result object to a row with all the relevant fields - phone number, risk level, spam score, carrier, attestation level, and location data. Handle missing values gracefully with empty strings or "N/A".

The function creates a blob from the CSV content, generates a temporary URL, and triggers a download - all without any server round-trip. The filename includes today's date so users can track when they ran each audit.

---

### 10. **Build Main Page with Tabs**

Wire everything together in your main page component. Use tabs to switch between single-number and bulk-upload modes. Manage the loading state and results array at this level so all child components can share data.

For bulk processing, implement batch logic that processes numbers in groups of 10 to avoid rate limits. Show real-time progress with toast notifications so users know exactly how far along the process is. When complete, highlight any high-risk numbers found.

---

### 11. **Deploy to Vercel**

Deploy your app using the Vercel CLI. The first deployment links your local project to Vercel and sets up automatic deployments for future pushes. Add your API keys as environment variables in the Vercel dashboard - these are encrypted and only available server-side.

Once environment variables are set, run the production deploy command. Your app goes live in seconds with automatic SSL, global CDN distribution, and serverless function scaling. Share the URL with your team and start checking numbers.

---

## 9. GITHUB REPO
📂 **Get the Code:**
[View on GitHub: github.com/tisa-pixel/am-i-spam](https://github.com/tisa-pixel/am-i-spam)

**What's included in the repo:**
- Full Next.js 15 source code (TypeScript)
- API routes for single + bulk validation
- CSV upload/export functionality
- README with setup instructions
- Example environment variables
- Type definitions for all data structures

---

## 10. DOWNLOAD THE TEMPLATE
⬇️ **Download Resources:**
- [Clone the repo](https://github.com/tisa-pixel/am-i-spam) - Full source code ready to deploy
- [.env.example](https://github.com/tisa-pixel/am-i-spam/blob/main/.env.example) - Environment variable template
- [TypeScript Types](https://github.com/tisa-pixel/am-i-spam/blob/main/src/types/index.ts) - Complete type definitions

**Setup Checklist:**
1. Clone repo: `git clone https://github.com/tisa-pixel/am-i-spam.git`
2. Install dependencies: `npm install`
3. Get API keys (IPQualityScore + NumVerify)
4. Create `.env.local` with keys
5. Run locally: `npm run dev`
6. Test single number check
7. Test bulk CSV upload
8. Deploy to Vercel: `vercel --prod`
9. Set environment variables in Vercel
10. Test it live!

---

## 11. QUESTIONS? DROP THEM BELOW
💬 **Have questions or want to share your results?**
- Comment on the [YouTube video](#) (TBD)
- DM me on Instagram: [@donottakeifallergictorevenue](https://www.instagram.com/donottakeifallergictorevenue/)
- Open an issue on [GitHub](https://github.com/tisa-pixel/am-i-spam/issues)

---

## 12. RELATED BUILDS
| Build 1 | Build 2 | Build 3 |
|---------|---------|---------|
| **How I Built Check Yo Rep - Civic Engagement Tool** | **How I Built a Retell AI Dialer for $0** | **How I Automated Lead Routing in Salesforce** |
| Find all your elected officials from City Hall to Capitol Hill | Built an outbound calling system using Retell AI and Claude | Smart lead distribution based on rep capacity and time zones |
| [View Build](https://github.com/tisa-pixel/check-yo-rep) | [View Build] | [View Build] |

---

## Additional Metadata (for SEO / Backend)

**Published:** December 11, 2025
**Author:** Tisa Daniels
**Category:** Phone Systems / API Integration / Next.js
**Tags:** #PhoneReputation #SpamDetection #STIRSHAKEN #NextJS #TypeScript #OutboundCalling #DID #PhoneValidation
**Estimated Read Time:** 15 minutes
**Video Duration:** TBD

---

## Design Notes for Wix Implementation

### Layout Style:
- **Dark background** (charcoal #1B1C1D)
- **High contrast text** (white headings, light gray body)
- **Accent colors:** Blue (#2563eb), Green (#16a34a for "low risk"), Red (#dc2626 for "high risk"), Yellow (#eab308 for "medium risk")
- **Clean, modern, mobile-first**

### Call-to-Action Buttons:
- **Primary CTA** (Clone on GitHub): Purple (#7c3aed)
- **Secondary CTA** (Watch on YouTube): Blue (#2563eb)
- **Export CSV**: Green (#16a34a)

### Interactive Elements:
- Risk level badges with color coding
- Real-time toast notifications demo
- CSV upload interaction preview

---

## Use Cases

**Real Estate Investors:**
- Check DIDs before cold calling campaigns
- Verify numbers aren't flagged before Mojo/Batch dialing
- Audit entire number pool quarterly

**Sales Teams:**
- Pre-check outbound numbers before SDR onboarding
- Identify which numbers to rotate out
- Share reports with VoIP provider for remediation

**Call Centers:**
- Bulk audit all caller IDs monthly
- Catch spam flags before they kill connect rates
- Export data for compliance reporting

**Marketing Agencies:**
- Check client phone numbers before SMS campaigns
- Verify reputation before launching appointment reminders
- Audit twilio/telnyx number pools

---

**Template Version:** 1.0
**Created:** December 11, 2025
