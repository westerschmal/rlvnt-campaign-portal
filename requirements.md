# RLVNT Campaign Portal - Requirements Specification

> **Version**: 1.1  
> **Last Updated**: January 6, 2026  
> **Status**: Planning Phase

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Platform Overview](#2-platform-overview)
3. [User Management](#3-user-management)
4. [Order & Flight Management](#4-order--flight-management)
5. [Pricing System](#5-pricing-system)
6. [Campaign Workflow](#6-campaign-workflow)
7. [Creative Assets](#7-creative-assets)
8. [Notifications](#8-notifications)
9. [Channel & Product Matrix](#9-channel--product-matrix)
10. [Data Model](#10-data-model)
11. [Technical Considerations](#11-technical-considerations)
12. [Scope & Roadmap](#12-scope--roadmap)

---

## 1. Executive Summary

### 1.1 Purpose

This document outlines the requirements for expanding the RLVNT Campaign Portal - a tool that enables advertisers and media agencies to:

- Upload and validate creative assets (partly existing functionality)
- **Request media campaigns** across multiple channels (new functionality)
- **Track campaign status** from request to completion (new functionality)

For RLVNT internal staff, the platform provides:

- A dashboard to review and approve campaign requests
- Tools to manage pricing and customer organizations
- Campaign status management and booking confirmation

### 1.2 Target Users

| User Type | Organization | Primary Use Cases |
|-----------|--------------|-------------------|
| Campaign Planner | Advertiser / Agency | Create orders, upload assets, track status |
| Media Buyer | Advertiser / Agency | Review pricing, manage budgets |
| Campaign Manager | RLVNT | Review orders, approve campaigns |
| Admin | RLVNT | Manage pricing, organizations, users |

### 1.3 Business Context

RLVNT ([rlvnt.world](https://rlvnt.world)) is a full-service Media Marketplace solution that combines technology, AI, and data-driven insights for digital advertising. Their product offering spans the consumer lifecycle:

- **RLVNT Catch** - Awareness stage (reach new audiences)
- **RLVNT Connect** - Engagement stage (foster connections)
- **RLVNT Convert** - Conversion stage (drive actions)

---

## 2. Platform Overview

### 2.1 Core Concepts

| Term | Definition |
|------|------------|
| **Organization** | A company (advertiser or agency) using the platform |
| **Order** | A campaign request containing one or more flights |
| **Flight** | A specific media buy with defined channel, format, targeting, dates, and budget |
| **Creative Asset** | Media file (video, image) uploaded for use in campaigns |
| **Unit** | The billing metric (impressions, completed views, clicks, etc.) |

### 2.2 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          RLVNT Campaign Portal                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────┐         ┌─────────────────────────────┐   │
│  │    External Portal      │         │    Internal Dashboard       │   │
│  │    (Advertisers/        │         │    (RLVNT Staff)            │   │
│  │     Agencies)           │         │                             │   │
│  │                         │         │  • View all orders          │   │
│  │  • Create orders        │         │  • Review & approve         │   │
│  │  • Add flights          │  ────►  │  • Manage status            │   │
│  │  • Upload assets        │  ◄────  │  • Manage pricing           │   │
│  │  • Track status         │         │  • Manage organizations     │   │
│  │  • Switch organizations │         │                             │   │
│  └─────────────────────────┘         └─────────────────────────────┘   │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │                       Shared Services                             │  │
│  │                                                                   │  │
│  │  • Asset validation & hosting (existing)                         │  │
│  │  • Pricing engine                                                 │  │
│  │  • User authentication & authorization                           │  │
│  │  • Email notifications                                           │  │
│  │  • Organization management                                       │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. User Management

### 3.1 User Types

| Type | Description | Access Level |
|------|-------------|--------------|
| **External User** | Works at advertiser or agency | Organization-scoped access |
| **Internal User** | Works at RLVNT | Full admin access (for MVP) |

### 3.2 Multi-Organization Support

- Users can belong to **multiple organizations**
- Users can **switch between organizations** via UI selector
- Each organization has isolated:
  - Orders and flights
  - Creative assets
  - Custom pricing (if configured)
  - User membership

### 3.3 Roles & Permissions (Future Enhancement)

For MVP, all internal users have admin access. Future roles may include:

| Role | Permissions |
|------|-------------|
| Viewer | Read-only access to orders |
| Editor | Create and edit orders |
| Approver | Approve/reject orders |
| Admin | Full access including pricing and user management |

---

## 4. Order & Flight Management

### 4.1 Order Structure

An **Order** represents a campaign request with the following properties:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | String | ✅ | Campaign/order name |
| `advertiser` | Reference | ✅ | The advertiser (may differ from organization) |
| `organization_id` | Reference | ✅ | Owning organization |
| `start_date` | Date | ✅ | Campaign start date |
| `end_date` | Date | ✅ | Campaign end date |
| `total_budget` | Decimal | ❌ | Total budget for the order |
| `po_number` | String | ❌ | Customer's purchase order number |
| `status` | Enum | ✅ | Current order status |
| `created_by` | Reference | ✅ | User who created the order |

### 4.2 Flight Structure

A **Flight** is a specific media buy within an order:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `order_id` | Reference | ✅ | Parent order |
| `channel` | Enum | ✅ | Video, Display, Audio, etc. |
| `product` | String | ✅ | Instream, Outstream, Display, etc. |
| `network` | String | ✅ | RLVNT, Broadcaster, CTV, RON, etc. |
| `format` | String | ✅ | Duration (5s, 30s) or size (APTO, Big IAB) |
| `targeting_enabled` | Boolean | ✅ | Whether targeting is applied |
| `targeting_config` | JSON | ❌ | Targeting options (geo, demographics, device) |
| `start_date` | Date | ✅ | Flight start (defaults to order start) |
| `end_date` | Date | ✅ | Flight end (defaults to order end) |
| `unit_type` | String | ✅ | Impressions, Completed views, etc. |
| `unit_price` | Decimal | ✅ | Price per unit (calculated) |
| `units` | Integer | ✅ | Number of units ordered |
| `budget` | Decimal | ✅ | Total flight budget |

### 4.3 Order Creation Flow

```
┌──────────────────┐
│  1. Order Info   │
│  ─────────────── │
│  • Name          │
│  • Advertiser    │
│  • Start/End     │
│  • Total Budget  │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  2. Add Flight   │◄─────────────────┐
│  ─────────────── │                  │
│  • Channel       │                  │
│  • Product       │                  │
│  • Network       │                  │
│  • Format        │                  │
│  • Targeting     │                  │
│  • Dates         │                  │
└────────┬─────────┘                  │
         │                            │
         ▼                            │
┌──────────────────┐                  │
│  3. Set Budget   │                  │
│  ─────────────── │                  │
│  Unit Price: €X  │                  │
│  ─────────────── │                  │
│  Enter units OR  │     ┌────────────┴────────────┐
│  Enter budget    │     │  Add another flight?    │
└────────┬─────────┘     │  [Yes] [No, Submit]     │
         │               └─────────────────────────┘
         │
         ▼
┌──────────────────┐
│  4. Review &     │
│     Submit       │
│  ─────────────── │
│  • Order summary │
│  • All flights   │
│  • Total budget  │
│  • Link assets   │
└──────────────────┘
```

### 4.4 Budget Validation

- **Enforcement**: Sum of all flight budgets must be ≤ order total budget (if specified)
- **Calculation**: User can input either:
  - Number of units → System calculates budget (`units × unit_price`)
  - Budget → System calculates units (`budget ÷ unit_price`)

### 4.5 Flight Modification Rules

| Condition | Modification Allowed |
|-----------|---------------------|
| Status: Draft | ✅ Full modification |
| Status: Submitted/Under Review | ✅ Full modification |
| Status: Approved, >72h before go-live | ✅ Full modification |
| Status: Approved, ≤72h before go-live | ❌ No modification (locked) |
| Status: Live/Completed | ❌ No modification |

---

## 5. Pricing System

### 5.1 Pricing Data Structure

| Field | Description | Example |
|-------|-------------|---------|
| `channel` | Media channel | Video, Display |
| `product` | Product type | Instream, Outstream |
| `network` | Inventory source | RLVNT, Broadcaster, CTV, RON |
| `format` | Duration or size | 15, 30, APTO, Big IAB |
| `targeting_allowed` | Can apply targeting | TRUE, FALSE |
| `targeting_price_index` | Price multiplier for targeting | 1.1 (= +10%) |
| `units` | Billing unit type | Completed views, Impressions |
| `default_pricing` | Base price | 0.038 |
| `monthly_prices` | Price by month | Jan: 0.038, Nov: 0.0456, etc. |

### 5.2 Price Calculation Logic

```
1. Look up pricing rule by: channel + product + network + format
2. Get unit price for flight's start date month
3. If targeting enabled AND targeting_allowed:
   - Multiply price by targeting_price_index
4. Calculate:
   - If user enters units: budget = units × unit_price
   - If user enters budget: units = budget ÷ unit_price (rounded)
```

### 5.3 Pricing Management

#### Default Pricing
- Applies to **all organizations** unless overridden
- Managed by RLVNT staff via spreadsheet upload
- CSV format matching current `rlvnt_pricing.csv` structure

#### Custom Pricing
- Can be uploaded **per organization**
- Overrides default pricing for that organization
- Same CSV format as default pricing

#### Pricing Upload Options

| Option | Pros | Cons |
|--------|------|------|
| **CSV Upload** | Simple, familiar, works offline | Manual sync, version control harder |
| **Google Sheets Integration** | Real-time sync, collaborative editing | Requires Google auth, dependency |
| **In-app Editor** | Full control, audit trail | More complex to build |

**Recommendation**: Start with **CSV upload** for MVP, with ability to:
1. Download current pricing as CSV
2. Edit in Excel/Sheets
3. Upload modified CSV
4. Preview changes before applying
5. View upload history

### 5.4 Sample Pricing (Video Instream)

| Network | Format | Unit | Jan-Oct | Nov-Dec |
|---------|--------|------|---------|---------|
| RLVNT | 5s | Completed views | €0.021 | €0.0252 |
| RLVNT | 15s | Completed views | €0.030 | €0.0360 |
| RLVNT | 30s | Completed views | €0.042 | €0.0504 |
| Broadcaster | 15s | Completed views | €0.038 | €0.0456 |
| CTV | 15s | Completed views | €0.051 | €0.0612 |
| SAVOD | 15s | Impressions (CPM) | €41.25 | €49.50 |

### 5.5 Sample Pricing (Display)

| Network | Format | Unit | Jan-Oct | Nov-Dec |
|---------|--------|------|---------|---------|
| RON | APTO | Impressions (CPM) | €15 | €18 |
| RON | Small IAB | Impressions (CPM) | €4 | €4.80 |
| RON | Big IAB | Impressions (CPM) | €8 | €9.60 |
| ROC | APTO | Impressions (CPM) | €20 | €24 |

---

## 6. Campaign Workflow

### 6.1 Status Flow

```
                                    ┌──────────────┐
                                    │   Rejected   │
                                    └──────────────┘
                                          ▲
                                          │ (with reason)
                                          │
┌────────┐    ┌───────────┐    ┌──────────┴──┐    ┌──────────┐    ┌────────┐    ┌───────────┐
│ Draft  │───►│ Submitted │───►│ Under Review│───►│ Approved │───►│  Live  │───►│ Completed │
└────────┘    └───────────┘    └─────────────┘    └──────────┘    └────────┘    └───────────┘
    │              │                                    │
    │              │                                    │
    ▼              ▼                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                     Can resubmit after rejection                        │
└────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Status Definitions

| Status | Description | Triggered By |
|--------|-------------|--------------|
| **Draft** | Order being created, not yet submitted | System (on create) |
| **Submitted** | Order submitted for RLVNT review | External user |
| **Under Review** | RLVNT is actively reviewing | Internal user |
| **Approved** | Order approved, awaiting go-live | Internal user |
| **Rejected** | Order rejected (reason provided) | Internal user |
| **Live** | Campaign is running | Internal user |
| **Completed** | Campaign has ended | System (auto) or Internal user |

### 6.3 Status Transition Rules

| From | To | Conditions | Actor |
|------|-----|------------|-------|
| Draft | Submitted | At least 1 flight, assets linked | External |
| Submitted | Under Review | - | Internal |
| Under Review | Approved | All linked assets verified | Internal |
| Under Review | Rejected | Reason required | Internal |
| Rejected | Submitted | Modifications made | External |
| Approved | Live | Go-live date reached or manual | Internal/System |
| Live | Completed | End date reached or manual | Internal/System |

### 6.4 Approval Requirements

An order **cannot be approved** unless:

- ✅ All required fields are complete
- ✅ At least one flight is defined
- ✅ Budget validation passes
- ✅ All linked creative assets are **verified**

---

## 7. Creative Assets

### 7.1 Asset Management

Creative assets are handled by the **existing asset management component**. Key points for the booking tool integration:

| Feature | Description |
|---------|-------------|
| **Reusability** | Assets can be linked to multiple campaigns/flights |
| **Organization-scoped** | Assets belong to an organization |
| **Verification required** | Only verified assets can be used in approved campaigns |

### 7.2 Asset-Flight Linking

- Flights can have one or more linked assets
- Assets are validated against channel/format specifications (existing functionality)
- Campaign approval blocked until all linked assets are verified

---

## 8. Notifications

### 8.1 Email Notifications

Users should receive email notifications for:

| Event | Recipients | Content |
|-------|------------|---------|
| Order submitted | Internal users | Order summary, link to review |
| Order approved | Order creator | Confirmation, expected go-live |
| Order rejected | Order creator | Rejection reason, how to resubmit |
| Campaign live | Order creator | Confirmation, campaign details |
| Campaign completed | Order creator | Summary, (future: link to report) |
| Flight locked (72h warning) | Order creator | Reminder that modifications closing |

### 8.2 In-App Notifications (Future)

- Notification center in UI
- Real-time updates for status changes
- Activity log per order

---

## 9. Channel & Product Matrix

### 9.1 Current Channels (Complete)

#### Video

| Product | Networks | Formats | Targeting | Unit |
|---------|----------|---------|-----------|------|
| Instream | RLVNT | 5, 6, 10, 12, 15, 20, 21, 25, 30s | ✅ | Completed views |
| Instream | Broadcaster | 5, 6, 10, 12, 15, 20, 21, 25, 30s | ✅ | Completed views |
| Instream | CTV | 5, 6, 10, 12, 15, 20, 21, 25, 30s | ✅ | Completed views |
| Instream | SAVOD | 5, 6, 10, 12, 15s | ❌ | Impressions |
| Outstream | RON | 5, 6s | ✅ | Completed views |

#### Display

| Product | Networks | Formats | Targeting | Unit |
|---------|----------|---------|-----------|------|
| Display | RON | APTO, Small IAB, Big IAB | ✅ | Impressions |
| Display | ROC | APTO, Small IAB, Big IAB | ✅ | Impressions |

### 9.2 Channels To Be Added

| Channel | Products | Status |
|---------|----------|--------|
| Audio | Podcast, Streaming | Pending |
| Social | Rectangle, Video, Carousel, Story | Pending |
| Native | TBD | Pending |
| YouTube | Non-Skip 15", Bumper, CTV Non-Skip, Masthead, Homepage Takeover | Pending |
| DOOH | TBD | Pending |
| Branded Content | Branded Article | Pending |

### 9.3 Targeting Options

Targeting is available for most channel/network combinations (see `targeting_allowed` in pricing).

| Targeting Key | Description | Example Values |
|---------------|-------------|----------------|
| `geo` | Geographic targeting | Country, Region, City, Postal code |
| `demographics` | Demographic targeting | Age range, Gender, Income |
| `device` | Device targeting | Desktop, Mobile, Tablet, CTV |
| *(extensible)* | Additional keys can be added | Interests, Behaviors, Custom segments |

**Implementation Note**: Targeting configuration should use a flexible JSON structure to allow easy addition of new targeting keys without schema changes.

```json
{
  "geo": {
    "countries": ["NL", "BE"],
    "regions": ["Noord-Holland"]
  },
  "demographics": {
    "age_range": ["25-34", "35-44"],
    "gender": ["all"]
  },
  "device": {
    "types": ["mobile", "desktop"]
  }
}
```

---

## 10. Data Model

### 10.1 Entity Relationship Diagram

```
┌─────────────────┐       ┌─────────────────┐
│   Organization  │       │      User       │
├─────────────────┤       ├─────────────────┤
│ id          PK  │       │ id          PK  │
│ name            │       │ email           │
│ created_at      │       │ name            │
│ updated_at      │       │ type (internal/ │
└────────┬────────┘       │   external)     │
         │                │ created_at      │
         │                └────────┬────────┘
         │                         │
         │    ┌────────────────────┘
         │    │
         ▼    ▼
┌─────────────────────┐
│ UserOrganization    │
├─────────────────────┤
│ user_id         FK  │
│ organization_id FK  │
│ role                │
│ is_default          │
└─────────────────────┘

┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│     Order       │       │     Flight      │       │   FlightAsset   │
├─────────────────┤       ├─────────────────┤       ├─────────────────┤
│ id          PK  │◄──────│ id          PK  │◄──────│ flight_id   FK  │
│ name            │       │ order_id    FK  │       │ asset_id    FK  │
│ organization_id │       │ channel         │       └─────────────────┘
│ advertiser_id   │       │ product         │               │
│ start_date      │       │ network         │               │
│ end_date        │       │ format          │               ▼
│ total_budget    │       │ targeting_on    │       ┌─────────────────┐
│ po_number       │       │ targeting_config│       │  CreativeAsset  │
│ status          │       │ start_date      │       ├─────────────────┤
│ created_by  FK  │       │ end_date        │       │ id          PK  │
│ created_at      │       │ unit_type       │       │ organization_id │
│ updated_at      │       │ unit_price      │       │ filename        │
└─────────────────┘       │ units           │       │ url             │
                          │ budget          │       │ type            │
                          │ created_at      │       │ status          │
                          │ updated_at      │       │ specs       JSON│
                          └─────────────────┘       │ verified_at     │
                                                    │ created_at      │
┌─────────────────┐                                 └─────────────────┘
│   PricingRule   │
├─────────────────┤
│ id          PK  │
│ organization_id │  (NULL = default pricing)
│ channel         │
│ product         │
│ network         │
│ format          │
│ targeting_allowed│
│ targeting_index │
│ unit_type       │
│ default_price   │
│ monthly_prices  │  (JSON)
│ valid_from      │
│ valid_until     │
│ created_at      │
└─────────────────┘

┌─────────────────┐
│  Advertiser     │
├─────────────────┤
│ id          PK  │
│ organization_id │
│ name            │
│ external_id     │  (AMP number / CRM reference)
│ created_at      │
└─────────────────┘

┌─────────────────┐
│  StatusHistory  │
├─────────────────┤
│ id          PK  │
│ order_id    FK  │
│ from_status     │
│ to_status       │
│ changed_by  FK  │
│ reason          │
│ created_at      │
└─────────────────┘
```

### 10.2 Key Indexes

| Table | Index | Purpose |
|-------|-------|---------|
| Order | `organization_id, status` | Filter orders by org and status |
| Order | `created_by` | User's orders |
| Flight | `order_id` | Flights per order |
| PricingRule | `organization_id, channel, product, network, format` | Price lookup |
| CreativeAsset | `organization_id, status` | Assets per org |

---

## 11. Technical Considerations

### 11.1 Existing Stack

Based on the current codebase:

| Layer | Technology |
|-------|------------|
| Frontend | Next.js, React, TypeScript |
| UI Components | shadcn/ui, Tailwind CSS |
| Backend | Node.js, Express/Hono |
| Database | PostgreSQL with Drizzle ORM |
| Authentication | NextAuth.js |
| File Storage | Existing asset hosting solution |

### 11.2 New Components Needed

| Component | Purpose |
|-----------|---------|
| Pricing Engine | Calculate unit prices, handle monthly variations |
| Order Service | CRUD for orders and flights |
| Notification Service | Email sending for status changes |
| Pricing Import | CSV parsing and validation |
| Organization Context | Multi-org switching in UI |

### 11.3 API Endpoints (Draft)

```
# Orders
POST   /api/orders                    # Create order
GET    /api/orders                    # List orders (org-scoped)
GET    /api/orders/:id                # Get order with flights
PUT    /api/orders/:id                # Update order
DELETE /api/orders/:id                # Delete order (draft only)
POST   /api/orders/:id/submit         # Submit for review
POST   /api/orders/:id/approve        # Approve order
POST   /api/orders/:id/reject         # Reject order (with reason)

# Flights
POST   /api/orders/:id/flights        # Add flight
PUT    /api/flights/:id               # Update flight
DELETE /api/flights/:id               # Delete flight

# Pricing
GET    /api/pricing/calculate         # Calculate price for selection
GET    /api/pricing/rules             # List pricing rules
POST   /api/pricing/import            # Import pricing CSV
GET    /api/pricing/export            # Export pricing CSV

# Organizations
GET    /api/organizations             # List user's organizations
PUT    /api/organizations/:id/pricing # Upload custom pricing
```

---

## 12. Scope & Roadmap

### 12.1 MVP Scope

| Feature | In Scope |
|---------|----------|
| Order creation with flights | ✅ |
| Video & Display channels | ✅ |
| Dynamic pricing calculation | ✅ |
| Budget validation | ✅ |
| Campaign status workflow | ✅ |
| Multi-organization support | ✅ |
| Internal admin dashboard | ✅ |
| Email notifications | ✅ |
| CSV pricing upload | ✅ |
| Custom pricing per org | ✅ |
| Creative asset linking | ✅ |
| Flight modification (72h rule) | ✅ |

### 12.2 Out of Scope (Future)

| Feature | Notes |
|---------|-------|
| Reporting & Analytics | Phase 2 |
| Audio, Social, Native, YouTube, DOOH, Branded Content | Add as pricing defined |
| External platform integrations | Booking remains manual |
| In-app notifications | Email only for MVP |
| Advanced role-based permissions | All internal = admin for MVP |
| Google Sheets integration | CSV upload for MVP |

### 12.3 Suggested Development Phases

#### Phase 1: Foundation (Weeks 1-2)
- Database schema and migrations
- Organization context and switching
- Basic order CRUD
- Flight CRUD with validation

#### Phase 2: Pricing Engine (Weeks 3-4)
- Pricing rule storage
- CSV import/export
- Price calculation API
- Custom pricing per organization

#### Phase 3: Workflow (Weeks 5-6)
- Status management
- Approval flow
- 72-hour lock rule
- Status history tracking

#### Phase 4: Notifications & Polish (Weeks 7-8)
- Email notification system
- Internal dashboard
- Budget validation
- Asset-flight linking
- UI polish and testing

---

## Appendix A: Glossary

| Term | Definition |
|------|------------|
| APTO | Ad format type (Display) |
| AVOD | Advertising Video On Demand |
| BVOD | Broadcaster Video On Demand |
| CPM | Cost Per Mille (cost per 1000 impressions) |
| CTV | Connected TV |
| DOOH | Digital Out-of-Home |
| FAST | Free Ad-Supported Television |
| IAB | Interactive Advertising Bureau (standard ad sizes) |
| ROC | Run of Channel |
| RON | Run of Network |
| SAVOD | Subscription Ad-supported Video On Demand |
| SVOD | Subscription Video On Demand |

---

## Appendix B: Reference Documents

- Miro Board: [RLVNT Portal Flow](https://miro.com/app/board/uXjVGYfPQyA=/)
- RLVNT Website: [rlvnt.world](https://rlvnt.world)
- Pricing Template: `rlvnt_pricing.csv`
