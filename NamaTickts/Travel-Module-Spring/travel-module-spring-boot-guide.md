# Travel Module — Spring Boot Learning Guide

> A complete journey from understanding a real ERP module to building it from scratch with Spring Boot.

---

## Table of Contents

- [Step 1 — Business Understanding](#step-1--business-understanding)
- [Step 2 — Business Requirements](#step-2--business-requirements)
- [Step 3 — Structured Requirements Document](#step-3--structured-requirements-document)
- [Step 4 — Spring Boot Learning Levels](#step-4--spring-boot-learning-levels)
- [Step 5 — Task-by-Task Implementation Guide](#step-5--task-by-task-implementation-guide)

---

# Step 1 — Business Understanding

## What Is This Module?

You run a **travel agency**. Every day you:

1. **Register your partners** — hotels, restaurants, tour guides you work with
2. **Design tour programs** — reusable itinerary templates like "7 Days in Egypt Classic"
3. **Run actual tours** — when customers book, you create a real tour with real dates and real bookings
4. **Issue vouchers** — hand guests hotel/restaurant vouchers so they can check in
5. **Handle money** — buy services from suppliers (Purchase Orders/Invoices), sell packages to customers (Sales Orders/Invoices)

## The Five Layers of Data

### Layer 1: Reference Data (The "Phone Book")

Simple lookup tables. Rarely change once created.

| Entity | What It Is | Examples |
|--------|-----------|---------|
| **Country** | A country | Egypt, Turkey, UAE |
| **City** | A city or airport | Cairo, Istanbul, Dubai |
| **Hotel Class** | Star rating / category | 5-Star, 4-Star, Boutique |
| **Restaurant Class** | Restaurant category | Fine Dining, Casual |
| **Tour Service Class** | Service category | Transfer, Sightseeing, Guided Tour |

A City belongs to a Country. A City can be a "sub-city" of another (e.g. "Cairo Airport" is a sub-city of "Cairo").

### Layer 2: Supplier Master Files (The "Partners")

Your business partners — the companies and people you buy services from.

| Entity | What It Is | Key Data |
|--------|-----------|----------|
| **Hotel** | A hotel you work with | Class, country, city, contacts, bank accounts, tax info |
| **Restaurant** | A restaurant partner | Class, country, city, contacts, bank account, tax info |
| **Tour Guide** | A freelance guide | Languages spoken, country, city, contacts, bank account |
| **Tour Service** | A sellable service item | Category, tax plan, accounting accounts |

Each partner stores: multiple contact persons, bank details for payments, subsidiary accounting accounts, and tax registration.

### Layer 3: Tour Planning (The "Product")

| Entity | What It Is | Analogy |
|--------|-----------|---------|
| **Tour Program** | A reusable tour template | A **recipe** — "7 Days Egypt Classic" |
| **Tour** | An actual running tour | A **cooked meal** — the real trip on specific dates |

**Tour Program** contains:
- Accommodation lines (which hotels, how many nights, room types)
- Service lines (which activities, on which day number)
- Flight lines (which routes)
- Staff assignments (guide, leader, operator)
- Status: `Initial → InProgress → Finished → Canceled`

**Tour** contains everything a Program has, PLUS:
- Actual dates (arrival, departure)
- Actual flight times
- A customer (the travel agent who booked)
- Links back to Purchase Orders for each hotel/flight/service booking

### Layer 4: Operational Documents (The "Tickets")

| Entity | What It Is |
|--------|-----------|
| **Hotel Voucher** | Check-in document given to the guest for a hotel stay |
| **Restaurant Voucher** | Reservation document for a restaurant meal |

Vouchers reference: a Tour, a Guest (Customer), and the specific hotel/restaurant. They include check-in/out dates, guest counts (adults, children, pax), room types (single, double, triple, suite), and meal plans (BB, HB, FB, All-Inclusive).

### Layer 5: Financial Documents (The "Money")

The classic purchase/sales cycle:

```
PURCHASE SIDE (buying from suppliers):
  Purchase Order  →  Purchase Invoice  →  Purchase Return

SALES SIDE (selling to customers):
  Sales Order  →  Sales Invoice  →  Sales Return
```

Every financial document has:
- **Header** — who (supplier/customer), total amount, tax, currency, due date
- **Detail Lines** — which tour service, quantity, unit price, line total
- **Payment Lines** — how payment was made (cash, card, bank transfer)
- **Schedule Payment Lines** — installment plan (30% now, 70% in 30 days)
- **Standard Term Lines** — terms and conditions attached
- **External Payment Lines** — payments tracked in other documents

## Key Workflows

### Workflow 1: Tour Lifecycle

```
Create Tour Program (reusable template)
       │
       ▼
Create Tour (actual trip based on program)
       │
       ▼
Add accommodation, flight, and service lines
       │
       ▼
Create Purchase Orders for each supplier (hotel, airline, guide)
       │
       ▼
Issue Hotel Vouchers and Restaurant Vouchers to guests
       │
       ▼
Create Sales Invoice to the customer (travel agent)
       │
       ▼
Receive Purchase Invoices from suppliers
       │
       ▼
Tour status moves: Initial → InProgress → Finished
```

### Workflow 2: Purchase Cycle

```
Purchase Order (you commit to buying from a supplier)
       │
       ▼
Purchase Invoice (supplier sends you a bill)
       │
       ▼
Purchase Return (if something goes wrong, you get a credit)
```

### Workflow 3: Sales Cycle

```
Sales Order (customer requests a package)
       │
       ▼
Sales Invoice (you bill the customer)
       │
       ▼
Sales Return (partial cancellation or refund)
```

## Key ERP Design Patterns

| Pattern | What It Means | Example |
|---------|--------------|---------|
| **Master File vs Document** | Static data vs transactions | Hotel (master) vs Purchase Invoice (document) |
| **Header / Lines** | Every document has a summary header and item detail lines | Tour header + accommodation lines |
| **Abstract Inheritance** | Shared fields extracted to a base class | All invoices share the same money/tax fields |
| **Status Workflow** | Entities move through lifecycle states | Initial → InProgress → Finished → Canceled |
| **Template → Instance** | Create a template, then instantiate it | Tour Program → Tour |
| **Generic References** | One field can point to different entity types | Subsidiary can be Hotel, Supplier, or Employee |
| **Accounting Integration** | Documents post journal entries to the ledger | Sales Invoice creates a GL posting |

---

# Step 2 — Business Requirements

## 2.1 Business Context

A travel agency needs a system to:
- Manage its network of hotels, restaurants, and tour guides
- Design and sell tour packages to travel agents and direct customers
- Track all purchases from suppliers and sales to customers
- Issue operational vouchers for guest check-ins
- Integrate with accounting for financial reporting

## 2.2 Functional Requirements

### FR-1: Reference Data Management

| ID | Requirement |
|----|-------------|
| FR-1.1 | The system shall allow creating, editing, and listing Countries |
| FR-1.2 | The system shall allow creating, editing, and listing Cities, each linked to a Country |
| FR-1.3 | A City may optionally be a sub-city of another City |
| FR-1.4 | The system shall allow creating Hotel Classes, Restaurant Classes, and Tour Service Classes as simple lookup values |

### FR-2: Supplier Management

| ID | Requirement |
|----|-------------|
| FR-2.1 | The system shall allow creating Hotels with: class, country, city, salesman, contacts, bank accounts, tax info, and attachments |
| FR-2.2 | The system shall allow creating Restaurants with: class, country, city, salesman, contacts, bank account, and tax info |
| FR-2.3 | The system shall allow creating Tour Guides with: country, city, languages spoken, contacts, bank account, and tax info |
| FR-2.4 | The system shall allow creating Tour Services with: service class, tax plan, and accounting accounts |
| FR-2.5 | Each supplier entity shall support multiple contact persons (name, job title, phone, mobile, fax, email, address) |
| FR-2.6 | Hotels shall support up to 5 bank accounts with different currencies |

### FR-3: Tour Program Management

| ID | Requirement |
|----|-------------|
| FR-3.1 | The system shall allow creating Tour Programs as reusable templates |
| FR-3.2 | A Tour Program shall have a status lifecycle: Initial → InProgress → Finished → Canceled |
| FR-3.3 | A Tour Program shall contain accommodation lines (country, city, hotel, room types, nights, meal plan, situation) |
| FR-3.4 | A Tour Program shall contain service lines (country, city, service, day number, pax, guide, supplier) |
| FR-3.5 | A Tour Program shall contain flight lines (flight number, route, airline, day number, pax, situation) |
| FR-3.6 | Each line's situation can be: OK (confirmed), RQ (requested), or WL (wait-listed) |

### FR-4: Tour Management

| ID | Requirement |
|----|-------------|
| FR-4.1 | The system shall allow creating Tours based on a Tour Program |
| FR-4.2 | A Tour shall copy the structure of its Program (accommodation, services, flights) but add real dates |
| FR-4.3 | Tour accommodation lines shall include check-in/check-out dates and a link to a Purchase Order |
| FR-4.4 | Tour service lines shall include actual date/time and links to Purchase Orders (service, guide, restaurant) |
| FR-4.5 | Tour flight lines shall include actual date/time and a link to a Purchase Order |
| FR-4.6 | A Tour shall reference a customer (travel agent), nationality, and arrival/departure airports |

### FR-5: Voucher Management

| ID | Requirement |
|----|-------------|
| FR-5.1 | The system shall allow creating Hotel Vouchers linked to a Tour, a Guest, and a Hotel |
| FR-5.2 | A Hotel Voucher shall record: check-in/out dates, pax (adults/children), room types (sgl/dbl/tpl/suite), meal plans (BB/HB/FB/AI) |
| FR-5.3 | The system shall allow creating Restaurant Vouchers linked to a Tour, a Guest, and a Restaurant |
| FR-5.4 | A Restaurant Voucher shall record: meal date, meal description, pax (adults/children) |
| FR-5.5 | Vouchers shall indicate whether extras are Included or Excluded |

### FR-6: Purchase Documents

| ID | Requirement |
|----|-------------|
| FR-6.1 | The system shall support Purchase Orders with header (supplier, money, tax) and detail lines (tour service, quantity, price) |
| FR-6.2 | The system shall support Purchase Invoices that may reference Purchase Orders |
| FR-6.3 | The system shall support Purchase Returns for refunds/credits |
| FR-6.4 | Purchase Invoices shall support multi-currency lines (currency + exchange rate per line) |
| FR-6.5 | All purchase documents shall support payment lines, schedule payment lines, standard terms, and external payment references |

### FR-7: Sales Documents

| ID | Requirement |
|----|-------------|
| FR-7.1 | The system shall support Sales Orders with header (customer, salesman, money, tax) and detail lines |
| FR-7.2 | The system shall support Sales Invoices for billing customers |
| FR-7.3 | The system shall support Sales Returns for customer refunds |
| FR-7.4 | All sales documents shall support payment lines, schedule payment lines, standard terms, and external payment references |

### FR-8: Accounting Integration

| ID | Requirement |
|----|-------------|
| FR-8.1 | Purchase and Sales Invoices shall generate accounting entries (GL postings) |
| FR-8.2 | Each invoice line shall be mappable to a subsidiary account |
| FR-8.3 | Tax calculations shall follow configured tax plans |

## 2.3 Constraints and Validations

### Entity-Level Validations

| Entity | Rule |
|--------|------|
| City | Must have a Country. If `subCity = true`, must have a `subCityFor` reference |
| Hotel | Must have a Hotel Class. Must have a Country and City |
| Tour Program | Cannot transition from Finished/Canceled back to Initial |
| Tour | Must reference a valid Tour Program (optional but recommended) |
| Hotel Voucher | Must reference a valid Tour and Hotel. Check-out must be after check-in |
| Restaurant Voucher | Must reference a valid Tour and Restaurant |

### Document Validations

| Rule | Applies To |
|------|-----------|
| Detail lines are required (at least 1 line) | All purchase and sales documents |
| Quantity must be > 0 on each line | All invoice detail lines |
| Total = sum of line totals | All purchase and sales documents |
| Schedule payment total must equal document total | When schedule payments are used |
| Supplier is required on header | All purchase documents |
| Customer is required on header | All sales documents |
| Currency and exchange rate required on purchase invoice lines | Purchase Invoice lines only |

### Status Transition Rules

```
Tour / Tour Program Status Transitions:

  Initial ──────→ InProgress
     │                │
     │                ▼
     │            Finished
     │
     └──────────→ Canceled

Invalid transitions:
  - Finished → Initial (cannot reopen)
  - Canceled → Initial (cannot reopen)
  - Finished → InProgress (cannot revert)
  - Canceled → InProgress (cannot revert)
```

### Booking Situation Rules

```
Accommodation / Flight Situation:

  RQ (Requested) → OK (Confirmed)
  RQ (Requested) → WL (Wait-listed)
  WL (Wait-listed) → OK (Confirmed)
  WL (Wait-listed) → RQ (Requested)
  OK (Confirmed) → RQ (Requested)  [re-request]
```

## 2.4 Edge Cases

| # | Edge Case | Expected Behavior |
|---|-----------|-------------------|
| 1 | Tour created without a Tour Program | Allowed — user fills in all data manually |
| 2 | Tour Program has no accommodation lines | Valid — a tour may be services-only (e.g., day trips) |
| 3 | Hotel Voucher for a Canceled tour | System should warn but may allow (for already-booked rooms) |
| 4 | Purchase Invoice amount doesn't match PO | Allowed — invoices may differ from PO (partial delivery, price changes) |
| 5 | Sales Return amount exceeds original Sales Invoice | Should be rejected — cannot return more than what was sold |
| 6 | City marked as sub-city but no parent specified | Validation error — `subCityFor` is required when `subCity = true` |
| 7 | Multiple Purchase Orders for the same hotel in one Tour | Allowed — different rooms or date ranges may need separate POs |
| 8 | Schedule payment lines don't sum to document total | Validation error — total of installments must match document total |
| 9 | Changing Tour status from InProgress to Canceled | Allowed — but should warn if there are unresolved Purchase Orders |
| 10 | Deleting a Hotel that is referenced by active Tours | Should be blocked — master data integrity must be maintained |

## 2.5 User Actions

| Action | Who Does It | What Happens |
|--------|------------|--------------|
| Register a new Hotel | Operations Manager | Creates master file with all details, assigns to salesman |
| Design a Tour Program | Product Manager | Creates template with planned accommodations, services, flights |
| Create a Tour | Tour Operator | Instantiates a program with real dates, assigns agent |
| Book Hotel for Tour | Tour Operator | Adds accommodation line to Tour, creates Purchase Order |
| Issue Hotel Voucher | Tour Operator | Creates voucher document for guest check-in |
| Create Sales Invoice | Accountant | Bills the customer/agent for the tour |
| Record Supplier Invoice | Accountant | Enters Purchase Invoice from hotel/airline/guide |
| Process Payment | Accountant | Adds payment lines to invoice (cash, card, transfer) |
| Cancel a Tour | Manager | Changes status to Canceled, triggers review of related POs |

---

# Step 3 — Structured Requirements Document

## 3.1 Module Overview

**Module Name**: Travel & Tour Management
**Domain**: Travel Agency Operations
**Core Purpose**: End-to-end management of tour packages from design through execution and billing

**Scope for Spring Boot Implementation**:

We will implement a focused subset to learn all major patterns:

| Area | Entities Included |
|------|-------------------|
| Reference Data | Country, City |
| Suppliers | Hotel (with contacts), Tour Service |
| Tour Planning | Tour Program (with accommodation + service lines), Tour |
| Operations | Hotel Voucher |
| Purchasing | Purchase Order (with detail lines) |
| Sales | Sales Invoice (with detail lines + payment lines) |

This subset covers: simple master files, rich master files with detail collections, the header/lines pattern, template-to-instance, status workflow, and the purchase/sales cycle.

## 3.2 Entity Descriptions

### Country

```
Type: Master File (simple reference)
Fields:
  - code        : String, unique, required (e.g., "EG", "TR")
  - name        : String, required (e.g., "Egypt")
  - name2       : String, optional (Arabic name)
Operations: CRUD + List
```

### City

```
Type: Master File (reference with relationship)
Fields:
  - code        : String, unique, required
  - name        : String, required
  - name2       : String, optional
  - country     : FK → Country, required
  - subCity     : Boolean, default false
  - subCityFor  : FK → City, required if subCity = true
Operations: CRUD + List + Filter by Country
```

### Hotel

```
Type: Master File (rich, with collections)
Fields:
  - code          : String, unique, required
  - name          : String, required
  - name2         : String, optional
  - hotelClass    : FK → HotelClass, required
  - country       : FK → Country, required
  - city          : FK → City, required
  - salesMan      : FK → Employee (simplified as String for learning)
  - bankAccount   : String, optional
Collections:
  - contacts[]    : List of ContactInfo
      - contactName   : String
      - jobDescription: String
      - phone         : String
      - mobile        : String
      - email         : String
Operations: CRUD + List + Filter by Country/City/Class
```

### Tour Service

```
Type: Master File (catalog item)
Fields:
  - code              : String, unique, required
  - name              : String, required
  - name2             : String, optional
  - tourServiceClass  : FK → TourServiceClass, required
Operations: CRUD + List
```

### Tour Program

```
Type: Master File (template with collections)
Fields:
  - code              : String, unique, required
  - name              : String, required
  - status            : Enum (INITIAL, IN_PROGRESS, FINISHED, CANCELED)
  - tourGuide         : FK → TourGuide (simplified as String)
  - tourLeader        : String
  - tourOperator      : String
  - groupName         : String
  - nationality       : String
  - pax               : Integer
  - sgl               : Integer (single rooms)
  - dbl               : Integer (double rooms)
  - tpl               : Integer (triple rooms)
  - tourDays          : Integer
  - arrivalAirport    : FK → City
  - departureAirport  : FK → City
Collections:
  - accommodationLines[] :
      - country       : FK → Country
      - city          : FK → City
      - hotel         : FK → Hotel
      - nights        : Integer, required
      - sgl, dbl, tpl : Integer (room counts)
      - pax           : Integer
      - meals         : String
      - situation     : Enum (OK, RQ, WL)
  - serviceLines[] :
      - country       : FK → Country
      - city          : FK → City
      - service       : FK → TourService
      - dayNo         : Integer
      - pax           : Integer
      - description   : String
  - flightLines[] :
      - flight        : String (flight number)
      - route         : String
      - dayNo         : Integer
      - pax           : Integer
      - situation     : Enum (OK, RQ, WL)
Operations: CRUD + List + Status transitions
```

### Tour

```
Type: Document (transaction instance)
Fields:
  - documentNumber    : String, auto-generated, unique
  - documentDate      : Date, required
  - tourProgram       : FK → TourProgram (optional)
  - status            : Enum (INITIAL, IN_PROGRESS, FINISHED, CANCELED)
  - customer          : String (agent name)
  - tourGuide         : String
  - tourLeader        : String
  - tourOperator      : String
  - groupName         : String
  - nationality       : String
  - pax, sgl, dbl, tpl: Integer
  - arrivalDate       : Date
  - departureDate     : Date
  - arrivalAirport    : FK → City
  - departureAirport  : FK → City
  - arrivalFlight     : String
  - departureFlight   : String
Collections:
  - accommodationLines[] :
      (same as Tour Program lines, plus:)
      - checkIn       : Date
      - checkOut      : Date
      - purchaseOrder : FK → PurchaseOrder (optional)
  - serviceLines[] :
      (same as Tour Program lines, plus:)
      - date          : Date
      - time          : Time
      - purchaseOrder : FK → PurchaseOrder (optional)
  - flightLines[] :
      (same as Tour Program lines, plus:)
      - date          : Date
      - time          : Time
      - purchaseOrder : FK → PurchaseOrder (optional)
Operations: CRUD + List + Create from Program + Status transitions
```

### Hotel Voucher

```
Type: Document (operational)
Fields:
  - documentNumber    : String, auto-generated
  - documentDate      : Date, required
  - tour              : FK → Tour, required
  - guest             : String, required
  - hotel             : FK → Hotel, required
  - checkIn           : Date, required
  - checkOut          : Date, required
  - pax               : Integer
  - adults            : Integer
  - children          : Integer
  - sgl, dbl, tpl, suite : Integer (room counts)
  - mealPlan          : String (BB/HB/FB/AI)
  - extras            : Enum (INCLUDED, EXCLUDED)
Validations:
  - checkOut must be after checkIn
  - tour must exist and not be Canceled
Operations: CRUD + List + Filter by Tour
```

### Purchase Order

```
Type: Document (financial, header/lines)
Header Fields:
  - documentNumber    : String, auto-generated
  - documentDate      : Date, required
  - supplier          : String, required
  - supplierName      : String
  - totalAmount       : Decimal, calculated
  - taxAmount         : Decimal
  - netAmount         : Decimal
  - dueDate           : Date
  - notes             : String
Detail Lines:
  - tourService       : FK → TourService, required
  - description       : String
  - quantity          : Decimal, required, > 0
  - unitPrice         : Decimal, required
  - lineTotal         : Decimal, calculated (quantity * unitPrice)
  - remarks           : String
Validations:
  - At least 1 detail line required
  - Quantity must be > 0
  - totalAmount = sum of all lineTotal values
Operations: CRUD + List + Filter by Supplier
```

### Sales Invoice

```
Type: Document (financial, header/lines + payments)
Header Fields:
  - documentNumber    : String, auto-generated
  - documentDate      : Date, required
  - customer          : String, required
  - salesMan          : String
  - totalAmount       : Decimal, calculated
  - taxAmount         : Decimal
  - netAmount         : Decimal
  - dueDate           : Date
Detail Lines:
  - tourService       : FK → TourService, required
  - description       : String
  - quantity          : Decimal, required, > 0
  - unitPrice         : Decimal, required
  - lineTotal         : Decimal, calculated
  - remarks           : String
Payment Lines:
  - paymentMethod     : String, required (CASH, CARD, BANK_TRANSFER)
  - amount            : Decimal, required
  - authorizationNo   : String (for card payments)
  - date              : Date
Validations:
  - At least 1 detail line required
  - Total of payment lines must not exceed totalAmount
  - Quantity must be > 0
Operations: CRUD + List + Filter by Customer
```

## 3.3 REST API Requirements

### Reference Data APIs

```
Countries:
  GET    /api/countries              — List all countries
  GET    /api/countries/{id}         — Get country by ID
  POST   /api/countries              — Create country
  PUT    /api/countries/{id}         — Update country
  DELETE /api/countries/{id}         — Delete country (if not referenced)

Cities:
  GET    /api/cities                 — List all cities
  GET    /api/cities?countryId={id}  — Filter cities by country
  GET    /api/cities/{id}            — Get city by ID
  POST   /api/cities                 — Create city
  PUT    /api/cities/{id}            — Update city
  DELETE /api/cities/{id}            — Delete city (if not referenced)
```

### Supplier APIs

```
Hotels:
  GET    /api/hotels                         — List all hotels
  GET    /api/hotels?countryId=&cityId=      — Filter hotels
  GET    /api/hotels/{id}                    — Get hotel with contacts
  POST   /api/hotels                         — Create hotel
  PUT    /api/hotels/{id}                    — Update hotel
  DELETE /api/hotels/{id}                    — Delete hotel (if not referenced)
  POST   /api/hotels/{id}/contacts           — Add contact person
  DELETE /api/hotels/{id}/contacts/{cId}     — Remove contact person

Tour Services:
  GET    /api/tour-services                  — List all
  GET    /api/tour-services/{id}             — Get by ID
  POST   /api/tour-services                  — Create
  PUT    /api/tour-services/{id}             — Update
  DELETE /api/tour-services/{id}             — Delete (if not referenced)
```

### Tour Planning APIs

```
Tour Programs:
  GET    /api/tour-programs                  — List all programs
  GET    /api/tour-programs/{id}             — Get program with all lines
  POST   /api/tour-programs                  — Create program
  PUT    /api/tour-programs/{id}             — Update program
  PATCH  /api/tour-programs/{id}/status      — Change status (with validation)
  DELETE /api/tour-programs/{id}             — Delete (only if INITIAL status)

Tours:
  GET    /api/tours                          — List all tours
  GET    /api/tours/{id}                     — Get tour with all lines
  POST   /api/tours                          — Create tour manually
  POST   /api/tours/from-program/{programId} — Create tour from program template
  PUT    /api/tours/{id}                     — Update tour
  PATCH  /api/tours/{id}/status              — Change status
```

### Operations APIs

```
Hotel Vouchers:
  GET    /api/hotel-vouchers                 — List all vouchers
  GET    /api/hotel-vouchers?tourId={id}     — Filter by tour
  GET    /api/hotel-vouchers/{id}            — Get voucher
  POST   /api/hotel-vouchers                 — Create voucher
  PUT    /api/hotel-vouchers/{id}            — Update voucher
  DELETE /api/hotel-vouchers/{id}            — Delete voucher
```

### Financial Document APIs

```
Purchase Orders:
  GET    /api/purchase-orders                — List all
  GET    /api/purchase-orders/{id}           — Get with lines
  POST   /api/purchase-orders                — Create with lines
  PUT    /api/purchase-orders/{id}           — Update header and lines
  DELETE /api/purchase-orders/{id}           — Delete (only if no invoice references it)

Sales Invoices:
  GET    /api/sales-invoices                 — List all
  GET    /api/sales-invoices/{id}            — Get with lines and payments
  POST   /api/sales-invoices                 — Create with lines
  PUT    /api/sales-invoices/{id}            — Update
  POST   /api/sales-invoices/{id}/payments   — Add payment line
  DELETE /api/sales-invoices/{id}            — Delete (only if no returns reference it)
```

## 3.4 Validation Rules Summary

| Rule ID | Entity | Rule Description |
|---------|--------|------------------|
| V-01 | City | `country` is required |
| V-02 | City | If `subCity = true`, then `subCityFor` is required |
| V-03 | City | `subCityFor` must not reference itself (no circular) |
| V-04 | Hotel | `hotelClass`, `country`, and `city` are required |
| V-05 | Hotel | `city` must belong to the specified `country` |
| V-06 | Tour Program | Status transitions follow the allowed state machine |
| V-07 | Tour Program | Cannot delete if status is not INITIAL |
| V-08 | Tour | `arrivalDate` must be before or equal to `departureDate` |
| V-09 | Tour | When created from a program, all program lines are copied |
| V-10 | Hotel Voucher | `checkOut` must be after `checkIn` |
| V-11 | Hotel Voucher | `tour` must not be in CANCELED status |
| V-12 | Purchase Order | Must have at least 1 detail line |
| V-13 | Purchase Order | Each line: `quantity > 0` |
| V-14 | Purchase Order | `totalAmount` must equal the sum of `lineTotal` values |
| V-15 | Sales Invoice | Must have at least 1 detail line |
| V-16 | Sales Invoice | Sum of payment amounts must not exceed `totalAmount` |
| V-17 | All Masters | Cannot delete if referenced by any document |

## 3.5 Example Scenarios

### Scenario 1: Create a Tour Package for "5 Nights Cairo & Luxor"

```
1. Ensure Countries exist: Egypt
2. Ensure Cities exist: Cairo (EG), Luxor (EG)
3. Create Hotels: Marriott Cairo (5-star), Winter Palace Luxor (5-star)
4. Create Tour Services: Airport Transfer, Nile Cruise, Pyramids Tour

5. Create Tour Program "Cairo & Luxor Classic":
   - 5 days, 10 pax, 5 dbl rooms
   - Accommodation lines:
     Line 1: Marriott Cairo, 2 nights, 5 dbl, FB, situation=OK
     Line 2: Winter Palace Luxor, 3 nights, 5 dbl, HB, situation=RQ
   - Service lines:
     Line 1: Airport Transfer, Day 1, 10 pax
     Line 2: Pyramids Tour, Day 2, 10 pax
     Line 3: Nile Cruise, Day 3, 10 pax
   - Status: INITIAL → IN_PROGRESS
```

### Scenario 2: Run an Actual Tour

```
1. POST /api/tours/from-program/{programId}
   - Sets arrival date: 2025-03-15, departure: 2025-03-20
   - Copies all lines from Tour Program
   - Sets customer: "Alpha Travel Agency"

2. Accommodation lines now have actual check-in/check-out dates:
   Line 1: Marriott, checkIn=Mar15, checkOut=Mar17
   Line 2: Winter Palace, checkIn=Mar17, checkOut=Mar20

3. Create Purchase Orders for each hotel:
   PO-001: Marriott Cairo, 10 pax * 2 nights * $150 = $3,000
   PO-002: Winter Palace, 10 pax * 3 nights * $200 = $6,000

4. Issue Hotel Vouchers for guests
5. Create Sales Invoice: $12,000 to Alpha Travel Agency
6. Move Tour status: INITIAL → IN_PROGRESS → FINISHED
```

### Scenario 3: Handle a Partial Cancellation

```
1. Tour is IN_PROGRESS
2. 3 guests cancel out of 10
3. Update Tour: pax = 7, dbl = 4 (reduced from 5)
4. Update accommodation lines (reduce rooms)
5. Issue new Purchase Orders or modify existing POs
6. Create Sales Return for the 3 canceled guests' portion
7. If all guests cancel: Tour status → CANCELED
```

---

# Step 4 — Spring Boot Learning Levels

## Scope for Implementation

We will build a **subset** of the Travel module that teaches all the key patterns:

```
Entities to build:
  ✓ Country          (simple master)
  ✓ City             (master with FK)
  ✓ Hotel            (rich master with collection)
  ✓ Tour Service     (catalog master)
  ✓ Tour Program     (template with 3 collections + status)
  ✓ Tour             (document instance from template)
  ✓ Hotel Voucher    (operational document)
  ✓ Purchase Order   (financial document with lines)
  ✓ Sales Invoice    (financial document with lines + payments)
```

---

## Level 1 — The Basics (Simplified Version)

**Goal**: Get a running Spring Boot app with basic CRUD. No database yet — use in-memory storage.

**What you learn**: Spring Boot project setup, REST controllers, basic service layer, request/response handling.

### L1-Task 1: Project Setup

Create a Spring Boot project with:
- Java 21
- Spring Web
- Spring Validation
- Maven
- Package structure:
  ```
  com.travel
  ├── controller/
  ├── service/
  ├── model/
  └── TravelApplication.java
  ```

**Acceptance criteria**: Application starts on port 8080, returns 200 on `GET /api/health`.

### L1-Task 2: Country CRUD (In-Memory)

Build the simplest possible master file:
- `Country` model class with: `id` (Long), `code` (String), `name` (String)
- `CountryService` with a `HashMap<Long, Country>` as storage
- `CountryController` with full CRUD endpoints
- Input validation: `code` and `name` required, `code` must be unique

**Acceptance criteria**: All 5 CRUD operations work via Postman/curl. Duplicate code is rejected.

### L1-Task 3: City with Foreign Key (In-Memory)

Build a master file that references another:
- `City` model with: `id`, `code`, `name`, `countryId`, `subCity`, `subCityForId`
- `CityService` validates that `countryId` references an existing Country
- If `subCity = true`, validates that `subCityForId` exists and is not self-referencing
- Filter endpoint: `GET /api/cities?countryId=1`

**Acceptance criteria**: Cannot create a City with an invalid Country. Sub-city validation works.

### L1-Task 4: Tour Program with Lines (In-Memory)

Build the first Header/Lines entity:
- `TourProgram` with basic header fields + `List<AccommodationLine>`
- Lines are embedded in the Tour Program (no separate endpoint)
- Create/Update sends the full object (header + lines together)
- Status field with allowed transitions

**Acceptance criteria**: Can create a Tour Program with 3 accommodation lines. Status transitions are enforced.

### L1-Task 5: Purchase Order with Calculated Fields

Build a financial document:
- `PurchaseOrder` header + `List<PurchaseOrderLine>`
- `lineTotal` = quantity * unitPrice (calculated, not user-input)
- `totalAmount` = sum of all lineTotals (calculated on header)
- Validation: at least 1 line, quantity > 0

**Acceptance criteria**: Totals auto-calculate. Invalid data is rejected with clear error messages.

---

## Level 2 — ERP Structure (Database + Relationships)

**Goal**: Replace in-memory storage with a real database. Add proper JPA entities and relationships.

**What you learn**: JPA/Hibernate, entity relationships, cascading, database constraints, DTOs.

### L2-Task 1: Add Database Layer

- Add Spring Data JPA + H2 (development) dependencies
- Convert model classes to `@Entity` classes
- Set up JPA relationships:
  - `City` → `@ManyToOne Country`
  - `Hotel` → `@ManyToOne Country`, `@ManyToOne City`, `@ManyToOne HotelClass`
  - `Hotel` → `@OneToMany ContactInfo` (cascade ALL)
- Create `@Repository` interfaces
- Replace HashMap services with repository calls

**Acceptance criteria**: Data persists across requests (within same run). Relationships are enforced at DB level.

### L2-Task 2: Tour Program with Proper Collections

- `TourProgram` → `@OneToMany AccommodationLine` (cascade ALL, orphanRemoval)
- `TourProgram` → `@OneToMany ServiceLine` (cascade ALL, orphanRemoval)
- `TourProgram` → `@OneToMany FlightLine` (cascade ALL, orphanRemoval)
- Each line has a `@ManyToOne` back-reference to the parent

**Acceptance criteria**: Adding/removing lines works correctly. Orphan lines are cleaned up.

### L2-Task 3: Tour Document (Template-to-Instance)

- `Tour` entity with all fields from Tour Program plus dates, customer, etc.
- `POST /api/tours/from-program/{id}` — copies all lines from program into a new Tour
- Tour accommodation lines get additional fields: checkIn, checkOut

**Acceptance criteria**: Creating a Tour from a Program copies all lines correctly. Tour has its own independent lines.

### L2-Task 4: Purchase Order with Proper Calculations

- `PurchaseOrder` → `@OneToMany PurchaseOrderLine`
- Add `@PrePersist` / `@PreUpdate` to auto-calculate totals
- Use `BigDecimal` for all money fields
- Add optimistic locking with `@Version`

**Acceptance criteria**: Totals are always consistent. Concurrent edits are detected.

### L2-Task 5: Sales Invoice with Payment Lines

- `SalesInvoice` → `@OneToMany SalesInvoiceLine` + `@OneToMany PaymentLine`
- Validation: sum of payments must not exceed total
- Payment methods: CASH, CARD, BANK_TRANSFER

**Acceptance criteria**: Payment validation works. Can partially pay an invoice.

### L2-Task 6: Hotel Voucher with Cross-Entity Validation

- `HotelVoucher` → `@ManyToOne Tour`, `@ManyToOne Hotel`
- Validation: checkOut > checkIn, Tour must not be CANCELED
- Filter: `GET /api/hotel-vouchers?tourId=1`

**Acceptance criteria**: Cannot create voucher for canceled tour. Date validation works.

---

## Level 3 — Realistic Implementation

**Goal**: Add real business logic, error handling, and data integrity rules that exist in production ERP systems.

**What you learn**: Business validation patterns, exception handling, data consistency, querying.

### L3-Task 1: Global Exception Handling

- Create `@ControllerAdvice` for centralized error handling
- Standard error response format:
  ```json
  {
    "timestamp": "2025-03-15T10:30:00",
    "status": 400,
    "error": "Validation Failed",
    "messages": [
      "Quantity must be greater than 0 on line 2",
      "Tour service is required on line 3"
    ],
    "path": "/api/purchase-orders"
  }
  ```
- Validation errors return field-level details
- Business rule violations return clear messages

**Acceptance criteria**: All errors follow the same format. No stack traces in responses.

### L3-Task 2: Referential Integrity Protection

- Cannot delete a Country that has Cities
- Cannot delete a Hotel that is used in any Tour or Voucher
- Cannot delete a Tour Service used in any Purchase Order or Sales Invoice
- Return clear error: `"Cannot delete Country 'Egypt': 5 cities reference it"`

**Acceptance criteria**: All delete operations check for references. Error messages name the blocking entities.

### L3-Task 3: Tour Status Workflow Engine

- Implement a proper state machine for Tour/TourProgram status
- Validate transitions (only allowed paths)
- When moving to CANCELED: warn about open Purchase Orders
- When moving to FINISHED: validate all accommodation situations are OK
- Return the list of allowed next statuses: `GET /api/tours/{id}/allowed-transitions`

**Acceptance criteria**: Invalid transitions return 400 with explanation. Allowed transitions endpoint works.

### L3-Task 4: Document Number Generation

- Auto-generate document numbers in format: `PO-2025-0001`, `SI-2025-0001`
- Numbers are sequential per document type per year
- Thread-safe (no duplicate numbers under concurrent creation)

**Acceptance criteria**: Numbers auto-increment. No duplicates even with concurrent requests.

### L3-Task 5: Tour Creation from Program (Full Business Logic)

- When creating Tour from Program:
  - Copy all header fields
  - Copy all accommodation lines, calculate checkIn/checkOut from arrivalDate + cumulative nights
  - Copy all service lines, calculate actual dates from arrivalDate + dayNo
  - Copy all flight lines similarly
  - Set Tour status to INITIAL
- If Tour Program is in INITIAL status, reject with: "Cannot create tour from a program that hasn't started"

**Acceptance criteria**: Dates are correctly calculated. All lines are copied with correct data.

### L3-Task 6: Pagination and Search

- All list endpoints support pagination: `?page=0&size=20`
- All list endpoints support sorting: `?sort=name,asc`
- Add search endpoints:
  - `GET /api/hotels/search?name=marriott&cityId=1`
  - `GET /api/tours/search?status=IN_PROGRESS&fromDate=2025-01-01`

**Acceptance criteria**: Pagination metadata is included in response. Search filters work in combination.

---

## Level 4 — Production-Level Design

**Goal**: Apply clean architecture patterns, DTOs, mappers, and production-grade practices.

**What you learn**: DTO pattern, MapStruct, layered architecture, auditing, specifications.

### L4-Task 1: DTO Layer

- Create separate DTOs for each operation:
  ```
  dto/
  ├── request/
  │   ├── CreateCountryRequest
  │   ├── UpdateCountryRequest
  │   ├── CreateTourProgramRequest
  │   └── ...
  ├── response/
  │   ├── CountryResponse
  │   ├── TourProgramResponse
  │   ├── TourProgramSummaryResponse  (for lists — no lines)
  │   └── ...
  ```
- Never expose JPA entities directly in controllers
- Summary DTOs for list endpoints (no nested collections)
- Full DTOs for single-entity endpoints (with collections)

**Acceptance criteria**: API request/response shapes are clean. JPA entities never leak to the API layer.

### L4-Task 2: MapStruct Mappers

- Add MapStruct for entity ↔ DTO mapping
- Custom mappings for calculated fields
- Handle nested collections (Tour Program → lines)

**Acceptance criteria**: No manual mapping code. MapStruct handles all conversions.

### L4-Task 3: Specification Pattern for Dynamic Queries

- Use Spring Data JPA Specifications for search endpoints
- Combine multiple filters dynamically:
  ```java
  Specification<Hotel> spec = HotelSpec.hasCountry(countryId)
      .and(HotelSpec.hasCity(cityId))
      .and(HotelSpec.nameContains(query));
  ```

**Acceptance criteria**: Filters compose cleanly. Adding a new filter requires only a new Specification method.

### L4-Task 4: Audit Fields

- Add to every entity: `createdAt`, `createdBy`, `updatedAt`, `updatedBy`
- Use `@EntityListeners(AuditingEntityListener.class)` with `@CreatedDate`, `@LastModifiedDate`
- Return audit fields in responses

**Acceptance criteria**: Audit fields auto-populate on create and update.

### L4-Task 5: Service Layer Refactoring

- Extract common document logic into `AbstractDocumentService<T>`
  - Common: calculateTotals(), validateLines(), generateDocNumber()
- Extract common master file logic into `AbstractMasterService<T>`
  - Common: checkReferences(), validateUniqueness()
- Tour-specific logic stays in `TourService`

**Acceptance criteria**: No duplicate logic across services. Adding a new document type reuses the abstract base.

### L4-Task 6: Integration Tests

- Write integration tests for each API endpoint using `@SpringBootTest` + `MockMvc`
- Test scenarios:
  - Happy path CRUD for each entity
  - Validation rejection (missing required fields)
  - Referential integrity (cannot delete referenced entity)
  - Status transitions (allowed and rejected)
  - Tour creation from program (line copying)
  - Purchase Order total calculation
  - Sales Invoice payment validation
- Use `@Transactional` for test isolation

**Acceptance criteria**: All scenarios pass. Tests run in < 30 seconds total.

---

# Step 5 — Task-by-Task Implementation Guide

## How This Works

1. Start at **Level 1, Task 1**
2. Implement the task
3. Send me your code for review
4. I will review strictly — pointing out:
   - Design issues
   - Business logic mistakes
   - Bad practices
   - Missing edge cases
5. Fix issues until approved
6. Move to the next task

## Task Progression Map

```
LEVEL 1 — In-Memory Basics
  ├── L1-T1: Project Setup ✦ START HERE
  ├── L1-T2: Country CRUD
  ├── L1-T3: City with FK
  ├── L1-T4: Tour Program with Lines
  └── L1-T5: Purchase Order with Calculations

LEVEL 2 — Database + JPA
  ├── L2-T1: Database Layer Migration
  ├── L2-T2: Tour Program Collections
  ├── L2-T3: Tour (Template-to-Instance)
  ├── L2-T4: Purchase Order (Proper Calculations)
  ├── L2-T5: Sales Invoice with Payments
  └── L2-T6: Hotel Voucher

LEVEL 3 — Real Business Logic
  ├── L3-T1: Global Exception Handling
  ├── L3-T2: Referential Integrity
  ├── L3-T3: Status Workflow Engine
  ├── L3-T4: Document Number Generation
  ├── L3-T5: Tour from Program (Full Logic)
  └── L3-T6: Pagination and Search

LEVEL 4 — Production Quality
  ├── L4-T1: DTO Layer
  ├── L4-T2: MapStruct Mappers
  ├── L4-T3: Specification Pattern
  ├── L4-T4: Audit Fields
  ├── L4-T5: Service Layer Refactoring
  └── L4-T6: Integration Tests
```

## Starting Point

**Your first task is: Level 1, Task 1 — Project Setup**

Create a new Spring Boot project with:
- Java 21
- Spring Web + Spring Validation
- Maven build
- Package: `com.travel`
- A health endpoint: `GET /api/health` → returns `{"status": "UP"}`

When you're done, send me your code and I'll review it.

---

## Quick Reference Card

### Entity Types Cheat Sheet

| Type | Purpose | Has Lines? | Has Status? | Example |
|------|---------|-----------|-------------|---------|
| Simple Master | Lookup data | No | No | Country, City |
| Rich Master | Partner data | Yes (contacts) | No | Hotel |
| Template Master | Reusable plan | Yes (3 types) | Yes | Tour Program |
| Document | Transaction | Yes (3 types) | Yes | Tour |
| Operational Doc | Guest ticket | No | No | Hotel Voucher |
| Financial Doc | Money flow | Yes (lines + payments) | No | Purchase Order, Sales Invoice |

### Relationship Cheat Sheet

```
Country  ←──  City  ←──  Hotel
                          ↑
Tour Program  ───→  Tour ─┤
                          ↓
              Purchase Order ←── Sales Invoice
                          ↓
                    Hotel Voucher
```

### Status Transitions Cheat Sheet

```
INITIAL ──→ IN_PROGRESS ──→ FINISHED
   │
   └──────→ CANCELED
```

### Validation Priority

```
1. Required field checks        (is it present?)
2. Format/type checks          (is it valid?)
3. Business rule checks        (does it make sense?)
4. Cross-entity checks         (do references exist?)
5. Consistency checks          (do totals add up?)
```
