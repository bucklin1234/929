# 929 Multi-Cuisine Food Cafe — Requirements Specification

**Version:** 0.1 (Prototype Scope)
**Last Updated:** 2026-05-15

---

## 1. Concept Overview

A multi-cuisine food cafe operating 9 AM – 9 PM ("9 to 9"), with 9 cuisine stations arranged in a semicircular layout. Customers browse each station, add items to a shared cart, and pay at a single central checkout. The web application mirrors this physical store experience — it is the launch vehicle before the physical store opens.

---

## 2. Canonical Data Model

### 2.1 Cuisine Station
| Field         | Type     | Notes                                      |
|---------------|----------|--------------------------------------------|
| id            | integer  | 1–9, determines arc position left to right |
| name          | string   | e.g., "Biryani", "Juice Centre"            |
| position      | integer  | Arc order (1 = leftmost, 9 = rightmost)    |
| is_active     | boolean  | Whether station is open today              |

**Ordered Station List (left → right on arc):**

| Position | Station      |
|----------|--------------|
| 1        | Coffee       |
| 2        | Sushi        |
| 3        | Boba         |
| 4        | Salad        |
| 5        | Ice Cream    |
| 6        | Biryani      |
| 7        | Pizza        |
| 8        | Juice Centre |
| 9        | Cake         |

### 2.2 Menu Item
| Field          | Type     | Notes                                        |
|----------------|----------|----------------------------------------------|
| id             | uuid     |                                              |
| station_id     | integer  | FK → Cuisine Station                         |
| name           | string   | Item name                                    |
| description    | string   | Short description                            |
| price          | decimal  | In local currency                            |
| image_url      | string   | Optional item image                          |
| is_available   | boolean  | Can be toggled off mid-day if sold out       |
| rotation_slot  | integer  | Which day-slot in the rotation cycle this belongs to |

### 2.3 Daily Menu (Rotation)
| Field       | Type    | Notes                                              |
|-------------|---------|----------------------------------------------------|
| id          | uuid    |                                                    |
| station_id  | integer | FK → Cuisine Station                               |
| cycle_day   | integer | Day number in the rotation cycle (e.g., Day 1–7)  |
| item_ids    | array   | List of Menu Item IDs active on this cycle day     |

**Rotation rules:**
- Menu changes overnight (midnight cutover)
- Each station has a predefined rotation cycle (configured in advance by admin)
- The active menu for today = items whose `cycle_day` matches today's position in the cycle

### 2.4 Cart
| Field        | Type    | Notes                             |
|--------------|---------|-----------------------------------|
| session_id   | uuid    | Anonymous session identifier      |
| items        | array   | List of CartItems                 |
| total_cost   | decimal | Sum of (item.price × quantity)    |

### 2.5 Cart Item
| Field      | Type     | Notes                        |
|------------|----------|------------------------------|
| item_id    | uuid     | FK → Menu Item               |
| station_id | integer  | Denormalized for display     |
| name       | string   | Snapshot at time of adding   |
| price      | decimal  | Snapshot at time of adding   |
| quantity   | integer  | Min 1                        |

---

## 3. Operating Rules

| Rule                     | Detail                                              |
|--------------------------|-----------------------------------------------------|
| Operating hours          | 9:00 AM – 9:00 PM daily                            |
| Outside hours            | Website shows "We're closed" state, ordering disabled |
| Menu cadence             | Changes overnight via predefined rotation cycle     |
| Ordering model           | Customer collects from stations, pays at one point  |
| Consumption model        | Takeout only (no dine-in)                           |
| Payment point            | Single central checkout (all cuisines combined)     |

---

## 4. Web Application — Feature Requirements

### 4.1 Pages

| Page         | Purpose                                            |
|--------------|----------------------------------------------------|
| Home / Store | The semicircular station map — main experience     |
| Checkout     | Cart review and order placement                    |
| Contacts     | Static info: phone, email, address, hours          |

### 4.2 Home / Store View

**Layout:**
- Render a **semicircular arc** with 9 equal radiating segments
- Each segment is a clickable cuisine station
- Station name is labelled on/near each segment
- A **central counter** element sits inside the arc (visual only)
- **Entrance** indicator at the flat bottom edge
- **Checkout** button anchored at bottom-right

**Interaction — Segment Selection:**
- User clicks a segment
- That segment **expands / pops out** to reveal its item list
- All other 8 segments **gray out** (visually dimmed, non-interactive while one is open)
- User can close the expanded segment to return to full arc view

**Interaction — Item Selection (within an expanded segment):**
- Items for today's menu are listed (name, description, price, optional image)
- Each item has an **Add to Cart** button
- User can add multiple items from the same station before closing
- Quantity can be adjusted (+ / −) before or from the cart

**Cart Behaviour:**
- Persistent across station browsing (does not reset when a segment closes)
- Floating cart indicator (item count + running total) always visible
- Cart panel accessible at any point without navigating away
- Cart displays: item name, station, quantity, line total
- Cart displays: **grand total**
- Cart has a **Proceed to Checkout** action

### 4.3 Checkout View

- Summary of all cart items grouped by station
- Grand total displayed prominently
- Order confirmation action (payment integration: deferred to later phase)

### 4.4 Contacts Page

- Static content: store name, address, phone, email
- Operating hours: 9 AM – 9 PM
- No contact form (deferred)

### 4.5 Operating Hours Gate

- If current time is outside 9 AM – 9 PM, the store view shows a closed state
- Ordering is disabled (Add to Cart is hidden/disabled)
- Browsing the menu is still permitted (read-only mode)

---

## 5. Admin / Operations (Deferred — Not in Prototype)

The following are captured for future phases:

- Admin login and dashboard
- Menu item CRUD per station
- Rotation cycle configuration
- Per-item availability toggle (sold out)
- Order management and fulfilment tracking
- Payment gateway integration
- Customer accounts and order history
- Ready time / queue estimation

---

## 6. Open Questions (To Be Resolved)

| # | Question                                                                 |
|---|--------------------------------------------------------------------------|
| A | What are the specific cuisine details for each station (full item list)? |
| B | What is the rotation cycle length? (7-day? 14-day?)                     |
| C | What currency / locale does pricing use?                                 |
| D | Is there a store name / brand name for the cafe?                         |
| E | Payment at checkout: online or pay-at-counter?                           |
| F | Do customers need an account, or is guest ordering sufficient?           |

---

## 7. Prototype Scope Summary

**In scope for prototype:**
- Semicircular 9-station UI with segment pop-out and gray-out behaviour
- Today's menu items per station (static/seeded data)
- Add to cart, quantity management, running total
- Checkout summary view
- Contacts page
- Operating hours gate

**Out of scope for prototype:**
- Live payment processing
- Admin panel
- User authentication
- Ready time / queue
- Real-time menu rotation (use seeded static data)
