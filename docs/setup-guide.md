# Setup Guide — Automated Coupon Campaign

A complete guide to setting up and running the automated coupon campaign workflow.

---

## Prerequisites

- n8n instance (cloud or self-hosted)
- Google account with Sheets and Gmail access

---

## Google Sheet Structure

Create one Google Sheet with two tabs.

### Tab 1: `Discounted`

Managed manually. The workflow reads from this tab but never writes to it.

| Column | Type | Example | Notes |
|--------|------|---------|-------|
| Name | Text | Akhil Singh | Full name |
| Email | Text | akhil@example.com | Valid email |
| DiscountPercent | Number | 60 | Defaults to 20 if blank |
| Opted_In | Text | YES | Must be exactly `YES` — case sensitive |
| Status | Text | (blank) | Leave empty for new recipients |

### Tab 2: `Coupons`

Auto-filled by the workflow. Do not edit manually during a run.

| Column | Value |
|--------|-------|
| CouponCode | Generated (e.g. `VIPZK7J60`) |
| UserEmail | Recipient's email |
| UserName | Recipient's name |
| DiscountPercent | From Discounted tab |
| Status | `ACTIVE` on creation → `EXPIRED` after wait |
| CreatedAt | Timestamp |
| ExpiresAt | Timestamp |
| CouponType | `PERSONALIZED` |
| UsageCount | `0` |
| MaxUsage | `1` |

---

## n8n Credentials

In n8n → Credentials, create:
- **Google Sheets OAuth2** — for reading/writing the sheet
- **Gmail OAuth2** — for sending all emails

Both can use the same Google account.

---

## Workflow Overview

### Main Workflow (`workflow.json`)

```
Schedule / Manual Trigger
  → Read Recipients (Google Sheets)
    → Filter Eligible Recipients
      → Coupon Config (Set node)
        → Generate Coupon Code (Code node)
          → Prepare Coupon Data (Set node)
            → Log to Google Sheets
              → Send Coupon Email (Gmail)
                → Wait Until Reminder Time
                  → Send Reminder Email (Gmail)
                    → Wait Until Expiry
                      → Expire in Google Sheets
                        → Notify User of Expiry (Gmail)
                          → Call Email Digest Subflow
```

### Subflow (`email-digest-subflow.json`)

```
Execute Workflow Trigger
  → Read Coupons Sheet
    → Build Daily Summary (Code node)
      → Send Admin Digest Email (Gmail)
```

---

## Configuration Reference

All campaign settings are in the **Coupon Config** node:

| Field | Production | Test | Description |
|-------|-----------|------|-------------|
| `validityHours` | `24` | `0.0167` | How long the coupon is valid |
| `reminderHours` | `2` | `0.008` | How far before expiry to send reminder |
| `notificationEmail` | your@email.com | your@email.com | Admin digest destination |
| `storeUrl` | https://yourstore.com | https://yourstore.com | Appended as `?discount=CODE` |
| `discountPercent` | From sheet | From sheet | Per-recipient, defaults to 20 |

> ⚠️ Always reset `validityHours` to `24` and `reminderHours` to `2` before running in production.

---

## Key Node Settings

**Generate Coupon Code node**
- Go to Settings tab → set Mode to **Run Once for Each Item**
- Without this, only the first recipient gets a coupon

**Read Coupons Sheet (inside subflow)**
- Go to Settings tab → enable **Execute Once**
- Without this, the digest email shows duplicate rows

**Expire Coupon node**
- The Status field must be the hardcoded string `EXPIRED` — not an expression
- Match column must be set to `CouponCode`

---

## Testing

1. Set `validityHours` to `0.0167` and `reminderHours` to `0.008`
2. Add yourself as a test recipient (Opted_In = `YES`, Status blank)
3. Click Manual Trigger → Execute Workflow
4. Within ~1 minute the full flow completes:
   - Coupon email → Reminder email → Status = EXPIRED → Expiry email → Admin digest

### What Success Looks Like

| Check | Expected |
|-------|---------|
| All nodes green | No errors on canvas |
| Coupon email | Branded email with unique code |
| Reminder email | Orange-themed urgency email |
| Expiry email | Dark-themed expired message |
| Coupons tab | All 10 columns filled, Status = EXPIRED |
| Admin digest | One email with a table of all recipients |

---

## Troubleshooting

**Filter passes nobody**
- `Opted_In` must be exactly `YES` (uppercase, no trailing spaces)
- `Status` must be blank in the Discounted tab

**Only first recipient processed**
- Generate Coupon Code node → Settings → Mode must be **Run Once for Each Item**

**Coupons tab not filling**
- Column headers must match exactly: `CouponCode`, `UserEmail`, `UserName`, `DiscountPercent`, `Status`, `CreatedAt`, `ExpiresAt`, `CouponType`, `UsageCount`, `MaxUsage`

**Expire node not updating**
- Status value must be the plain string `EXPIRED`, not `=EXPIRED` or a variable
- Match column must be `CouponCode`

**Duplicate rows in admin digest**
- Read Coupons Sheet node (inside subflow) → Settings → enable **Execute Once**

---

## Going Live

1. Restore `validityHours` to `24`, `reminderHours` to `2`
2. Fill the Discounted tab with real recipients
3. Confirm every row has `Opted_In = YES` and blank Status
4. Activate the workflow for the daily 9AM schedule, or trigger manually

> After a run: set recipient Status to `ACTIVE` in the Discounted tab to prevent duplicate sends on the next daily run.
