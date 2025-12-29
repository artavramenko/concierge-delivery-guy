# 🚚 Delivery Guy - Setup Guide

## Prerequisites

- n8n Cloud account (or self-hosted)
- Gmail accounts to monitor
- Google Sheets for storage
- Claude API key (already configured from Mailroom)

---

## Step 1: Create Google Sheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Create a new spreadsheet named **"Delivery Guy - Orders"**
3. Rename the first sheet to **"Orders"**
4. Add these headers in row 1:

```
order_key | order_id | owner | email_account | store | item_description | order_date | amount | currency | status | delivery_provider | tracking_number | tracking_url | store_order_url | expected_delivery | actual_delivery | last_updated | last_email_subject | last_email_date | notes
```

5. Copy the **Sheet ID** from the URL:
   ```
   https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID_HERE/edit
   ```

---

## Step 2: Configure Gmail OAuth in n8n

For each Gmail account you want to monitor:

1. Go to n8n → **Settings** → **Credentials**
2. Click **Add Credential**
3. Search for **"Gmail OAuth2 API"**
4. Click **Sign in with Google**
5. Select the Gmail account
6. Grant permissions for:
   - Read emails
   - View email messages
7. Name the credential clearly (e.g., "Gmail Artem", "Gmail Carlotta")

**Note:** Each Gmail account needs its own credential!

---

## Step 3: Import the Workflow

1. Go to n8n → **Workflows** → **Import from File**
2. Upload `delivery_guy_simple_v1.json` (start with this)
3. The workflow will appear as "🚚 Delivery Guy (Simple)"

---

## Step 4: Configure the Workflow

### 4.1 Update Owner Configuration

Open the **"Config"** node and update the `OWNER` variable:

```javascript
const OWNER = 'Artem'; // Change to your name
```

### 4.2 Update Gmail Node Credentials

For **"Gmail: Get Emails"** node:
- Click the node
- Under **Credential to connect with**, select your Gmail credential

### 4.3 Update Google Sheets Nodes

For **"Sheets: Lookup"**, **"Sheets: Create"**, and **"Sheets: Update"**:

1. Click each node
2. Update **Document ID** → paste your Google Sheet ID
3. Ensure **Sheet Name** is "Orders"
4. Select your Google Sheets credential

### 4.4 Claude Credentials

The **"Analyze with Claude"** node should use your existing Claude API credential (same as Mailroom/Postman).

---

## Step 5: Test the Workflow

### Manual Test

1. Click **Execute Workflow** (play button)
2. Watch the execution flow
3. Check your Google Sheet for new entries

### Test with Sample Email

Send yourself a test order confirmation email:
- Subject: "Your Amazon Order #123-456-789 has shipped"
- Body: Include tracking number, delivery date, store order URL, etc.

Wait a few minutes, then run the workflow manually.

---

## Step 6: Activate Scheduled Runs

1. Click the **Schedule (Every 3h)** node
2. Verify the interval (3 hours)
3. Toggle the workflow to **Active** (top right switch)

---

## Column Reference

| Column | Description |
|--------|-------------|
| `order_key` | Auto-generated unique key for deduplication |
| `order_id` | Order number from store (if found) |
| `owner` | Who placed the order |
| `email_account` | Which email received the notification |
| `store` | Store name (Amazon, Zalando, etc.) |
| `item_description` | Brief description of items |
| `order_date` | When order was placed |
| `amount` | Order total |
| `currency` | EUR, USD, etc. |
| `status` | Current status (ordered/shipped/delivered) |
| `delivery_provider` | DHL, DPD, Hermes, etc. |
| `tracking_number` | Tracking number |
| `tracking_url` | Link to tracking page |
| `store_order_url` | Link to order on store's website |
| `expected_delivery` | Expected delivery date |
| `actual_delivery` | Actual delivery date |
| `last_updated` | When record was last updated |
| `last_email_subject` | Subject of last related email |
| `last_email_date` | Date of last related email |
| `notes` | Any additional notes |

---

## Google Sheet Formatting Tips

### Conditional Formatting

Add these rules to make the sheet more visual:

1. **Status column colors:**
   - `ordered` → Yellow background
   - `shipped` → Blue background
   - `out_for_delivery` → Orange background
   - `delivered` → Green background
   - `returned` → Red background

2. **Expected delivery:**
   - Past date + not delivered → Red text
   - Today → Bold orange
   - Tomorrow → Orange

### Filters & Views

Create saved filter views:
- "Active Orders" - status not "delivered" and not "returned"
- "Artem's Orders" - owner = "Artem"
- "Carlotta's Orders" - owner = "Carlotta"

---

## Troubleshooting

### Gmail returns no emails
- Check the Gmail filter query in the node
- Ensure the credential has proper permissions
- Check if emails are in INBOX (not in spam/promotions)

### Duplicate orders created
- Check the `order_key` logic in "Parse Response"
- Some stores may have inconsistent order IDs

### Claude doesn't extract order info correctly
- Review the email content in the "Filter: Order Emails" output
- Some emails may be heavily styled HTML - try adjusting text extraction

---

## Phase 2: Slack Integration (Future)

The roadmap includes:
- `#deliveries` Slack channel for notifications
- Real-time alerts for "out for delivery"
- Interactive buttons for marking returns
- Daily digest of expected deliveries

This will be added as a separate workflow that reads from the Google Sheet.

