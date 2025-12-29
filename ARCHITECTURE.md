# 🚚 Delivery Guy - Architecture

## Overview

Automated tracking of online orders and deliveries by monitoring Gmail inboxes.

## Data Flow

```
┌─────────────────┐
│ Schedule (3h)   │
└────────┬────────┘
         ↓
┌─────────────────┐
│ Gmail Accounts  │ ← Multiple accounts (Artem, Carlotta)
└────────┬────────┘
         ↓
┌─────────────────┐
│ Fetch Emails    │ ← Recent emails (last 4h or unread)
└────────┬────────┘
         ↓
┌─────────────────┐
│ Filter Keywords │ ← Order, delivery, tracking, shipped, etc.
└────────┬────────┘
         ↓
┌─────────────────┐
│ Claude Analysis │ ← Extract order details from email
└────────┬────────┘
         ↓
┌─────────────────┐
│ Check Sheet     │ ← Does this order exist?
└────────┬────────┘
         ↓
    ┌────┴────┐
    ↓         ↓
┌───────┐ ┌────────┐
│ New   │ │ Update │
│ Row   │ │ Row    │
└───────┘ └────────┘
         ↓
┌─────────────────┐
│ Slack Notify    │ ← Optional: new orders, status changes
└─────────────────┘
```

## Google Sheet Structure

**Sheet Name:** `Orders`

| Column | Type | Description |
|--------|------|-------------|
| `order_key` | String | Unique identifier (order number or generated) |
| `owner` | String | Artem / Carlotta |
| `email_account` | String | Which Gmail account |
| `store` | String | Amazon, Zalando, ASOS, etc. |
| `item_description` | String | What was ordered (brief) |
| `order_id` | String | Order/reference number |
| `order_date` | Date | When order was placed |
| `amount` | Number | Total amount |
| `currency` | String | EUR, USD, etc. |
| `status` | String | ordered / confirmed / shipped / out_for_delivery / delivered / returned |
| `delivery_provider` | String | DHL, DPD, Hermes, UPS, GLS, Amazon Logistics |
| `tracking_number` | String | Tracking number |
| `tracking_url` | String | Link to tracking page |
| `store_order_url` | String | Link to order status on store's website |
| `expected_delivery` | Date | Expected delivery date |
| `actual_delivery` | Date | Actual delivery date (when delivered) |
| `last_updated` | DateTime | Timestamp of last update |
| `last_email_subject` | String | Subject of last related email |
| `last_email_date` | DateTime | Date of last related email |
| `notes` | String | Additional notes |

## Email Filtering Keywords

**Subject/Body indicators for order-related emails:**

```
Order confirmation, Bestellbestätigung, Order #, Bestellung #
Shipped, Versandt, Your order is on its way, Dein Paket ist unterwegs
Delivery, Lieferung, Out for delivery, Zustellung
Tracking, Sendungsverfolgung, Track your order
Delivered, Zugestellt, Your package has arrived
Return, Rückgabe, Refund, Erstattung
```

## Status Lifecycle

```
ordered → confirmed → shipped → out_for_delivery → delivered
                                                  ↘ returned
```

## Claude Analysis Prompt

For each email, Claude extracts:

```json
{
  "is_order_related": true/false,
  "email_type": "order_confirmation|shipping_notification|delivery_update|delivered|return|other",
  "order_id": "extracted order/reference number or null",
  "store": "store name",
  "item_description": "brief description of items (max 100 chars)",
  "amount": 123.45,
  "currency": "EUR",
  "status": "ordered|confirmed|shipped|out_for_delivery|delivered|returned",
  "delivery_provider": "DHL|DPD|Hermes|UPS|GLS|FedEx|Amazon Logistics|other|null",
  "tracking_number": "tracking number or null",
  "tracking_url": "full tracking URL or null",
  "store_order_url": "URL to order status page on store's website or null",
  "expected_delivery": "YYYY-MM-DD or null",
  "notes": "any important details"
}
```

## Gmail Accounts Configuration

```json
{
  "accounts": [
    { "owner": "Artem", "email": "artem@example.com" },
    { "owner": "Artem", "email": "artem.work@example.com" },
    { "owner": "Carlotta", "email": "carlotta@example.com" }
  ]
}
```

## Slack Integration (Phase 2)

**Channel:** `#deliveries`

**Notifications:**
- New order detected
- Package shipped
- Out for delivery (same day alert)
- Delivered

**Interactive features:**
- Mark as returned (creates to-do)
- Add note to order
- Manually update status

## Deduplication Strategy

1. **Primary key:** `order_id + store` combination
2. **Fallback:** If no order_id, use `store + amount + order_date` hash
3. **Email threading:** Track `last_email_date` to avoid reprocessing

## Schedule Considerations

- **Frequency:** Every 3 hours
- **Email lookback:** Fetch emails from last 4 hours (1h overlap to catch delays)
- **Unread filter:** Optionally only process unread emails, mark as read after processing

