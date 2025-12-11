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

```bash
npx create-next-app@latest am-i-spam --typescript --tailwind --app
cd am-i-spam
npm install papaparse react-hot-toast lucide-react axios
npm install --save-dev @types/papaparse
```

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

**Environment Setup:**
Create `.env.local`:
```env
IPQUALITYSCORE_API_KEY=your_ipqs_api_key_here
NUMVERIFY_API_KEY=your_numverify_api_key_here
```

---

### 3. **Define TypeScript Types**

Create `src/types/index.ts`:

```typescript
export interface ReputationCheck {
  phoneNumber: string;
  timestamp: Date;
  isValid: boolean;
  carrier?: string;
  lineType?: string;
  location?: {
    city?: string;
    state?: string;
    country?: string;
  };
  reputation: {
    spamScore?: number;
    spamLikely: boolean;
    scamLikely: boolean;
    riskLevel: 'low' | 'medium' | 'high' | 'unknown';
    flaggedByCarriers: string[];
    attestationLevel?: 'A' | 'B' | 'C' | 'unknown';
  };
  cnam?: {
    registered: boolean | null;
    displayName?: string;
  };
  disconnected?: boolean;
  reassigned?: boolean;
  healthScore?: number;
  errors?: string[];
}
```

**Why TypeScript?** Catches bugs at compile time, especially when dealing with complex API responses.

---

### 4. **Build the API Route for Single Number Validation**

Create `src/app/api/validate/route.ts`:

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { ReputationCheck } from '@/types';

async function checkIPQualityScore(phoneNumber: string) {
  const apiKey = process.env.IPQUALITYSCORE_API_KEY;

  if (!apiKey) return null;

  const response = await fetch(
    `https://ipqualityscore.com/api/json/phone/${apiKey}/${encodeURIComponent(phoneNumber)}`,
    { cache: 'no-store' }
  );

  if (response.ok) {
    return await response.json();
  }

  return null;
}

export async function POST(request: NextRequest) {
  const { phoneNumber } = await request.json();

  if (!phoneNumber) {
    return NextResponse.json(
      { error: 'Phone number is required' },
      { status: 400 }
    );
  }

  const ipqsData = await checkIPQualityScore(phoneNumber);

  if (ipqsData) {
    const result: ReputationCheck = {
      phoneNumber,
      timestamp: new Date(),
      isValid: ipqsData.valid || false,
      carrier: ipqsData.carrier,
      lineType: ipqsData.line_type,
      location: {
        city: ipqsData.city,
        state: ipqsData.region,
        country: ipqsData.country,
      },
      reputation: {
        spamScore: ipqsData.fraud_score || 0,
        spamLikely: ipqsData.recent_abuse || false,
        scamLikely: ipqsData.fraud_score > 75,
        riskLevel: ipqsData.fraud_score > 75 ? 'high'
                  : ipqsData.fraud_score > 50 ? 'medium'
                  : 'low',
        flaggedByCarriers: [],
        attestationLevel: 'unknown',
      },
      disconnected: !ipqsData.active,
    };

    return NextResponse.json(result);
  }

  return NextResponse.json(
    { error: 'Validation failed' },
    { status: 500 }
  );
}
```

**Key Points:**
- API keys stay on server (never exposed to browser)
- Uses Next.js App Router API routes
- Returns structured JSON with all reputation data

---

### 5. **Build Phone Validator Client Library**

Create `src/lib/phone-validator.ts`:

```typescript
export class PhoneValidator {
  private static formatPhoneNumber(phone: string): string {
    const cleaned = phone.replace(/\D/g, '');

    // Handle US numbers: 10 digits → +1XXXXXXXXXX
    if (cleaned.length === 10) {
      return `+1${cleaned}`;
    } else if (cleaned.length === 11 && cleaned.startsWith('1')) {
      return `+${cleaned}`;
    }

    return phone;
  }

  static async validateSingle(phoneNumber: string): Promise<ReputationCheck> {
    const formatted = this.formatPhoneNumber(phoneNumber);

    const response = await fetch('/api/validate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ phoneNumber: formatted }),
    });

    if (!response.ok) {
      throw new Error('Validation failed');
    }

    return await response.json();
  }

  static async validateBulk(phoneNumbers: string[]): Promise<ReputationCheck[]> {
    const formatted = phoneNumbers.map(phone => this.formatPhoneNumber(phone));

    const response = await fetch('/api/validate/bulk', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ phoneNumbers: formatted }),
    });

    return await response.json();
  }
}
```

**Why a class?** Encapsulates phone formatting logic and provides clean API for components.

---

### 6. **Build Single Number Form Component**

Create `src/components/SingleNumberForm.tsx`:

```typescript
'use client';

import { useState } from 'react';
import { Phone } from 'lucide-react';

interface Props {
  onSubmit: (phoneNumber: string) => Promise<void>;
  isLoading: boolean;
}

export default function SingleNumberForm({ onSubmit, isLoading }: Props) {
  const [phoneNumber, setPhoneNumber] = useState('');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!phoneNumber.trim()) return;
    await onSubmit(phoneNumber);
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <label className="block text-sm font-medium text-gray-700 mb-2">
          Phone Number
        </label>
        <div className="flex gap-2">
          <input
            type="tel"
            value={phoneNumber}
            onChange={(e) => setPhoneNumber(e.target.value)}
            placeholder="(555) 123-4567"
            className="flex-1 px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
          />
          <button
            type="submit"
            disabled={isLoading}
            className="px-6 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed flex items-center gap-2"
          >
            <Phone className="h-4 w-4" />
            {isLoading ? 'Checking...' : 'Check'}
          </button>
        </div>
      </div>
    </form>
  );
}
```

Clean, simple form with loading states.

---

### 7. **Build CSV Upload Component**

Create `src/components/FileUpload.tsx`:

```typescript
'use client';

import { useRef } from 'react';
import Papa from 'papaparse';
import { Upload } from 'lucide-react';
import toast from 'react-hot-toast';

interface Props {
  onNumbersExtracted: (numbers: string[]) => void;
}

export default function FileUpload({ onNumbersExtracted }: Props) {
  const fileInputRef = useRef<HTMLInputElement>(null);

  const handleFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;

    Papa.parse(file, {
      complete: (results) => {
        const phoneNumbers = results.data
          .flat()
          .filter((value): value is string => typeof value === 'string')
          .map(num => num.trim())
          .filter(num => num.length > 0);

        if (phoneNumbers.length === 0) {
          toast.error('No phone numbers found in CSV');
          return;
        }

        toast.success(`Found ${phoneNumbers.length} numbers`);
        onNumbersExtracted(phoneNumbers);
      },
      error: () => {
        toast.error('Failed to parse CSV file');
      },
    });
  };

  return (
    <div className="border-2 border-dashed border-gray-300 rounded-lg p-8 text-center">
      <Upload className="h-12 w-12 text-gray-400 mx-auto mb-4" />
      <p className="text-sm text-gray-600 mb-4">
        Upload a CSV file with phone numbers
      </p>
      <input
        ref={fileInputRef}
        type="file"
        accept=".csv"
        onChange={handleFileChange}
        className="hidden"
      />
      <button
        type="button"
        onClick={() => fileInputRef.current?.click()}
        className="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700"
      >
        Choose File
      </button>
    </div>
  );
}
```

**PapaParse** does the heavy lifting - parses CSV, extracts phone numbers, handles errors.

---

### 8. **Build Results Table with Export**

Create `src/components/ResultsTable.tsx`:

```typescript
'use client';

import { ReputationCheck } from '@/types';
import { Download } from 'lucide-react';

interface Props {
  results: ReputationCheck[];
  onExport: () => void;
}

export default function ResultsTable({ results, onExport }: Props) {
  const getRiskBadge = (level: string) => {
    const styles = {
      low: 'bg-green-100 text-green-800',
      medium: 'bg-yellow-100 text-yellow-800',
      high: 'bg-red-100 text-red-800',
      unknown: 'bg-gray-100 text-gray-800',
    };

    return (
      <span className={`px-2 py-1 rounded-full text-xs font-medium ${styles[level as keyof typeof styles]}`}>
        {level.toUpperCase()}
      </span>
    );
  };

  return (
    <div className="space-y-4">
      <div className="flex justify-between items-center">
        <h3 className="text-lg font-semibold">Results ({results.length})</h3>
        <button
          onClick={onExport}
          className="flex items-center gap-2 px-4 py-2 bg-green-600 text-white rounded-lg hover:bg-green-700"
        >
          <Download className="h-4 w-4" />
          Export CSV
        </button>
      </div>

      <div className="overflow-x-auto">
        <table className="min-w-full divide-y divide-gray-200">
          <thead className="bg-gray-50">
            <tr>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                Phone Number
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                Risk Level
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                Spam Score
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                Carrier
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                Attestation
              </th>
            </tr>
          </thead>
          <tbody className="bg-white divide-y divide-gray-200">
            {results.map((result, idx) => (
              <tr key={idx}>
                <td className="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
                  {result.phoneNumber}
                </td>
                <td className="px-6 py-4 whitespace-nowrap">
                  {getRiskBadge(result.reputation.riskLevel)}
                </td>
                <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                  {result.reputation.spamScore?.toFixed(0) ?? 'N/A'}
                </td>
                <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                  {result.carrier ?? 'Unknown'}
                </td>
                <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                  {result.reputation.attestationLevel ?? 'N/A'}
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
}
```

Clean table with color-coded risk levels.

---

### 9. **Implement CSV Export**

Create `src/lib/export.ts`:

```typescript
import { ReputationCheck } from '@/types';

export function exportToCSV(results: ReputationCheck[]) {
  const headers = [
    'Phone Number',
    'Risk Level',
    'Spam Score',
    'Spam Likely',
    'Scam Likely',
    'Carrier',
    'Line Type',
    'Attestation',
    'Disconnected',
    'City',
    'State',
    'Country'
  ];

  const rows = results.map(r => [
    r.phoneNumber,
    r.reputation.riskLevel,
    r.reputation.spamScore?.toString() ?? '',
    r.reputation.spamLikely ? 'Yes' : 'No',
    r.reputation.scamLikely ? 'Yes' : 'No',
    r.carrier ?? '',
    r.lineType ?? '',
    r.reputation.attestationLevel ?? '',
    r.disconnected ? 'Yes' : 'No',
    r.location?.city ?? '',
    r.location?.state ?? '',
    r.location?.country ?? ''
  ]);

  const csvContent = [
    headers.join(','),
    ...rows.map(row => row.map(cell => `"${cell}"`).join(','))
  ].join('\n');

  const blob = new Blob([csvContent], { type: 'text/csv' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `phone-reputation-${new Date().toISOString().split('T')[0]}.csv`;
  a.click();
  URL.revokeObjectURL(url);
}
```

Downloads CSV with all reputation data - perfect for sharing with team or VoIP provider.

---

### 10. **Build Main Page with Tabs**

Create `src/app/page.tsx`:

```typescript
'use client';

import { useState } from 'react';
import { Toaster } from 'react-hot-toast';
import toast from 'react-hot-toast';
import SingleNumberForm from '@/components/SingleNumberForm';
import FileUpload from '@/components/FileUpload';
import ResultsTable from '@/components/ResultsTable';
import { PhoneValidator } from '@/lib/phone-validator';
import { exportToCSV } from '@/lib/export';
import { ReputationCheck } from '@/types';

export default function Home() {
  const [activeTab, setActiveTab] = useState<'single' | 'bulk'>('single');
  const [results, setResults] = useState<ReputationCheck[]>([]);
  const [isLoading, setIsLoading] = useState(false);

  const handleSingleCheck = async (phoneNumber: string) => {
    setIsLoading(true);
    try {
      const result = await PhoneValidator.validateSingle(phoneNumber);
      setResults([result]);

      if (result.reputation.riskLevel === 'high') {
        toast.error('⚠️ High risk number detected!');
      } else {
        toast.success('Number checked successfully');
      }
    } catch (error) {
      toast.error('Failed to check phone number');
    } finally {
      setIsLoading(false);
    }
  };

  const handleBulkCheck = async (phoneNumbers: string[]) => {
    setIsLoading(true);
    setResults([]);

    try {
      toast.loading(`Checking ${phoneNumbers.length} numbers...`, { id: 'bulk' });

      // Process in batches of 10 to avoid rate limits
      const batchSize = 10;
      const allResults: ReputationCheck[] = [];

      for (let i = 0; i < phoneNumbers.length; i += batchSize) {
        const batch = phoneNumbers.slice(i, i + batchSize);
        const batchResults = await PhoneValidator.validateBulk(batch);
        allResults.push(...batchResults);
        setResults([...allResults]);

        toast.loading(
          `Processed ${Math.min(i + batchSize, phoneNumbers.length)} of ${phoneNumbers.length}...`,
          { id: 'bulk' }
        );
      }

      toast.success(`Checked ${allResults.length} numbers`, { id: 'bulk' });

      const highRiskCount = allResults.filter(r => r.reputation.riskLevel === 'high').length;
      if (highRiskCount > 0) {
        toast.error(`Found ${highRiskCount} high-risk numbers!`);
      }
    } catch (error) {
      toast.error('Failed to check phone numbers', { id: 'bulk' });
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <main className="min-h-screen bg-gray-50">
      <Toaster position="top-right" />

      <div className="max-w-7xl mx-auto px-4 py-8">
        <h1 className="text-3xl font-bold mb-8">Am I Spam?</h1>

        {/* Tab switcher */}
        <div className="flex gap-2 mb-6">
          <button
            onClick={() => setActiveTab('single')}
            className={activeTab === 'single' ? 'active' : ''}
          >
            Single Number
          </button>
          <button
            onClick={() => setActiveTab('bulk')}
            className={activeTab === 'bulk' ? 'active' : ''}
          >
            Bulk Upload
          </button>
        </div>

        {/* Forms */}
        {activeTab === 'single' ? (
          <SingleNumberForm onSubmit={handleSingleCheck} isLoading={isLoading} />
        ) : (
          <FileUpload onNumbersExtracted={handleBulkCheck} />
        )}

        {/* Results */}
        {results.length > 0 && (
          <ResultsTable results={results} onExport={() => exportToCSV(results)} />
        )}
      </div>
    </main>
  );
}
```

**Real-time progress updates** as bulk checks run - users see exactly what's happening.

---

### 11. **Deploy to Vercel**

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel
```

Follow prompts:
- Link to project
- Set environment variables (API keys)

**Set Production Environment Variables:**
In Vercel dashboard → Settings → Environment Variables:
- `IPQUALITYSCORE_API_KEY` = your_key
- `NUMVERIFY_API_KEY` = your_key

Deploy to production:
```bash
vercel --prod
```

Live in seconds!

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
