# Orders to FeetF1rst (`custom_shafts`)

This is the order. A partner, usually an orthopaedic shoemaker, asks FeetF1rst to make something for one customer: a test shoe, a last, a shaft, a sole, or the complete shoe. Every such request is one row in `custom_shafts`.

The name says "shafts", but the table holds every kind of production order, not only shafts. The kind is in `catagoary`.

The menu the partner picks from is the catalogue, described in `../01-catalogue/01-catalogue.md`. The order is the request that uses one menu item, or the partner's own model, for one customer.

This chapter is split into files:

| File | Subject |
| --- | --- |
| `02-custom-shafts.md` | This file. What an order is, every column, every status, every link to another table |
| `02.1-custom-shafts-shaft-and-complete-shoe.md` | Ordering a shaft or complete shoe (`Massschafterstellung`, `Komplettfertigung`), drafts, sending a draft |
| `02.2-custom-shafts-other-kinds.md` | Test shoe, sample, last, insole, sole orders, and the feedback after a fitting |
| `02.3-custom-shafts-status-and-payment.md` | What happens after the order exists: status, payment, cancel, trash, admin edits, lists |
| `02.4-custom-shafts-shipping.md` | Getting lasts to FeetF1rst and the finished product back to the partner |
| `02.5-custom-shafts-problems.md` | Problems that the code really has, and the fix each one needs |
| `02.6-custom-shafts-chat.md` | The conversation between FeetF1rst and the shop about one order |
| `02.7-custom-shafts-invoices-and-emails.md` | Producer sheets, collected invoices, every e-mail, copies in the customer's file area |
| `02.8-custom-shafts-dates-and-counts.md` | The admin's count cards, "late", working days, holidays and re-planning |
| `02.9-custom-shafts-manufacturer.md` | Giving an order to an outside producer, and the producer's screens |

The table is in `prisma/admin-order.schma.prisma`, lines 92–390. The related tables are in the same file, after line 391.

## The kinds of order

| What the partner orders | `catagoary` | In plain words |
| --- | --- | --- |
| Test shoe | `Halbprobenerstellung` | FeetF1rst builds a fitting shoe from the partner's 3D scan. The customer tries it on. |
| Sample | `Probenerstellung` | A sample order on an existing shoe order. Only one active per shoe order. |
| Last | `Leistenerstellung` | FeetF1rst mills the last (the foot form). Often follows a test shoe. Price is `0`. |
| Insole bedding | `Bettungserstellung` | The footbed. Linked to the test shoe of the same shoe order when there is one. |
| Shaft | `Massschafterstellung` | The upper part of the shoe, from a catalogue model or the partner's own model. |
| Complete shoe | `Komplettfertigung` | Shaft plus sole, built together. |
| Sole only | `Bodenkonstruktion` | The bottom of the shoe without a shaft. |
| — | `row` | Exists in the list of values. No code creates it. Two list filters accept it. |

## The usual life of an order

1. The partner prepares the order on the Partner Dashboard. They pick the customer, the model, the materials, and upload the 3D files of the lasts if they have them.
2. They either send it now, or save it as a draft.
3. A sent order becomes `active`, gets an order number, and is added to the partner's open amount on the balance page.
4. FeetF1rst's team works on it in the admin dashboard (`feetf1rst-landing-page`). The status moves from "Bestellung eingegangen" through production to "Qualitätskontrolle".
5. If the partner had to post physical lasts, they arrive and the order is marked as "Leisten eingegangen". Production time is counted from that day.
6. After quality control, FeetF1rst ships to the partner. Status becomes "Versandt".
7. When the parcel is delivered, or the admin closes it, status becomes "Ausgeführt".
8. The partner pays. The admin marks the order `Paid`, and the open amount goes down.

An order can be cancelled. A cancelled order goes to the admin's Trash page, from where it can be restored or removed for good.

## The two dashboards

| Who | App | Where the orders are |
| --- | --- | --- |
| Partner | Partner Dashboard | Created on the category pages (`custom-shafts/details/[id]`, `custom-shafts/product-order/[id]`, `bodenkonstruktion`, `leistenkonfigurator`, shoe-order steps). Listed on `balance-dashboard` in the "Aktivität" table. |
| Admin | `feetf1rst-landing-page` | `partnerbestellungen` (main list and detail), `manage-order` (older list), `versand`, `finanzkontrolle`, `trash` |

The partner list does not read `/custom_shafts/get`. It reads `/v2/admin-order-transitions/get-all-transitions-tree`, which lists the billing lines of the partner and the orders under them. The admin list reads `/v3/admin-order/get-all-orders`. The detail page in both apps reads `/massschuhe-order/admin-order/get/:id`.

## Every column

The tables below follow the order of the schema. "Who writes it" names the step that sets it in practice.

### Identity and customer

| What it means | Column | What is saved | Who writes it |
| --- | --- | --- | --- |
| The real identity | `id` | A cuid | Database |
| Order number the partner sees | `orderNumber` | Text with digits. Starts at `10000` for each partner, then highest number + 1 for that partner. | Every create, except the manual last order (see `02.2`) |
| Customer number when there is no customer record | `other_customer_number` | Text | Create, from the body or from the customer's `customerNumber` |
| Customer name when there is no customer record | `other_customer_name` | Text | Create |
| Customer record | `customerId` | Customer id | Create, when the partner picked a saved customer |
| Display name | `customerName` | Text | Draft save (`update-admin-order-after-draft`) |

A sent order must have either a customer record or a typed customer name. A draft may have neither.

### Money

| What it means | Column | What is saved | Who writes it |
| --- | --- | --- | --- |
| Price the partner pays | `totalPrice` | Number. The partner screen computes it, including extras and courier. The server does not recompute it. | Create, draft save |
| Discount | `discount` | `0` to `100`. Default `0`. | Admin bulk edit |
| Paid or not | `payment_status` | `Unpaid` (default), `Paid`, `Unclear` | Admin |
| When it was marked paid | `paid_at` | Date, or empty when not paid | Admin payment change |
| FeetF1rst's own production cost | `production_cost` | Number. On create it is copied from the setting for that category (`custom_shafts_delivery_dates.production_cost`). | Create, admin edit |
| Production reference | `production_number` | The partner's `prefix` plus the last 3 digits of the order number | Create (shaft, sole, insole), admin edit, manufacturer assignment |
| Whether FeetF1rst paid its producer | `production_payment_status` | `Unpaid` (default), `Paid`, `Unclear` | Admin |
| Invoice the partner sees | `invoice` | File address (PDF the partner screen generates) | Create |
| Second invoice | `invoice2` | File address | Create |
| Link to a collected invoice | `custom_shafts_invoice_id` | Points at `custom_shafts_invoice` | Invoice tools in `custom_shafts/treack` |
| Which month's bill | `billing_month` | Always the 1st of a month | See "Billing month" below |

### What is being built

| What it means | Column | What is saved | Who writes it |
| --- | --- | --- | --- |
| Kind of order | `catagoary` | One of the kinds above | Create |
| 3D file of the right last | `image3d_1` | File address | Create, draft save |
| 3D file of the left last | `image3d_2` | File address | Create, draft save |
| Partner's own model instead of a catalogue one | `isCustomeModels` | Yes/no | Create; then a `custom_models` row holds the model |
| Test shoe details | `Halbprobenerstellung_json` | JSON from the test-shoe screen | Test shoe create |
| Test shoe PDF | `Halbprobenerstellung_pdf` | File address | Test shoe create |
| Fitting checklist | `checkliste_halbprobe` | JSON | Fitting step |
| Insole details | `bettungserstellung_json` | JSON | Insole create |
| Shaft specification | `Massschafterstellung_json1` | One JSON object with the shaft's leather, lining, height, padding, seams, extras, and a `meta` part with the catalogue id and display names. Built in the partner's browser by `prepareMassschafterstellungJson1`. | Shaft create, draft save |
| Leather layout picture | `ledertyp_image` | File address | Shaft create |
| Zipper picture | `zipper_image` | File address | Shaft create |
| Painted leather areas | `paintImage` | File address. When the partner uses "Ausführung wie im Katalog", the screen puts the catalogue's `ledertyp_image` address here. | Shaft create |
| Sending details of the shaft step | `versenden` | JSON | Shaft create |
| Sole specification for a complete shoe | `Massschafterstellung_json2` | JSON | Complete-shoe create |
| Sole specification for a sole-only order | `bodenkonstruktion_json` | JSON | Sole create |
| Sole picture | `staticImage` | File address | Sole create |
| Sole 3D file | `threeDFile` | File address. The single-order view returns it only for `Komplettfertigung` and `Bodenkonstruktion`. | Sole create |
| Partner's own sole design | `isCustomBodenkonstruktion` | Yes/no. Always `true` on a sole-only order. | Sole create |
| Which catalogue model | `maßschaftKollektionId` | Catalogue shoe id. The request field is spelled `mabschaftKollektionId`. | Shaft create |
| Last type of a test shoe | `last_type` | `gross_last` (default): footbed allowance is inside the last. `net_last`: no allowance. | Test shoe create |
| Who models the last | `modulation_by` | `feetf1rst` (default) or `internal` (the partner) | Test shoe create |
| Only a testing shoe | `testing_shoe` | Yes/no, default no | Test shoe create |
| Card colour in the admin list | `bg_colore` | Text | Admin (`/v3/admin-order/manage/update-bg-colore`) |

### Status

| What it means | Column | Values | Who writes it |
| --- | --- | --- | --- |
| Production step | `status` | See "Production status" below. Default `Bestellung_eingegangen`. | Admin, crons |
| Did the physical lasts arrive | `last_status` | `Unangekommen` (default), `Angekommen`, `fehlen` | Admin |
| Is the order live | `order_status` | `active` (default), `canceled`, `completed`, `neutral` | Create, send draft, cancel, restore |
| Draft or normal | `order_type` | `normal` (default), `draft` | Create, send draft |
| Original draft date | `draft_created_at` | The draft's `createdAt`, kept when the draft is sent | Send draft |
| Urgent | `urgency` | `normal` (default), `urgent` | Insole create, admin bulk edit |
| Marked "Leistenerstellung" in the schema comment | `isFeedback` | Yes/no, default no | No code writes it. Every order keeps `false`. |
| Too old to be the current one | `isExpaired` | Yes/no, default no. Used to ignore old test shoes and samples. | Feedback flow |
| Has really gone to production | `handover` | `not_prepared` (default), `sheet_generated`, `sent_to_production`. Only `sent_to_production` means it was really sent. | Admin, `/v3/admin-order/production-handover/:orderId` |
| When it was sent to production | `handover_sent_at` | Date. Set only for `sent_to_production`, cleared otherwise. | Same |
| Which admin changed handover | `handover_by` | The admin's user id, always from the login | Same |

### Dates and production time

| What it means | Column | What is saved |
| --- | --- | --- |
| When the order was placed | `createdAt` | Set by the database. When a draft is sent, it is reset to that moment. |
| Last change | `updatedAt` | Set by the database |
| When the production clock started | `production_start_date` | See below |
| How long production takes | `delivery_length` | Number of calendar days from `production_start_date` to `deliveryDate` |
| Planned delivery day | `deliveryDate` | Date |
| When the partner will post the sample or lasts | `sample_sending_date` | Date. Empty when the 3D files were uploaded. |
| When FeetF1rst shipped | `shipping_time` | Set when status becomes `Versandt` |
| When the order was finished | `delivered_at` | Set when status becomes `Ausgeführt` |

How the production clock works:

- Every kind of order has a standard number of working days in `custom_shafts_delivery_dates`, one row per kind. The admin sets them (`POST /custom_shafts/delivery-dates/manage`); see `02.8-custom-shafts-dates-and-counts.md`.
- When the clock starts, the server counts that many working days forward. It skips Saturdays, Sundays, and every day in `holyday_and_vacation`. That day is `deliveryDate`.
- `delivery_length` is then saved as the calendar days between the start and that day, so that start + `delivery_length` lands on `deliveryDate`. The partner's list and the admin's "late orders" counts rely on this. Two paths break the rule: the test shoe and the automatic last order after "no issue" feedback save the working days as they are, with no `deliveryDate` (problem 11 in `02.5-custom-shafts-problems.md`).
- When the admin adds or removes a holiday, open orders are re-planned.

When the clock starts depends on the kind:

| Kind | Clock starts |
| --- | --- |
| Shaft, complete shoe | When both 3D last files are on the order, or when the order is marked as lasts received |
| Test shoe | When 3D files are on the order, or when the order is marked as lasts received |
| Sample, last | When the order is created |
| Sole only | Not at create. It starts later, when the lasts are received. |
| Insole | Not set |

### Billing month

Every sent order is put in one month's bill. The rule is in `module/v1/admin_order/billing_month.util.ts`:

1. Take the production start, or today when there is none.
2. Add `delivery_length` days, or 14 days when there is none.
3. If that end day is the 25th or later (Berlin time), the order goes into next month's bill. Otherwise into this month's.

A draft gets no billing month. It gets one when it is sent. The admin can move an order to another month on the billing page (`/v3/admin-order/billing-month/update-billing-month`).

### Shipping columns

| What it means | Column | What is saved |
| --- | --- | --- |
| How the partner sends lasts to FeetF1rst | `shipping_type1` | `Express`, `Standard`, or a FedEx service name |
| What that costs the partner | `shipping_cost1` | Number. From the global price table `shipping_pricing` when the screen does not send one. |
| How FeetF1rst sends the product back | `shipping_type2` | Same values |
| What that costs | `shipping_cost2` | Number |
| Who sends the lasts | `lest_sending_type` | `feetf1rst_shipping` (FeetF1rst sends a courier) or `self_shipping` (the partner posts them) |
| Pickup address and delivery details | `delivery_info` | JSON: postal code, street, city, state, country, and more |
| FedEx label for the pickup | `pickup_label` | File address. The admin can also upload one by hand. |
| When that label was made | `label_created_at` | Date |
| FedEx tracking number | `fedex_tracking_number` | Text. Also filled from incoming e-mails by the mail center. |
| Delivery note number | `delivery_note_number` | Text such as `1290` |
| Link to the delivery note | `shipping_delivery_note_id` | Points at `shipping_delivery_note` |
| Link to the shipment row | `custom_shafts_shipping_id` | Points at `custom_shafts_shipping`. Many orders can point at one shipment. |
| Copy of the shipment before grouping | `custom_shafts_shipping_backup` | JSON, put back when the order leaves a group |
| Same-day delivery group | `package_group_id` | Points at `package_group` |
| Old shipment link | `custom_shafts_shipping` (relation) | Shipment rows that point at this order with their own `custom_shafts_id`. Kept for older rows. |

`02.4-custom-shafts-shipping.md` explains how these work together.

### Chat columns

The admin and the partner chat about one order. The messages are in `custom_shafts_chat`; `02.6-custom-shafts-chat.md` explains the chat. The order keeps a summary so lists do not have to read every message:

| Column | What is saved |
| --- | --- |
| `chat_last_at`, `chat_last_id`, `chat_last_message`, `chat_last_sender_role`, `chat_last_message_type` | The last message |
| `chat_admin_unread`, `chat_partner_unread` | Unread counts, default `0` |
| `chat_id` | Link to a `chat` row |

## Production status

| Saved value | Label in the admin dashboard | What the code does with it |
| --- | --- | --- |
| `Bestellung_eingegangen` | Bestellung eingegangen | Default. A partner may cancel only in this status or in `Neu`. |
| `Neu` | Shown as Bestellung eingegangen | Same as above |
| `In_Produktiony` | In Produktion | The saved value has a `y` at the end. The admin bulk screen turns `In_Produktion` into `In_Produktiony` before sending. |
| `Warten_auf_Anprobe` | Warten auf Anprobe | The test shoe is ready for fitting. Starts the fitting step on the shoe order and tells the partner. |
| `Leisten_wird_gefräst` | Leisten wird gefräst | The last is being milled. A test shoe waiting for fitting moves here when a last order, a shaft order, or a sole order is placed on it, or when the fitting feedback says "minor issue" or "no issue". |
| `lasts_received` | Leisten eingegangen | The partner's physical lasts arrived. Also sets `last_status` to `Angekommen` and starts the production clock. |
| `Qualitätskontrolle` | Qualitätskontrolle | For shafts and complete shoes, creates or joins a shipment back to the partner. |
| `Versandt` | Versandt | Sets `shipping_time`. Tells the partner. Completes shoe-order steps. |
| `Ausgeführt` | Ausgeführt | Sets `delivered_at`. |
| `problem` | — | Accepted by the list filter |
| `Zu_Produzent_abgeschickt`, `In_Bearbeitung`, `Zu_Kunde_abgeschickt`, `Bei_uns_angekommen`, `Beim_Kunden_angekommen` | — | Older values still in the list. The monthly revenue figure (`total-price-resio`) still counts `Beim_Kunden_angekommen`. |

Every status change made on the normal status page is written to `custom_shafts_status_history`, with the old and new value.

## Links to other tables

### Tables the order points at

| Column | Points at | If that row is deleted |
| --- | --- | --- |
| `partnerId` | The partner (`users`) | The order stays. `partnerId` becomes empty. (Deleting a partner through the partner-delete service removes their orders first; see below.) |
| `customerId` | The customer | The order stays. `customerId` becomes empty. |
| `maßschaftKollektionId` | The catalogue shoe | The catalogue shoe cannot be deleted while an order points at it. |
| `massschuhe_order_id` | Older custom-shoe order (`massschuhe_order`) | Empty |
| `shoe_order_id` | The partner's shoe order (`shoe_order`) | Empty |
| `parent_custom_shafts_id` | Another order: the test shoe this one follows | Empty |
| `custom_shafts_feedback_id` | The feedback this order is pinned to | Empty |
| `custom_shafts_invoice_id` | Collected invoice | Empty |
| `package_group_id` | Same-day group | Empty |
| `custom_shafts_shipping_id` | Shipment row | Empty |
| `shipping_delivery_note_id` | Delivery note | Empty |
| `clinica_assistant_order_components_id` | The component in the partner's clinical care plan that sent this order | Empty |
| `chat_id` | Chat | Empty |
| `handover_by` | The admin who changed handover | Empty |

### Tables that point at the order

| Table | What it is | If the order is deleted |
| --- | --- | --- |
| `custom_shafts` (children) | Later orders that follow this one, through `parent_custom_shafts_id` | Delete removes the children too (the delete code walks down the tree first) |
| `admin_order_transitions` | The billing line on the partner's balance page | Delete removes them and lowers the partner's open amount |
| `custom_models` | The partner's own model for this order | Deleted |
| `custom_shafts_document` | One row of PDFs: producer sheet, second producer sheet, quality-check invoice | Link becomes empty |
| `custom_shafts_status_history` | Every status, urgency, payment, and order-status change | Link becomes empty |
| `custom_shafts_feedback` | Fitting feedback | Delete removes the feedback and its files |
| `feedback_images` | Pictures uploaded before a feedback is saved | Link becomes empty |
| `fixing_last` | Corrected 3D lasts after a minor issue, when the partner models internally | Link becomes empty |
| `custom_shafts_shipping` | Old-style shipment rows | Link becomes empty |
| `CourierContact` | Earlier courier contacts and addresses | Link becomes empty |
| `delivery_date_change` | Every change of the delivery date, with reason and who was told | Link becomes empty |
| `manufacturer_order` | Which producer was given the order | Deleted with the order |
| `custom_shafts_chat`, `custom_shafts_chat_employee` | Chat messages and which employees take part | They stay, with an empty order link (see `02.6-custom-shafts-chat.md`) |

### Small tables used by orders

| Table | What it holds |
| --- | --- |
| `custom_shafts_delivery_dates` | One row per kind of order: standard working days (`day`) and FeetF1rst's production cost |
| `holyday_and_vacation` | Days that do not count as working days. One holiday and one vacation are allowed on the same day. |
| `shipping_pricing` | What the partner pays for Express and Standard shipping |
| `package_group` | Orders of one partner that should ship together, because they are due the same day |
| `shipping_address` | The partner's saved pickup addresses, newest first |
| `damian_count` | One global number the admin sets in the "Date Selection" box of the manage-order page. Not tied to an order. |

## What happens when a partner is deleted

`utils/partner-delete.service.ts` removes the partner's orders before it removes the partner. In order: the parent links of child orders and the pinned feedback are cleared, feedback and its files are deleted, then shipments, status history, delivery-date changes, courier contacts, custom models, every billing line of that partner, documents, and the orders themselves. Orphaned invoices and courier rows are cleaned after that. Then the user is deleted.
