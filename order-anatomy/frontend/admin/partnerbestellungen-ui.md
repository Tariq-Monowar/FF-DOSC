# Partnerbestellungen — UI improvement

**Pages:** `/dashboard/partnerbestellungen`, `/dashboard/partnerbestellungen-nachrichten`, and `/dashboard/partnerbestellungen/[id]`  
**Based on:** current screens (9 Oct 2026)  
**Status:** proposal for the client. Not implemented yet.

One document, five lessons. Lessons 1–3 are the order list. Lesson 4 is partner support. Lesson 5 is the order detail page: which data each tab shows, and how the PDF stays in step when a field changes.

---

## Summary

| Area | Today | Target |
| --- | --- | --- |
| Right sidebar “Bestellungen” | Open on first visit. Takes about 320 px on desktop. | Closed by default. The “Bestellungen anzeigen” button stays. |
| Filter bar | 12 controls in two full rows, always open. | One row: search, status, partner, sort. The rest sits behind “Filter”. |
| Bulk actions | 4 dropdowns and 3 buttons in one row as soon as orders are selected. | One compact row. The four actions live in one menu. |
| Order row | Six fixed columns, minimum width 1,280 px. Horizontal scroll until 1,891 px. | No sideways scroll. Important fields stay on the row. The rest moves to a second line or the expanded detail. |
| Partner support inbox | One shared list. Every admin user sees every partner chat. | Each partner has a dedicated staff member. That staff member sees only their own partners. Admin sees everyone. |
| Order detail | Finanzen shows the production form. Empty fields render as a broken character. PDF uses a second field list. | Each tab has one job. Screen and PDF read the same field catalog. A renamed field cannot disappear quietly. |

---

## 1. Right sidebar off by default

The page has a **Bestellungen** card on the right (insight cards, revenue, AI search). It is sticky and about 320 px wide on large screens (`xl:w-80`).

![Right sidebar: Umsatz card and Bestellungen insight list, open by default](https://feetf1rst.s3.eu-central-1.amazonaws.com/docs/order-anatomy/frontend/admin/partnerbestellungen-right-sidebar.png)

The header switch (“Bestellungen ausblenden” / “Bestellungen anzeigen”) remembers the choice. Until someone closes the panel, it stays **open**. A first visit therefore starts with less room for the list.

**Target**

- First visit: panel closed. The full width belongs to the order list.
- Anyone who needs it opens it with **Bestellungen anzeigen**.
- The last choice is saved. Open it once, and it is open again next time.
- Below the `lg` breakpoint nothing visible changes, because the panel already sits under the list there. Default “closed” only stops it from taking width on desktop immediately.

On a normal laptop (1,366–1,536 px) the list is no longer squeezed by the sidebar and a 1,280 px grid at the same time.

---

## 2. Filter bar

This is the area today. View tabs on top, two filter rows under them, and — once orders are selected — the bulk-action row.

![Filters, view tabs, and bulk actions on Partnerbestellungen](https://feetf1rst.s3.eu-central-1.amazonaws.com/docs/order-anatomy/frontend/admin/partnerbestellungen-filters.png)

### What is wrong

These are always visible, with no labels in the page layout:

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
11. Versandt / Ausgeführt switch
12. Alle Partner

At `2xl` (1,536 px) the second row is forced onto one line (`2xl:flex-nowrap`). Below that it wraps unevenly: search, the Prod. buttons, and “Ohne Prod.” stick together, while partner and shipping share the next line. On a laptop with the sidebar open it reads as a form, not a toolbar.

The bulk row appears on top of that as soon as at least one order is selected:

- Status wählen + **Status ändern**
- Leistenstatus wählen + **Leistenstatus ändern**
- Payment status wählen + **Zahlungsstatus ändern**
- Prioritaet waehlen

Each pair has a minimum width of about 180–240 px. Together with the text “10 Bestellungen ausgewählt”, the row is wider than a normal window and breaks in the middle of the actions.

### Target layout

**Always visible, one row**

`Search` · `Status` · `Partner` · `Sort` · **Filter** button · **Reset** (only when something is active)

The **Filter** button shows a count once a hidden filter is set, for example `Filter · 2`.

**Inside the filter panel** (opens under the row, closes again)

- Category
- Payment
- Order status (Auftrag)
- Last status (Leisten)
- Shipping direction
- Without production number
- Shipped + completed (Versandt + Ausgeführt)
- Sort by production number

**Active filters as chips** directly under the row. Each chip can be removed with ×. Example: `Category: Halbprobenerstellung` · `Payment: Unbezahlt` · `Ohne Prod.`

**Bulk actions**, only when something is selected, one row:

`10 selected` · `Clear selection` · **Change** button

**Change** opens a small panel with four groups, instead of four dropdowns side by side:

- Order status
- Last status
- Payment status
- Priority

Each group has its own dropdown and its own confirm button. Labels stay in German, and they stay consistent: “Zahlungsstatus wählen”, “Priorität wählen”.

On a narrow screen, search, status, partner, and sort stack. The filter panel is full width: two columns from `sm`, one column on a phone. Nothing in this bar gets a minimum width that pushes the page sideways.

Every filter that exists today, and all four bulk actions, stay available. They are just no longer all in the first view at once.

---

## 3. Order list

![Order rows with partner, product, finance, status, and actions](https://feetf1rst.s3.eu-central-1.amazonaws.com/docs/order-anatomy/frontend/admin/partnerbestellungen-orders.png)

### What is wrong

Each row is a fixed 6-column grid. The columns have fixed minimum widths:

| Column | Content | Minimum width |
| --- | --- | --- |
| 1 | Select, expand, order number | 11 rem |
| 2 | Partner, email, country | 13 rem |
| 3 | Product, category, customer | 11 rem |
| 4 | Amount, margin, payment | 10.5 rem |
| 5 | Status, order date, delivery date | flex, min. 11.5 rem |
| 6 | Handover, warning, three action icons | 14.5 rem |

The minimums plus gaps add up to about **80 rem (1,280 px)**. Below 1,891 px the list forces exactly that minimum width and scrolls horizontally (`max-[1890px]:min-w-[80rem]`). A 14-inch laptop, a split window, or the open right sidebar always produces a second scrollbar.

Row heights are also uneven. A short partner name stays on one line. “Orthopädieschuhtechnik GmbH & Co. KG” wraps onto three lines. Status, the amount, and the action icons no longer line up.

### Target layout

Same information, two densities. No column may make the page wider than the window.

**Desktop from 1,280 px window width, sidebar closed**

One row, left-aligned, with a steady row height where the text allows it:

- Select, expand
- Order number and type (Order / Unterbestellung)
- Partner name, one line, overflow as `…`, email in a tooltip
- Status badge
- Delivery date
- Amount
- Actions (handover, warning, the three icons)

A second, smaller line in the same card, in muted text:

Product · category · customer · margin · payment status · order date · country

Long names are truncated, not wrapped. The expanded detail (the chevron) stays the place for the email, the full partner name, and the remaining fields.

**Below 1,280 px, and whenever the right sidebar is open**

Cards, no grid:

- Header: select, order number, status
- Middle: partner, product, customer
- Footer: amount, delivery date, actions

The cards are 100% of the available width. The list has no horizontal scrollbar.

**Child orders** stay indented, with the colored left edge of the shipping group. The indent stays small (it is already `ml-2` / `sm:ml-4`) and must not push the card out of the window.

The “select all on this page” bar and the total count stay above the list, on one line, and wrap under each other on a phone.

---

## 4. Partner support — dedicated staff per partner

**Page:** `/dashboard/partnerbestellungen-nachrichten`

This inbox is the support channel. A partner who waits here, or who gets two different answers from two people, will not stay. The list, the filters, and the order screen can be tidy, and support can still fail if every chat is everyone’s chat.

Today every person under the admin account opens the same inbox. The list is one row per order. The only filters are **Alle** and **Ungelesen**. Search covers messages, customers, and orders. Mentioning a Mitarbeiter with @ only drops their name into one message. It does not give them the conversation, and it does not hide it from anyone else. The header says **Partner ↔ Admin**. There is no owner on the partner and no owner on the thread.

![Support inbox: one shared list of every partner order chat](https://feetf1rst.s3.eu-central-1.amazonaws.com/docs/order-anatomy/frontend/admin/partnerbestellungen-support-inbox.png)

That holds while the team is small. It does not hold once many partners write at the same time.

- A staff member scrolls partners they do not look after.
- Two people answer the same thread, or nobody answers because each person assumes the other will.
- The unread badge is one number for the whole team. It does not say “this one is yours”.
- A dedicated person cannot sit down and clear only their partners.

### Target

Assignment sits on the **partner**, not on each order. One partner has one primary staff member and one backup. Every order chat of that partner follows that assignment. Several staff members can support several partners at the same time, because their lists do not overlap.

**Admin**

- Sees every conversation.
- On the partner, sets the primary staff member and the backup.
- Can move a partner to someone else. Open chats move with the partner. The history stays in the thread.
- Has an **Unassigned** view, so a new partner is not left without an owner.

**Staff**

- The inbox opens on **Mine**: only the partners assigned to them.
- They do not see another staff member’s partner chats.
- Their unread badge counts only their partners.
- An @mention can pull them into one thread when the owner needs help. That opens that thread only. It does not open the rest of that partner’s list.

**While several people are online**

- Each person works their own list. Replies do not cross.
- The thread header shows the owner’s name, so the partner keeps seeing the same person.
- If a second person opens a chat the owner already has open, the header shows who is in it.
- The backup receives the partner only when the primary is marked away, or when a partner message has had no reply for a set time (start with 30 minutes). The backup keeps it until the primary takes it back.

**How the list should read**

Group by partner, with that partner’s open orders under the partner. A partner row shows the assigned staff member, the oldest message still waiting for a reply, and the unread count. Filters for staff: **Mine**, **Unread**, **Waiting on us**, **Waiting on partner**. Admin also gets **All staff**, **Unassigned**, and a filter by staff member.

**How a reply should work**

- A reply from FeetF1rst is sent under the assigned staff member’s name.
- An internal note stays inside the team. The partner does not see it. That is the handover between primary and backup.
- Each thread has a status: **Open**, **Waiting on partner**, **Done**. Done leaves the working list and keeps the history.

The chat itself stays on the order. The link to the Auftrag, file attachments, and search stay. Admin can still open any chat.

### Check

- Staff A signs in and sees only their partners.
- Staff B does not see those threads, and Staff A does not see Staff B’s.
- Admin sees every thread and the assignee on it.
- A new partner with no staff member appears only in the admin **Unassigned** view.
- Reassigning the partner removes the chats from Staff A and gives them to Staff B. The messages stay.
- Two staff members online at once: each one clears their own unread count, and the other person’s count does not change.

---

## 5. Order detail — one field list for the screen and the PDF

**Page:** `/dashboard/partnerbestellungen/[id]`

Example: Bestellung #10333, category Halbprobenerstellung. The bar is Übersicht, Produktion, Finanzen, Kommunikation, Rückmeldung, Historie, and the button Lieferung aktualisieren.

The form on the partner side changes. A field is renamed, a new yes/no is added, an old key is dropped. Today the detail page and the PDF each have their own copy of those fields. After a change, one of the two misses it. That is the gap to close.

### What is wrong

**Empty values look like broken text.** Kunde and Modell on #10333 are empty. The screen does not say “not set”. It prints the characters stored in the source where an em dash should be, so the value reads as `ä€"`. The back link is the same kind of damage: the file contains a broken “Zurück”. An empty field and a missing field look like a data bug.

![Übersicht for #10333: empty Kunde and Modell render as a broken dash](https://feetf1rst.s3.eu-central-1.amazonaws.com/docs/order-anatomy/frontend/admin/partnerbestellungen-detail-uebersicht.png)

**Finanzen is not finance.** That tab mounts the full order-detail view. For this category it prints “Halbprobenerstellung Details”: price, Kopfdaten, Leistentyp. Price, production cost, margin, and payment already sit in the Schnellübersicht on the right. The tab named Finanzen repeats the production form and does not show the invoice as the main content.

![Finanzen tab showing production fields, raw key names, and a column of Nein](https://feetf1rst.s3.eu-central-1.amazonaws.com/docs/order-anatomy/frontend/admin/partnerbestellungen-detail-finanzen.png)

**A new JSON key is shown as code.** Kopfdaten has a label map for `patient`, `auftraggeber`, and `leistenmaterial` only. Anything else is printed with `labelMap[key] || key`. On this order that is why `creationMode` and `modulation_by` appear as the raw names. There is no place that says “this key has no German label”. The next rename will either show another raw name or drop the old label with no trace.

**Every “no” is a row.** Leistentyp prints Ja or Nein for each boolean. A form that is mostly Nein becomes a long grid. The one selected option is hard to see. The same pattern will get worse each time a yes/no field is added.

**Three tabs are empty on purpose.** Produktion, Kommunikation, and Historie render a placeholder: “Inhalt folgt. API-Anbindung kann hier später ergänzt werden.” The production data that belongs on Produktion is sitting on Finanzen instead. Kommunikation does not open this order’s chat. Historie does not show the status timeline. The timeline component is only in the sidebar, and on this order it says no status changes have been recorded.

**The PDF is a second field list.** The Bodenkonstruktion PDF walks a long list of JSON paths (`form_data_v2`, then older keys, then another fallback). Halbprobenerstellung’s PDF is a stored file URL (`Halbprobenerstellung_pdf` or `kopfdaten.pdfUrl`), not a document built from the fields on screen. Change a field in the partner form and three things can diverge: the label map, the PDF path list, and the stored file. That is why a generated PDF is sometimes missing a field that the screen still shows, or the other way around.

**A category spelling hides a whole tab.** Visibility for AI-PDF, Versand, Rückmeldung, and the Hersteller block depends on the exact category string. The code also checks a broken spelling of Maßschafterstellung. One character of difference and a tab or the manufacturer block does not render, which looks like missing data.

### What the code should do

Keep **one field catalog per category**. Each entry has the JSON key, the German label, the group (Kopfdaten, Leistentyp, …), the type (text, yes/no, number, file), and which tab it belongs on. The detail screen and the PDF both read that catalog. A field change is one entry, not an edit in two files.

Render rules:

- The label always comes from the catalog. A key that is not in the catalog is not shown as `creationMode`. It is listed once under **Neue Felder**, with the raw key and the value, so it is visible until someone gives it a German label. Nothing disappears quietly, and nothing looks like a finished label when it is not.
- A yes/no group shows the **Ja** answers only, and one line for how many options were not selected. Do not print a column of Nein.
- An empty text value is **Nicht angegeben**. Replace the broken dash strings in the detail files, and save those files as UTF-8, so the back link reads “Zurück” and empty Kunde / Modell read “Nicht angegeben”.
- A file (PDF, STL) is a button. It is not a row of URLs. The catalog marks which keys are files.

What each tab is for:

| Tab | Shows |
| --- | --- |
| Übersicht | Partner, customer, model, type, received date, status, manufacturer, delivery. Six Stammdaten fields, from the customer and model already on the order. |
| Produktion | The category specification that is now dumped under Finanzen. Grouped, German labels, Ja only. |
| Finanzen | Price, production cost, margin, payment, invoice download, pickup label. Not the specification. |
| Kommunikation | This order’s existing chat, opened on this shaft. Not a placeholder. |
| Rückmeldung | Stays for Halbprobenerstellung. |
| Historie | The status timeline the sidebar already loads. The sidebar keeps the short list. The tab shows the full list. Same events, not a second source. |
| Lieferung aktualisieren | Stays a button. It writes the delivery date that Übersicht and Historie read. |

PDF:

- Build it from the same catalog as Produktion.
- If a required field has no value, the download names that label instead of leaving a blank line or keeping an old value.
- For Halbprobenerstellung, keep the partner’s original file as **Original vom Partner**. The generated sheet is a separate file, **Auftragszettel aus diesen Feldern**, and it uses the catalog.

### Check

- #10333 Übersicht: Kunde and Modell say Nicht angegeben. The back link says Zurück zu Partnerbestellungen.
- Finanzen shows price, cost, margin, payment, and the invoice. It does not show Leistentyp.
- Produktion shows Halbprobenerstellung in groups, with German labels, and only the selected options.
- A JSON key that is not in the catalog appears under Neue Felder, on Produktion and in the PDF check, until it has a label.
- PDF download names any required field that is missing.
- Kommunikation on this order opens that order’s chat.
- Historie and the sidebar timeline show the same status events.

---

## Order of work

1. Sidebar closed by default. The list gets its width back immediately. Behavior stays the same except for that first state.
2. Filters on one row plus a panel. Bulk actions in one menu. The two crowded rows in the first screenshot go away.
3. Order row moves to the two-line layout, or the card, and `min-w-[80rem]` is removed. After that the list does not scroll sideways on laptop, tablet, or phone.
4. Support inbox assigns each partner to one primary staff member and one backup. Staff see only their own partners. Admin keeps the full list and the unassigned queue.
5. Order detail uses one field catalog per category for the screen and the PDF. Finanzen shows money. Produktion shows the specification. Empty values say Nicht angegeben. Unknown keys show up under Neue Felder instead of disappearing.

---

## Check after implementation

- Fresh browser, page opened for the first time: right panel closed, list uses the full width.
- “Bestellungen anzeigen” opens the panel; after reload it stays open. “Bestellungen ausblenden” and reload: closed again.
- With every filter on its default, the toolbar fits on one line from 1,280 px.
- A set category filter and payment filter show up as chips and count on the Filter button.
- Ten selected orders: one bulk row, all four changes (status, last, payment, priority) reachable, and the page does not scroll sideways.
- List at 1,440 px, 1,280 px, 1,024 px, and 390 px: no horizontal scrollbar on the order list. Actions, status, and order number stay readable without expanding the row.
- Expanding a row still shows partner email, product, margin, and dates.
