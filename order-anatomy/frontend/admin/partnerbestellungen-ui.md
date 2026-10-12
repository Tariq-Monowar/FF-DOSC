# Partnerbestellungen

## Contents

| Section | Topic                                                                 |
| ------- | --------------------------------------------------------------------- |
| 1.1     | The right sidebar is open on the first visit                         |
| 1.2     | The filter bar is always two rows                                     |
| 1.3     | Bulk actions are four dropdowns in one row                            |
| 1.4     | The order row forces a sideways scroll                                |
| 2.1     | Every admin sees every partner chat                                   |
| 3.1     | Empty fields and the back link are a broken dash                     |
| 3.2     | Finanzen shows the production form                                    |
| 3.3     | Unknown keys and a column of Nein                                     |
| 3.4     | Produktion, Kommunikation, and Historie are placeholders              |
| 3.5     | The PDF and the category spelling                                     |

---

# Chapter 1: Order list

Route: `/dashboard/partnerbestellungen`

## 1.1 The right sidebar is open on the first visit

The page has a **Bestellungen** card on the right: insight cards, revenue, and AI search. It is sticky. On a large screen it is 320 px wide (`xl:w-80`). From `lg` it is up to 22 rem.

![Right sidebar: Umsatz card and Bestellungen insight list, open by default](https://feetf1rst.s3.eu-central-1.amazonaws.com/docs/order-anatomy/frontend/admin/partnerbestellungen-right-sidebar.png)

**Current behaviour**

The header switch is **Bestellungen ausblenden** / **Bestellungen anzeigen**. The choice is saved in `localStorage`. A missing value counts as open, so the first visit starts with the panel open and less room for the list. Below `lg` the page is a column, so the panel already sits under the list.

**Expected behaviour**

- The first visit starts with the panel closed. The list uses the full width.
- The user opens it with **Bestellungen anzeigen**.
- The last choice is saved. Open it once, and it is open again next time.
- Below `lg` the layout stays as it is. Closed by default only stops the panel from taking width on desktop.

**Code**

- Admin dashboard: `src/app/(dashbaord)/dashboard/partnerbestellungen/page.jsx:155` (storage key), `:248` (missing value means open), `:325` (`useState(true)`), `:510–512` (row from `lg`), `:616` (`xl:w-80`)
- Admin dashboard: `src/app/(dashbaord)/dashboard/_components/Partnerbestellungen/OrderHeader.jsx:54–63` (the switch labels)

## 1.2 The filter bar is always two rows

View tabs sit on top. Two filter rows sit under them. The bulk-action row (1.3) appears once orders are selected.

![Filters, view tabs, and bulk actions on Partnerbestellungen](https://feetf1rst.s3.eu-central-1.amazonaws.com/docs/order-anatomy/frontend/admin/partnerbestellungen-filters.png)

**Current behaviour**

These twelve controls are always visible:

1. Alle Status
2. Alle Kategorien
3. Zahlung (alle)
4. Auftrag (alle)
5. Sortierung
6. Search
7. Prod. ascending / descending
8. Ohne Prod.
9. Leisten (alle)
10. Alle Versandrichtungen
11. Versandt / Ausgeführt
12. Alle Partner

At `2xl` (1,536 px) the second row is forced onto one line (`2xl:flex-nowrap`). Below that it wraps: search, the Prod. buttons, and Ohne Prod. stay together, and partner and shipping share the next line. On a laptop with the sidebar open, the bar reads as a form.

**Expected behaviour**

One row stays visible: search, status, partner, sort, a **Filter** button, and **Reset** only when a filter is active. The **Filter** button shows a count once a hidden filter is set, for example `Filter · 2`.

The panel under that row holds category, payment, Auftrag, Leisten, shipping direction, Ohne Prod., Versandt + Ausgeführt, and sort by production number. Active filters sit as chips under the row. Each chip can be removed. Example: `Category: Halbprobenerstellung` · `Payment: Unbezahlt` · `Ohne Prod.`

On a narrow screen, search, status, partner, and sort stack. The panel is full width: two columns from `sm`, one column on a phone. No control in this bar has a minimum width that pushes the page sideways. Every filter that exists today stays available.

**Code**

- Admin dashboard: `src/app/(dashbaord)/dashboard/_components/Partnerbestellungen/PartnerbestellungenOrdersToolbar.jsx:433–549`, `:474` (`2xl:flex-nowrap`)

## 1.3 Bulk actions are four dropdowns in one row

**Current behaviour**

As soon as one order is selected, the row shows four dropdowns and three buttons:

- **Status wählen** and **Status ändern**
- **Leistenstatus wählen** and **Leistenstatus ändern**
- **Payment status wählen** and **Zahlungsstatus ändern**
- **Prioritaet waehlen** (this one applies on select, with no confirm button)

Each dropdown has a minimum width of 200 px. Together with “10 Bestellungen ausgewählt”, the row is wider than a normal window and breaks in the middle of the actions.

**Expected behaviour**

One row, only when something is selected: `10 selected` · `Clear selection` · **Change**.

**Change** opens one panel with four groups: order status, last status, payment status, and priority. Each group has its own dropdown and its own confirm button. The labels stay in German: “Zahlungsstatus wählen”, “Priorität wählen”.

**Code**

- Admin dashboard: `src/components/dashboard/ManageOrders/BulkOrderStatusBar.jsx:130–210`
- Admin dashboard: `src/app/(dashbaord)/dashboard/_components/Partnerbestellungen/PartnerbestellungenOrdersPanel.jsx:1067–1080`

## 1.4 The order row forces a sideways scroll

![Order rows with partner, product, finance, status, and actions](https://feetf1rst.s3.eu-central-1.amazonaws.com/docs/order-anatomy/frontend/admin/partnerbestellungen-orders.png)

**Current behaviour**

Each row is a fixed 6-column grid:

| Column | Content | Width |
| ------ | ------- | ----- |
| 1 | Select, expand, order number | 11 rem |
| 2 | Partner, email, country | 13 rem |
| 3 | Product, category, customer | 11–13 rem |
| 4 | Amount, margin, payment | 10.5–12 rem |
| 5 | Status, order date, delivery date | from 11.5 rem |
| 6 | Handover, warning, three action icons | from 14.5 rem |

Below 1,891 px the list forces a minimum width of 80 rem (1,280 px) and scrolls sideways (`max-[1890px]:min-w-[80rem]`). A 14-inch laptop, a split window, or the open sidebar produces a second scrollbar.

Row height follows the partner name. A short name stays on one line. “Orthopädieschuhtechnik GmbH & Co. KG” wraps onto three lines, so status, amount, and the action icons no longer line up.

**Expected behaviour**

The same information, and no column wider than the window.

From a 1,280 px window, with the sidebar closed, one row:

- Select, expand
- Order number and type (Order / Unterbestellung)
- Partner name, one line, overflow as `…`, email in a tooltip
- Status badge
- Delivery date
- Amount
- Actions (handover, warning, the three icons)

A second, smaller line in the same card, in muted text: product, category, customer, margin, payment status, order date, country. Long names are truncated. The expanded detail stays the place for the email, the full partner name, and the remaining fields.

Below 1,280 px, and whenever the right sidebar is open, the row becomes a card at 100% of the available width:

- Header: select, order number, status
- Middle: partner, product, customer
- Footer: amount, delivery date, actions

Child orders stay indented, with the colored left edge of the shipping group. The indent stays `ml-2` / `sm:ml-4`. The “select all on this page” bar and the total count stay above the list, on one line, and wrap on a phone.

**Code**

- Admin dashboard: `src/app/(dashbaord)/dashboard/_components/Partnerbestellungen/PartnerbestellungenOrderRowCard.jsx:626` (indent), `:658` (the six columns)
- Admin dashboard: `src/app/(dashbaord)/dashboard/_components/Partnerbestellungen/PartnerbestellungenOrdersPanel.jsx:1139–1140` (`min-w-[80rem]`)

---

# Chapter 2: Partner support

Route: `/dashboard/partnerbestellungen-nachrichten`

## 2.1 Every admin sees every partner chat

This inbox is the support channel. A partner who waits here, or who gets two different answers, will not stay.

![Support inbox: one shared list of every partner order chat](https://feetf1rst.s3.eu-central-1.amazonaws.com/docs/order-anatomy/frontend/admin/partnerbestellungen-support-inbox.png)

**Current behaviour**

Every person under the admin account opens the same inbox. The list is one row per order. The only filters are **Alle** and **Ungelesen**. Search covers messages, customers, and orders. The list request sends `limit`, `cursor`, `unread`, and `search`. It does not send a staff member.

An @mention writes a Mitarbeiter name into that one message. It does not assign the conversation, and it does not hide it from anyone else. The thread badge says **Partner ↔ Admin**. There is no owner on the partner and no owner on the thread.

That holds while the team is small. It does not hold once many partners write at the same time:

- A staff member scrolls partners they do not look after.
- Two people answer the same thread, or nobody answers because each person assumes the other will.
- The unread badge is one number for the whole team.
- A dedicated person cannot clear only their partners.

**Expected behaviour**

Assignment sits on the partner. One partner has one primary staff member and one backup. Every order chat of that partner follows that assignment.

**Admin**

- Sees every conversation.
- On the partner, sets the primary staff member and the backup.
- Can move a partner to someone else. Open chats move with the partner. The history stays in the thread.
- Has an **Unassigned** view, so a new partner is not left without an owner.

**Staff**

- The inbox opens on **Mine**: only the partners assigned to them.
- They do not see another staff member’s partner chats.
- Their unread badge counts only their partners.
- An @mention can pull them into one thread when the owner needs help. That opens that thread only.

**While several people are online**

- Each person works their own list.
- The thread header shows the owner’s name.
- If a second person opens a chat the owner already has open, the header shows who is in it.
- The backup receives the partner when the primary is marked away, or when a partner message has had no reply for a set time (start with 30 minutes). The backup keeps it until the primary takes it back.

The list groups by partner, with that partner’s open orders under the partner. A partner row shows the assigned staff member, the oldest message still waiting for a reply, and the unread count. Filters for staff: **Mine**, **Unread**, **Waiting on us**, **Waiting on partner**. Admin also gets **All staff**, **Unassigned**, and a filter by staff member.

A reply from FeetF1rst is sent under the assigned staff member’s name. An internal note stays inside the team. The partner does not see it. Each thread has a status: **Open**, **Waiting on partner**, **Done**. Done leaves the working list and keeps the history.

The chat itself stays on the order. The link to the Auftrag, file attachments, and search stay. Admin can still open any chat.

**Code**

- Admin dashboard: `src/app/(dashbaord)/dashboard/_components/Partnerbestellungen/PartnerbestellungenNachrichtenInbox.jsx:68–70` (Alle, Ungelesen), `:336–341` (the list request), `:770` (the @mention inserts the name), `:1208` (**Partner ↔ Admin**)

---

# Chapter 3: Order detail

Route: `/dashboard/partnerbestellungen/[id]`

Example: Bestellung #10333, category Halbprobenerstellung. The bar is Übersicht, Produktion, Finanzen, Kommunikation, Rückmeldung, Historie, and the button **Lieferung aktualisieren**. AI-PDF and Versand appear only for some categories (3.5).

The form on the partner side changes. A field is renamed, a new yes/no is added, an old key is dropped. The detail page and the PDF each have their own copy of those fields. After a change, one of the two misses it.

## 3.1 Empty fields and the back link are a broken dash

![Übersicht for #10333: empty Kunde and Modell render as a broken dash](https://feetf1rst.s3.eu-central-1.amazonaws.com/docs/order-anatomy/frontend/admin/partnerbestellungen-detail-uebersicht.png)

**Current behaviour**

An empty value is the string `â€”`, stored where an em dash should be. Kunde and Modell on #10333 therefore read as broken text. The same string is the fallback in `DataField`. The back link on this page is the same kind of damage: `ZurÃ¼ck zu Partnerbestellungen` in `ManageOrderDetailTabbedView.jsx:158`. `PartnerOrderDetailHeader.jsx:15` already has the correct “Zurück zu Partnerbestellungen”, and this page does not use it.

Modell also misses the catalogue name. This file reads `order.maÃŸschaft_kollektion`. The rest of the admin app reads `order.maßschaft_kollektion`. The two keys are not the same, so a name that is on the order still falls through to `â€”`.

**Expected behaviour**

An empty text value is **Nicht angegeben**. The back link reads **Zurück zu Partnerbestellungen**. Modell reads `maßschaft_kollektion.name`. These files are saved as UTF-8.

**Code**

- Admin dashboard: `src/app/(dashbaord)/dashboard/_components/Partnerbestellungen/Detail/partnerOrderDetailOrderUtils.js:246` (empty customer), `:295` (the wrong catalogue key), `:310` (empty model)
- Admin dashboard: `src/app/(dashbaord)/dashboard/_components/Partnerbestellungen/Detail/PartnerOrderOverviewTab.jsx:28`, `:127–128`
- Admin dashboard: `src/components/orderDetail/ManageOrderDetailTabbedView.jsx:158`

## 3.2 Finanzen shows the production form

![Finanzen tab showing production fields, raw key names, and a column of Nein](https://feetf1rst.s3.eu-central-1.amazonaws.com/docs/order-anatomy/frontend/admin/partnerbestellungen-detail-finanzen.png)

**Current behaviour**

The Finanzen tab mounts the full order-detail view. For Halbprobenerstellung that view prints “Halbprobenerstellung Details”: price, Kopfdaten, Leistentyp. Price, production cost, margin, and payment already sit in the Schnellübersicht on the right. The tab named Finanzen repeats the production form.

**Expected behaviour**

| Tab | Shows |
| --- | --- |
| Übersicht | Partner, customer, model, type, received date, status, manufacturer, delivery. Six Stammdaten fields. |
| Produktion | The category specification that is now under Finanzen. Grouped, German labels, Ja only (3.3). |
| Finanzen | Price, production cost, margin, payment, invoice download, pickup label. |
| Kommunikation | This order’s chat, opened on this shaft (3.4). |
| Rückmeldung | Stays for Halbprobenerstellung. |
| Historie | The status timeline the sidebar already loads. The sidebar keeps the short list. The tab shows the full list. |
| Lieferung aktualisieren | Stays a button. It writes the delivery date that Übersicht and Historie read. |

**Code**

- Admin dashboard: `src/app/(dashbaord)/dashboard/_components/Partnerbestellungen/Detail/PartnerOrderFinanzenDetails.jsx:47–54` (toolbar, then `ManageOrderDetailsView`)
- Admin dashboard: `src/components/dashboard/OrderDetails/OrderHalbprobenerstellungCard.jsx:167` (“Halbprobenerstellung Details”)

## 3.3 Unknown keys and a column of Nein

**Current behaviour**

Kopfdaten has a label map for `patient`, `auftraggeber`, and `leistenmaterial` only. Any other key is printed with `labelMap[key] || key`. On this order that is why `creationMode` and `modulation_by` appear as the raw names.

Leistentyp prints Ja or Nein for each boolean, including `false`. A form that is mostly Nein becomes a long grid. The selected option is hard to see.

**Expected behaviour**

One field catalog per category. Each entry has the JSON key, the German label, the group (Kopfdaten, Leistentyp, …), the type (text, yes/no, number, file), and the tab it belongs on. The detail screen and the PDF both read that catalog.

- The label always comes from the catalog. A key that is not in the catalog is listed once under **Neue Felder**, with the raw key and the value, until it has a German label.
- A yes/no group shows the **Ja** answers, and one line for how many options were not selected.
- A file (PDF, STL) is a button. The catalog marks which keys are files.

**Code**

- Admin dashboard: `src/components/dashboard/OrderDetails/OrderHalbprobenerstellungCard.jsx:16` (Ja / Nein), `:49` (`labelMap[key] || key`), `:58–62` (the three Kopfdaten labels)

## 3.4 Produktion, Kommunikation, and Historie are placeholders

**Current behaviour**

Those three tabs render one sentence: “Inhalt folgt. API-Anbindung kann hier später ergänzt werden.” The production data that belongs on Produktion is on Finanzen (3.2). Kommunikation does not open this order’s chat. Historie does not show the status timeline. The timeline component is only in the sidebar.

**Expected behaviour**

See the tab table in 3.2. Kommunikation opens this order’s chat. Historie and the sidebar timeline show the same status events.

**Code**

- Admin dashboard: `src/components/orderDetail/ManageOrderDetailTabbedView.jsx:246–247`
- Admin dashboard: `src/components/orderDetail/OrderDetailTabPlaceholder.jsx:25`
- Admin dashboard: `src/components/orderDetail/ManageOrderDetailTabbedView.jsx:251–260` (sidebar timeline)

## 3.5 The PDF and the category spelling

**Current behaviour**

The Bodenkonstruktion PDF walks `form_data_v2`, then older keys. The Halbprobenerstellung PDF is a stored file URL (`Halbprobenerstellung_pdf`, or `kopfdaten.pdfUrl`). It is not built from the fields on screen. Change a field in the partner form and three copies can diverge: the label map, the PDF path list, and the stored file.

Tab visibility uses the exact category string, and the strings do not agree:

| Check | Accepts |
| ----- | ------- |
| List filter | `Maßschafterstellung` is mapped to `Massschafterstellung` |
| AI-PDF and Versand | `Massschafterstellung` only |
| Hersteller block | `Massschafterstellung`, and the broken spelling `MaÃŸschafterstellung` |

The database enum is `Massschafterstellung`. A payload that carries the real `Maßschafterstellung` hides AI-PDF, Versand, and the Hersteller block. That looks like missing data.

**Expected behaviour**

The PDF is built from the same catalog as Produktion. If a required field has no value, the download names that label. For Halbprobenerstellung, the partner’s original file stays **Original vom Partner**. The generated sheet is a separate file, **Auftragszettel aus diesen Feldern**, and it uses the catalog.

Every category check uses one spelling, `Massschafterstellung`, through the same mapping the list filter already has.

**Code**

- Admin dashboard: `src/components/dashboard/OrderDetails/BodenkonstruktionPdfDownloadButton.jsx:469` (`form_data_v2`)
- Admin dashboard: `src/components/dashboard/OrderDetails/ManageOrderDetailsView.jsx:1090` (`Halbprobenerstellung_pdf`)
- Admin dashboard: `src/lib/orderDetail/orderDetailTabs.js:33–43` (AI-PDF, Versand), `:61–73` (which tabs render)
- Admin dashboard: `src/app/(dashbaord)/dashboard/_components/Partnerbestellungen/Detail/partnerOrderDetailOrderUtils.js:90–103` (Hersteller, broken spelling at `:92` and `:102`)
- Admin dashboard: `src/app/(dashbaord)/dashboard/_components/Partnerbestellungen/partnerbestellungenOrderFilters.js:184–188` (the mapping the list already uses)
