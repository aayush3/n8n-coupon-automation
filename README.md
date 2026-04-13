# 🎁 Automated Coupon Campaign — n8n

An end-to-end automated coupon distribution system I built using **n8n**, **Google Sheets**, and **Gmail**. The system handles the full coupon lifecycle — from generation to expiry — without any manual work after setup.

---

## 💡 Why I Built This

Managing coupon campaigns manually is messy — copy-pasting codes, remembering to send reminders, updating spreadsheets when coupons expire. I built this to automate the entire process so it runs on its own once triggered.

---

## ⚙️ What It Does

1. Reads a recipient list from Google Sheets
2. Filters out anyone already active or not opted in
3. Generates a unique personalized coupon code per recipient
4. Logs the coupon to a tracking sheet with status, timestamps, and usage limits
5. Sends a branded HTML coupon email to each recipient
6. Automatically sends an expiry reminder before the coupon dies
7. Marks the coupon as `EXPIRED` in Google Sheets
8. Sends the user a friendly expiry notification
9. Triggers a subflow that emails the admin a daily digest summary

---

## 🛠 Stack

| Tool | Role |
|------|------|
| n8n | Automation engine |
| Google Sheets | Recipient list & coupon log |
| Gmail | All email delivery |

---

## 📁 Repo Structure

```
├── workflow.json               → Main automation workflow (import into n8n)
├── email-digest-subflow.json   → Admin digest subflow (import separately)
└── docs/
    └── setup-guide.md          → Full setup and configuration guide
```

---

## 🚀 Getting Started

### 1. Google Sheets Setup

Create a Google Sheet with two tabs:

**Tab: `Discounted`** — your recipient input list

| Column | Type | Notes |
|--------|------|-------|
| Name | Text | Full name |
| Email | Text | Valid email |
| DiscountPercent | Number | e.g. 20, 40, 60 |
| Opted_In | Text | Must be exactly `YES` |
| Status | Text | Leave blank for new recipients |

**Tab: `Coupons`** — auto-filled by the workflow

| Column | Notes |
|--------|-------|
| CouponCode | Generated automatically |
| UserEmail | |
| UserName | |
| DiscountPercent | |
| Status | `ACTIVE` → `EXPIRED` |
| CreatedAt | |
| ExpiresAt | |
| CouponType | |
| UsageCount | |
| MaxUsage | |

### 2. n8n Credentials

In n8n → Credentials, create:
- **Google Sheets OAuth2**
- **Gmail OAuth2**

### 3. Import Workflows

- Import `workflow.json` as the main workflow
- Import `email-digest-subflow.json` as a separate workflow
- In the last node of the main workflow, link it to the digest subflow

### 4. Configure Settings

In the **Coupon Config** node, update:

| Field | Production | Test |
|-------|-----------|------|
| `validityHours` | `24` | `0.0167` (~1 min) |
| `reminderHours` | `2` | `0.008` (~30 sec) |
| `notificationEmail` | your admin email | |
| `storeUrl` | your store URL | |

### 5. Run

- **Manual:** Click Execute Workflow on the canvas
- **Scheduled:** Activate the workflow — runs daily at 9AM automatically

---

## 📬 Emails Sent Per Run

| Email | To | When |
|-------|----|------|
| 🎁 Coupon | User | Immediately on trigger |
| ⏰ Reminder | User | Before expiry |
| 😔 Expired | User | At expiry |
| 📊 Daily Digest | Admin | At expiry, one email per batch |

---

## 🔧 Coupon Code Patterns

Each run randomly picks one of 6 formats:
- `SAVE` + name initials + timestamp fragment
- Email prefix + `OFF` + discount %
- `VIP` + random chars + discount %
- `FLASH` + random chars
- Name + timestamp
- `DEAL` + discount % + random chars

---

Built by Aayush
