# package-integrated-carrier-ups-accelerator

**Version:** 0.0.2
**Spec Version:** 2.0.0

---

## Overview

This package adds UPS carrier support to the Integrated Carrier suite. It seeds UPS-specific reference data (carrier record, service levels, billing types), installs UPS-specific data flows for rate quoting and label generation via the UPS Developer API, and registers those flows with the `Integrated Carrier Router` so they are automatically dispatched when UPS is selected as the carrier.

---

## Package Contents

```
integrated-carrier-ups/
├── manifest.json
├── package-data.json
├── install/                     10 install steps
├── preinstall/                  6 preinstall verification steps
└── postinstall/                 2 postinstall registration steps
```

---

## Seed Data Installed

### IntegratedCarrier
| ID | Name |
|----|------|
| `ups` | UPS |

### IntegratedCarrierService (UPS service levels)
| Service |
|---------|
| UPS Ground |
| UPS 2nd Day Air |
| UPS Next Day Air |
| UPS Next Day Air Saver |
| UPS 3 Day Select |
| UPS Worldwide Expedited |
| Additional UPS services (added by install steps) |

### IntegratedCarrierBillingType (UPS billing options)
| Type |
|------|
| Shipper (bill to sender account) |
| Receiver (bill to recipient account) |
| Third Party (bill to a third-party account) |

---

## Installed Flows

### UPS Rate Quote (`Integration`)

Submits a shipment request to the UPS Rating API and returns available service rates:
- Formats the `IntegratedCarrierRequest` payload into UPS Rate API request structure (JSON)
- Authenticates via UPS OAuth 2.0 Client Credentials using the configured `IntegratedCarrierAccount`
- Calls UPS REST Rates and Service endpoint
- Parses response and returns normalized rate options with transit times, service levels, and pricing
- Handles UPS API error codes and warning messages

### UPS Label Generation (`Integration`)

Generates a UPS shipping label and tracking number:
- Authenticates via UPS OAuth 2.0
- Submits shipment to UPS Ship endpoint with package details, service level, and billing type
- Returns label image (GIF for thermal via ZPL conversion, or PDF), tracking number, and confirmation
- Stores response and tracking number in `IntegratedCarrierRequest`
- Triggers `Integrated Carrier Print Labels Standard` flow for label printing

### UPS Void Shipment (`Integration`)

Voids a previously generated UPS label:
- Calls UPS Void Shipment endpoint with the tracking number
- Updates `IntegratedCarrierRequest` status to voided
- Returns void confirmation status

### Supporting flows (7 additional)

OAuth token management (UPS token caching and refresh), End-of-Day pickup request, tracking status query, UPS Access Point lookup, and international shipment handling.

---

## Install Process

**Preinstall (6 steps):** Verifies uniqueness of the `IntegratedCarrier` (UPS), `IntegratedCarrierBillingType` records, `IntegratedCarrierService` records, and UPS flow IDs before installing.

**Install (10 steps):** Creates all seed data records and flow headers; creates and deploys flow versions.

**Postinstall (2 steps):**
1. Creates `IntegratedCarrierRequestFlow` records linking UPS + Rate Quote → UPS Rate Quote flow; UPS + Label Generation → UPS Label Generation flow; etc.
2. Verifies router can resolve UPS handler flows

---

## Installation

1. Install the core suite first: `package-integrated-carrier-core-schema-accelerator`, `package-integrated-carrier-core-flows-accelerator`, `package-integrated-carrier-core-screens-accelerator`
2. Import this package via Fuuz Package Manager
3. Create an `IntegratedCarrierAccount` record for UPS with your account number and API credentials
4. Create an `IntegratedCarrierConnectionConfiguration` record pointing to the UPS API endpoint
5. Test using the Integrated Carrier Shipment screen with UPS selected and a valid address

---

## Dependencies

- **`package-integrated-carrier-core-schema-accelerator`** — required
- **`package-integrated-carrier-core-flows-accelerator`** — required
- **UPS Developer Account** with OAuth 2.0 API credentials — [UPS Developer Portal](https://developer.ups.com)
- UPS Production or Customer Integration Environment (CIE) credentials

---

## Part of the Integrated Carrier Suite

| Package | Description |
|---------|-------------|
| `integrated-carrier-core-schema` | Data models |
| `integrated-carrier-core-flows` | Router and print flows |
| `integrated-carrier-core-screens` | Shipment management screens |
| `integrated-carrier-fedex` | FedEx seed data and label flows |
| **integrated-carrier-ups** (this) | UPS seed data and label flows |
| `integrated-carrier-addon-plex` | Plex ERP integration extension |

---

*Built on the [Fuuz Industrial Operations Platform](https://fuuz.com)*

## Service levels

No service level agreement applies to anything published here. It becomes a supported
deliverable only once it has been implemented by a Fuuz services professional or an
approved Fuuz partner.
