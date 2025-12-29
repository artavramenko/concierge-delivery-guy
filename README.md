# 🚚 Delivery Guy

Automated parcel and order tracking by monitoring Gmail inboxes.

## Features

- 📧 Monitors Gmail for order confirmations and shipping notifications
- 🤖 Uses Claude AI to extract order details from emails
- 📊 Stores order data in Google Sheets for easy tracking
- 🔄 Automatically updates order status as new emails arrive
- 🇩🇪 🇬🇧 Supports both German and English emails

## Quick Start

1. Create Google Sheet with headers (see `SETUP_GUIDE.md`)
2. Set up Gmail OAuth in n8n
3. Import workflow from `n8n_workflows/`
4. Configure credentials and sheet ID
5. Activate!

## Documentation

- [Architecture](ARCHITECTURE.md) - System design & data flow
- [Setup Guide](SETUP_GUIDE.md) - Step-by-step configuration

## Workflows

| File | Description |
|------|-------------|
| `delivery_guy_simple_v1.json` | Single Gmail account (recommended to start) |
| `delivery_guy_v1.json` | Multi-account support (advanced) |

## Part of Concierge Suite

This is a module of the [Concierge](../) home automation suite.

