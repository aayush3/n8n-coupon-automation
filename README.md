# 🎁 n8n Automated Coupon Campaign

A fully automated coupon distribution system built with **n8n**, **Google Sheets**, and **Gmail**.

---

## 📌 What It Does

This workflow automates the entire lifecycle of a personalized coupon campaign:

1. **Reads** a recipient list from Google Sheets
2. **Filters** out anyone already active or not opted in
3. **Generates** a unique personalized coupon code per recipient
4. **Logs** the coupon to a tracking sheet (status, timestamps, usage limits)
5. **Sends** a branded HTML coupon email to the recipient
6. **Waits**, then sends an **expiry reminder** email
7. **Marks** the coupon as `EXPIRED` in Google Sheets automatically
8. **Notifies** the user their coupon has expired
9. **Calls a subflow** that sends the admin a daily digest email

---

## 🛠 Tools Used

| Tool | Purpose |
|------|---------|
| n8n | Automation engine |
| Google Sheets | Recipient management & coupon log |
| Gmail | Email delivery (coupon, reminder, expiry, digest) |

---

## 📁 Files in This Repo

```
n8n-coupon-automation/
├── workflow.json              → Main coupon campaign workflow (import into n8n)
├── email-digest-subflow.json  → Admin digest subflow (import separately)
├── README.md                  → This file
└── docs/
    └── workshop-guide.md      → Full step-by-step build guide
```

---

## 🚀 How to Use

### Step 1 — Set Up Google Sheets

Create a Google Sheet with **two tabs**:

**Tab 1: `Discounted`** (you manage this manually)

| Column | Type | Notes |
|--------|------|-------|
| Name | Text | Full name |
| Email | Text | Valid email |
| DiscountPercent | Number | e.g. 20, 40, 60 |
| Opted_In | Text | Must be exactly `YES` |
| Status | Text | Leave blank for new recipients |

**Tab 2: `Coupons`** (auto-filled by the workflow)

| Column | Auto-filled value |
|--------|------------------|
| CouponCode | e.g. `VIPZK7J60` |
| UserEmail | Recipient's email |
| UserName | Recipient's name |
| DiscountPercent | From Discounted tab |
| Status | `ACTIVE` → `EXPIRED` |
| CreatedAt | Timestamp |
| ExpiresAt | Timestamp |
| CouponType | `PERSONALIZED` |
| UsageCount | `0` |
| MaxUsage | `1` |

---

### Step 2 — Set Up Credentials in n8n

In your n8n instance, go to **Credentials** and create:
- **Google Sheets OAuth2** — for reading/writing the sheet
- **Gmail OAuth2** — for sending all emails

---

### Step 3 — Import the Workflows

1. In n8n, click the menu icon (top right of canvas)
2. Select **Import from file**
3. Import `workflow.json` first (main workflow)
4. Import `email-digest-subflow.json` as a **separate workflow**
5. In the main workflow's last node ("Call Email Digest Subflow"), select the digest workflow from the dropdown

---

### Step 4 — Configure the Coupon Config Node

Open the **Coupon Config** node and update:

| Field | Value |
|-------|-------|
| `validityHours` | `24` (production) / `0.0167` (test ~1 min) |
| `reminderHours` | `2` (production) / `0.008` (test ~30 sec) |
| `notificationEmail` | Your admin email |
| `storeUrl` | Your store URL |

> ⚠️ **Testing tip:** Use `0.0167` and `0.008` to test the full flow in ~1 minute. Reset to `24` and `2` before going live.

---

### Step 5 — Run It

- **Manual:** Click **Execute Workflow** on the canvas
- **Scheduled:** Activate the workflow — it runs every day at 9AM automatically

---

## 🧪 Testing Checklist

| Check | Expected Result |
|-------|----------------|
| All nodes green | No red errors on canvas |
| Coupon email received | Branded email with unique code & discount % |
| Reminder email received | Orange-themed urgency email |
| Expiry email received | Dark-themed "your coupon expired" message |
| Coupons tab updated | All 10 columns filled, Status = `EXPIRED` after run |
| Admin digest received | One email showing all recipients in a table |

---

## ⚠️ Common Issues

**Filter passes nobody**
- Check `Opted_In` is exactly `YES` (uppercase, no spaces)
- `Status` must be blank in the Discounted tab

**Only first recipient processed**
- Open **Generate Coupon Code** node → Settings tab → set Mode to **Run Once for Each Item**

**Coupons tab not filling**
- Column headers in the sheet must exactly match: `CouponCode`, `UserEmail`, `UserName`, `DiscountPercent`, `Status`, `CreatedAt`, `ExpiresAt`, `CouponType`, `UsageCount`, `MaxUsage`

**Admin digest shows duplicate rows**
- Open the **Read Coupons Sheet** node inside the subflow → Settings tab → enable **Execute Once**

---

## ⚙️ Node Flow (Main Workflow)

```
Schedule / Manual Trigger
  → Read Recipients (Google Sheets)
    → Filter Eligible Recipients
      → Coupon Config (Set)
        → Generate Coupon Code (Code)
          → Prepare Coupon Data (Set)
            → Log to Google Sheets
              → Send Coupon Email (Gmail)
                → Wait Until Reminder
                  → Send Reminder Email (Gmail)
                    → Wait Until Expiry
                      → Expire in Google Sheets
                        → Notify User (Gmail)
                          → Call Email Digest Subflow
```

---

## 📄 License

MIT — free to use and modify.
