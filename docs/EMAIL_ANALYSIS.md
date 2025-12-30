# 📧 Email Analysis - Order Patterns

Based on real email samples from: Zalando, Massimo Dutti, GOREWEAR, ARYS

---

## Order Summaries

### 1. ZALANDO Order (Split into 2 packages!)
| Email | From | Key Data |
|-------|------|----------|
| Thanks for your order | Zalando | Order confirmed, NO order ID |
| Your parcel will arrive 30/12-31/12 | Zalando | Tracking: `H1024562675499501017` (Hermes) |
| Deine Hermes Sendung von Zalando ist auf dem Weg | Hermes | Tracking: `H1024562675499501017`, mentions "Zalando" |
| Deine Sendung kommt heute | Hermes | Tracking: `H1024562675499501017` |
| Your parcel will arrive 03/01-07/01 | Zalando | **SECOND PACKAGE!** Tracking: `JD000390015819013542` |

**Challenge:** No order ID - must link by tracking number AND store name mention in carrier email

---

### 2. MASSIMO DUTTI Order
| Email | From | Key Data |
|-------|------|----------|
| Confirmation of order No. 30101502670 | Massimo Dutti | Order ID: `30101502670` |
| eReceipt attached – Tracking of order No. 30101502670 | Massimo Dutti | Order ID + PDF receipt |
| Your order N. 30101502670 is on its way | Massimo Dutti | Order ID + Tracking: `YVMH7TNK` (GLS) |
| 📦 Dein Paket wird in wenigen Tagen zugestellt | GLS | Tracking: `33512352232900100030-30101502670` |
| 📦 Dein GLS Paket kommt heute! | GLS | Same tracking, mentions "MD-Plewiska" |
| Your payment information for Massimo Dutti | **Klarna** | Order ref: `30101502670` - **IGNORE** |

**Linking:** Order ID `30101502670` appears in all shop emails; GLS tracking includes order ID in extended format!

---

### 3. GOREWEAR Order
| Email | From | Key Data |
|-------|------|----------|
| Vielen Dank für deine Bestellung! | GOREWEAR | Order ID: `517746` |
| GOREWEAR Rechnung | GOREWEAR | Order ID: `517746` |
| GOREWEAR Bestellung versandt | GOREWEAR | Order ID: `517746`, Tracking: `00340434665233630945` |
| Ihre BARTH+CO... Sendung ist unterwegs | DHL | Tracking: `00340434665233630945` |
| Your payment information for Gorewear | **Klarna** | **IGNORE** |

**Linking:** Order ID + tracking number both available

---

### 4. ARYS Order
| Email | From | Key Data |
|-------|------|----------|
| Thank you for your order 🤍 | ARYS | Order confirmed |
| Order #5850 confirmed | ARYS (Shopify) | Order ID: `5850` |
| Deine Bestellung #5850 ist versandbereit | ARYS | Order ID: `5850`, Tracking: `00340434327779634800` |
| Your payment information for ARYS Store | **Klarna** | **IGNORE** |

**Linking:** Order ID in subject lines, tracking in shipping email

---

## Identifier Patterns

### Order IDs
| Store | Format | Example |
|-------|--------|---------|
| Zalando | **NONE** | - |
| Massimo Dutti | Numeric, 11 digits | `30101502670` |
| GOREWEAR | Numeric, 6 digits | `517746` |
| ARYS | Numeric, 4 digits | `5850` |

### Tracking Numbers
| Carrier | Format | Example |
|---------|--------|---------|
| Hermes | `H` + 19 digits | `H1024562675499501017` |
| DHL | 20 digits | `00340434665233630945` |
| GLS | Short: 8 chars / Long: includes order | `YVMH7TNK` / `33512352232900100030-30101502670` |
| Unknown (Zalando 2nd) | `JD` + 18 digits | `JD000390015819013542` |

---

## Stitching Rules

### Level 1: Exact Match by Order ID
```
IF email contains order_id
AND existing_order has same order_id + store
THEN → LINK
```

### Level 2: Exact Match by Tracking Number
```
IF email contains tracking_number
AND existing_order has same tracking_number
THEN → LINK
```

### Level 3: Carrier Mentions Store
```
IF carrier_email mentions store_name (e.g., "von Zalando", "from GOREWEAR")
AND existing_order from that store has status=shipped AND no tracking yet
THEN → LINK (and add tracking number)
```

### Level 4: Timing + Store Match
```
IF email is from store
AND existing_order from same store exists within ±3 days
AND no other orders from same store in that window
THEN → LIKELY LINK (ask Claude to verify)
```

---

## Special Cases

### Split Shipments (Zalando)
- Single order can become multiple packages
- Each package gets its own tracking number
- May use different carriers!
- **Solution:** Create child records or track multiple tracking numbers per order

### GLS Extended Tracking
- GLS includes order ID in tracking: `33512352232900100030-30101502670`
- Can extract order ID from tracking number!

### Carrier Sender Names
| Carrier | From Address | From Name |
|---------|--------------|-----------|
| Hermes | noreply@paketankuendigung.myhermes.de | Hermes Paketankündigung |
| DHL | noreply@dhl.de | DHL Paket |
| GLS | no-reply@gls-pakete.de | GLS Paket |

---

## Klarna Handling

**Rule: IGNORE Klarna emails for delivery tracking**

- Subject pattern: `Your payment information for {STORE}`
- Contains `Merchant order reference` with order ID
- Useful for: payment status (separate workflow)
- Not useful for: delivery tracking

---

## Claude Extraction Prompt (Updated)

```
Analyze this email for delivery tracking.

CLASSIFICATION:
- is_delivery_related: true/false
- ignore_reason: null | "payment_service" | "food_delivery" | "digital" | etc.

IDENTIFIERS (extract ALL found):
- order_ids: ["517746", "#5850"] - include # if present
- tracking_numbers: ["H1024562675499501017", "00340434665233630945"]
- tracking_urls: [full URLs]

SOURCE:
- email_type: order_confirmation | shipping | tracking_update | out_for_delivery | delivered
- sender_type: shop | carrier | payment_service
- store_name: extracted or mentioned store
- carrier_name: DHL | Hermes | GLS | DPD | UPS | etc.

STORE MENTION IN CARRIER EMAIL:
- mentioned_store: If carrier email mentions a store (e.g., "von Zalando"), extract it

DETAILS:
- items: what was ordered
- amount: order total
- expected_delivery: YYYY-MM-DD
- delivery_window: "between 16:10 and 18:10" if available
```

---

## Matching Algorithm

```python
def find_matching_order(new_email, existing_orders):
    # Level 1: Order ID match
    for order_id in new_email.order_ids:
        match = find_by_order_id(order_id, new_email.store)
        if match:
            return match, confidence="high"
    
    # Level 2: Tracking number match
    for tracking in new_email.tracking_numbers:
        match = find_by_tracking(tracking)
        if match:
            return match, confidence="high"
    
    # Level 3: Carrier mentions store
    if new_email.sender_type == "carrier" and new_email.mentioned_store:
        matches = find_by_store_and_status(
            store=new_email.mentioned_store,
            status="shipped",
            missing_tracking=True
        )
        if len(matches) == 1:
            return matches[0], confidence="medium"
    
    # Level 4: Fuzzy match
    if new_email.store:
        matches = find_by_store_and_date(
            store=new_email.store,
            date_range=±3_days
        )
        if len(matches) == 1:
            return matches[0], confidence="low"
    
    # No match - create new order
    return None, confidence=None
```

