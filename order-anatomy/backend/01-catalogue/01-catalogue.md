# Catalogue shoes (`maßschaft_kollektion`)

This is the shoe menu. It is not a customer order.

FeetF1rst prepares shoe models in advance: a name, a photo, a price, who the shoe is for, how it closes, and the usual leather and seam. A partner later opens one of those models and orders it for a customer. The menu item stays in the catalogue. The order is a separate record, filled from this menu item at the moment the partner sends it.

Two screens do this work.

| Who | Screen | What they do |
| --- | --- | --- |
| Admin | Product form in `feetf1rst-landing-page`, page `add-products` | Adds and edits a shoe in the menu |
| Partner | Order page in the Partner Dashboard, page `custom-shafts/details/[id]` | Picks that shoe and sends the customer order |

The database table is `maßschaft_kollektion` in `prisma/admin-order.schma.prisma`. One row is one shoe in the menu.

## The story, in order

1. Admin opens the product form and fills the shoe: name, price, category, men's or women's, closure, photo, leather, seam.
2. The form saves one catalogue row. The long-lived identity of that shoe is `id`.
3. The partner opens the catalogue, picks that shoe, and lands on the order page. The page address contains the same `id`.
4. The order page reads the catalogue row and fills the form: category, closure, zipper, name, price, photo. If the partner chooses "Ausführung wie im Katalog", leather, lining, seam, and the leather photo are filled too.
5. The partner adds the customer, the last, and any extras, then sends the order.
6. The order is stored on `custom_shafts` and points back at this shoe with `maßschaftKollektionId`. The catalogue row is still the menu item. Editing the menu later does not rewrite orders that were already sent.

The copy in step 4 happens in the partner's browser, in the Partner Dashboard, before the order is sent. The server checks that the shoe `id` still exists, then saves what the screen already prepared.

## What is stored on each shoe

| What a person sees | Column | What is actually saved |
| --- | --- | --- |
| The real identity. Every order uses this. | `id` | A cuid, created by the database. Nobody types it. |
| A short 4-digit label, for display | `ide` | A random number from `1000` to `9999`. Two shoes can share it. It is not the order number. |
| Shoe name | `name` | Required when saving. |
| Main photo | `image` | The file's web address. If no file is sent, the column is an empty string. Lists show that empty string as blank. |
| Price | `price` | A number. The product form refuses `0` and blank. |
| Category, such as Stiefel or Halbschuhe | `catagoary` | Free text. The spelling in the database is `catagoary`. The list filter is called `category`. |
| Men's, women's, or both | `gender` | The product form saves `Herren`, `Damen ` (there is a space after Damen), or `Both`. |
| Description | `description` | Required by the save API. The product form does not warn if it is empty, so an empty description comes back as an error. |
| How the shoe closes | `verschlussart` | `Eyelets` (Ösen), `Zipper` (Reißverschluss), or `Velcro` (Klettverschluss). |
| Extra zipper on this model | `is_zipper` | Yes or no. Saved separately from the closure above. |
| Decorative seam | `ziernaht` | Yes or no. |
| Leather, lining, and seam preset | `Massschafterstellung_json1` | One JSON object. The shape is below. |
| Picture of the leather layout | `ledertyp_image` | A web address. If no file is sent, the column is an empty string. |
| When the shoe was added, and when it was last changed | `createdAt`, `updatedAt` | Set by the database. |

### Closure and zipper are two answers

The form has a closure dropdown and a zipper yes/no. Both are saved. Choosing `Zipper` in the dropdown does not, by itself, turn `is_zipper` on. The person filling the form sets both.

On a partner order, `is_zipper: true` starts the order with the extra zipper switched on. `false` starts it switched off.

### Men's and women's

The product form saves these three values only:

| Saved text | Label on the form |
| --- | --- |
| `Herren` | Herren |
| `Damen ` | Damen. The saved text includes a space at the end. |
| `Both` | Beide |

A filter for Herren also shows shoes marked Both. A filter for Damen also shows Both. Damen with the space and Damen without the space both match a Damen search, because the search looks for the letters, not the exact text.

Changing many shoes at once (`update-as-bulk`) is stricter. Gender must be exactly `Herren`, `Damen ` with the space, or `Both`. `Damen` without the space is rejected.

### The material preset

`Massschafterstellung_json1` is the shoe's usual materials, saved with the menu item. The product form builds it like this (`add-products/page.jsx`):

- How many leathers (`1` to `4`)
- For one leather: leather type and colour
- For more than one leather: colour 1–4, plus the painted areas on the photo (`assignments`)
- Lining (`innenfutter`)
- Seam colour: `default`, `personal`, or `custom`, and the custom text when they typed their own
- Whether the decorative seam is on (`ziernahtVorhanden`, the same yes/no as `ziernaht`)

When the partner later chooses "Ausführung wie im Katalog", the order page reads this object and fills those fields. The order then saves its own, larger material note on the order itself. That note includes shaft height, padding, and delivery. It is not a second copy of this menu field. It is the finished specification for that one customer.

## Who is allowed to change the menu

Every catalogue route accepts an admin, a partner, or an employee login. The comment in the code says the admin builds the product and the partner orders it. The login check is wider than that comment: a partner who knows the address can create, edit, or delete any shoe in the menu. The row does not record who created it.

## Which address to call

The app answers the same routes with or without a `/v1` prefix. `/custom_shafts/...` and `/v1/custom_shafts/...` are the same code. The word in the address is `mabschaft_kollektion`.

Use this set. It saves zipper, seam, the material preset, and the leather photo.

| What you want | Method and path |
| --- | --- |
| Add a shoe | `POST /custom_shafts/create/mabschaft_kollektion` |
| List shoes | `GET /custom_shafts/mabschaft_kollektion` |
| Change price, gender, or category on many shoes | `PATCH /custom_shafts/mabschaft_kollektion/update-as-bulk` |
| Change one shoe | `PATCH /custom_shafts/mabschaft_kollektion/:id` |
| Open one shoe | `GET /custom_shafts/mabschaft_kollektion/:id` |
| Remove one shoe | `DELETE /custom_shafts/mabschaft_kollektion/:id` |

There is an older set under `/admin_product/...` with the same job names. The product form does not use it. That older save ignores zipper, seam, the material preset, and the leather photo, so a shoe saved there looks empty in those places. Its list also shows every shoe to every login, including shoes that were limited to certain partners.

## Adding a shoe

The product form will not submit until name, a price above 0, category, gender, closure, and a photo are filled. It still sends description. If description is blank, the API answers `400` with `description is required!`.

The form sends a file upload (`multipart`). The fields that become columns are:

| Sent field | Lands in |
| --- | --- |
| `name`, `price`, `catagoary`, `gender`, `description`, `verschlussart` | Those columns. All six are required. |
| `is_zipper`, `ziernaht` | Yes/no. The form sends the words `true` or `false`. Missing means no. |
| `image` | Main photo. |
| `ledertyp_image` | Leather-layout photo, only when the form has a painted preview. |
| `Massschafterstellung_json1` | The material preset, as one JSON text. |

The form also sends leather type, leather colours, lining, and seam colour as their own fields. Those names are not columns. The values that stick are the ones inside `Massschafterstellung_json1`.

On success the API answers `201` and returns the new shoe, including `id` and the random `ide`. If saving fails after the photos were uploaded, the main photo is removed from storage. The leather photo can be left behind.

The save itself is:

```ts
await prisma.maßschaft_kollektion.create({
  data: {
    ide: randomIde,
    name,
    price: parseFloat(price),
    catagoary,
    gender,
    description,
    image: imageUrl || "",
    ledertyp_image: ledertypImageUrl || "",
    verschlussart: verschlussart || null,
    is_zipper: parseOptionalBool(is_zipper) ?? false,
    ziernaht: parseOptionalBool(ziernaht) ?? false,
    Massschafterstellung_json1: Massschafterstellung_json1 || null,
  },
});
```

`parseOptionalBool` treats `true`, `1`, `yes`, and `on` as yes, and `false`, `0`, `no`, and `off` as no.

## Opening the menu

`GET /custom_shafts/mabschaft_kollektion`

| Query | Meaning |
| --- | --- |
| `page` | Page number. Starts at 1. |
| `limit` | How many shoes per page. Starts at 10. |
| `search` | Matches name, category, gender, or description. |
| `gender` | `Herren`, `Damen`, `both`, or a comma list of those. Herren and Damen also include Both. |
| `category` | Exact category, ignoring capital letters. |
| `sortPrice` | `asc` or `desc`. Otherwise newest shoes come first. |

Each shoe in the answer also has two flags that are not columns:

- `isFavorite` — this partner has marked the shoe as a favorite.
- `isExclusive` — for a partner, this shoe was shared with them by name. For an admin, the shoe is limited to a named partner list and is not on the public menu.

A partner or an employee sees a shoe when it is on the public menu, or when their partner is on that shoe's named list. An admin sees every shoe. Shoes shared with the viewer are listed before public shoes.

Opening one shoe is `GET /custom_shafts/mabschaft_kollektion/:id`. An unknown id answers `404`. A shoe the partner is not allowed to see answers with the same `404`. The message does not say "hidden".

An employee's list and an employee's open-one do not always resolve "which partner do I work for" the same way. A shoe can appear in the employee's list and then `404` when they open it.

## Changing a shoe

Change one shoe with `PATCH /custom_shafts/mabschaft_kollektion/:id`. Send only the fields that should change. A new photo replaces the old one, and the old file is removed from storage when its address starts with `http`. Unknown id is `404`. Sending nothing recognised is `400`.

Change many shoes with `PATCH /custom_shafts/mabschaft_kollektion/update-as-bulk` and a JSON body:

```json
{
  "ids": ["<shoe id>", "<shoe id>"],
  "price": 450,
  "gender": "Damen ",
  "catagoary": "Stiefel"
}
```

Only price, gender, and category can be changed this way. Photos, closure, zipper, seam, and the material preset stay as they are. If one of the ids does not exist, that id is skipped. The answer tells you how many ids you sent (`matchedIds`) and how many rows actually changed (`updatedCount`).

## Removing a shoe

`DELETE /custom_shafts/mabschaft_kollektion/:id`

If a customer order still points at this shoe, the delete is refused: `400`, "Cannot delete this kollektion because it is being used elsewhere in the system." The menu row stays. The main photo has already been removed from storage, so the shoe that remains can show a broken picture. The leather photo is not removed either way.

If no order points at it, the shoe is deleted. Partner favorites of that shoe go with it. A named partner list for that shoe goes with it. A clinical draft that pointed at it stays, and its link to the shoe becomes empty.

## How the partner order uses this shoe

The order page is in the Partner Dashboard, at `custom-shafts/details/[id]`. The `id` in that address is the catalogue shoe. The page loads `GET /custom_shafts/mabschaft_kollektion/:id`.

It then fills the order form from the row:

| From the catalogue shoe | Onto the order form |
| --- | --- |
| `catagoary` | Category |
| `verschlussart` | Closure. If the shoe has none, the form stays on Ösen (`Eyelets`). |
| `is_zipper` true | Extra zipper on |
| `is_zipper` false | Extra zipper off, zipper side and zipper photo cleared |
| `name` | Product name on the order |
| `price` | Starting price |
| `image` | The shoe photo on the screen. It is not uploaded again as a custom shoe picture. |
| `id` | Sent as `mabschaftKollektionId` |

This fill is skipped when the page is reopening an order that was already saved as Massschafterstellung or Komplettfertigung. In that case the form keeps the saved order.

"Ausführung wie im Katalog" reads `Massschafterstellung_json1` and fills leather count, leather type, leather colour, painted areas, lining, seam colour, decorative seam, and puts `ledertyp_image` into the paint image.

Sending the order posts to:

- A normal order: `POST /custom_shafts/create?custom_models=&isCourierContact=yes` or `no`
- A new draft: `POST /custom_shafts/create?is_draft=yes`

`isCourierContact` is `no` when a 3D last is already attached. Otherwise a pickup is `yes`, and sending the last yourself is `no`.

The order body carries `mabschaftKollektionId`, the customer, the 3D last, the invoice PDF when there is one, zipper and paint images when the partner kept them, the price, the delivery date, and a new `Massschafterstellung_json1` built from the filled form. Inside that order JSON, `meta.produkt_bezeichnung` is the catalogue name, `meta.kategorie_anzeige` is the catalogue category, and `meta.mabschaft_kollektion_id` is this shoe's id. `meta.ausfuehrung_wie_im_katalog` is true when the partner used the catalogue preset.

The server parses that JSON and stores it on the order. It links the order to the shoe. It does not read the catalogue name, price, or photo again. Those already travelled with the form.

A fully custom shoe, with no catalogue model, is a different button. That request uses `custom_models=true` and does not send `mabschaftKollektionId`.

## What can go wrong

**Saving through the old address loses half the shoe.**  
`/custom_shafts/create/mabschaft_kollektion` stores zipper, seam, materials, and the leather photo. `/admin_product/create/mabschaft_kollektion` stores the name, price, category, gender, description, closure, and main photo only. The product form uses the first address. Anything still pointed at `/admin_product` saves a shoe that later looks like it has no zipper, no seam, and no leather preset.

**A partner can edit the whole menu.**  
The intended work is: admin maintains the menu, partner orders from it. The API lets a partner or an employee create, change, and delete any shoe. There is no "created by" on the row. If the menu should stay an admin list, those three actions need to allow admin only.

**The 4-digit number can repeat.**  
`ide` is a random label from 1000 to 9999, and nothing stops two shoes from getting the same one. Orders, links, and the order page all use `id`. Treat `ide` as a label on the card.

**Women's shoes depend on a trailing space.**  
The form and the bulk editor save women's shoes as `Damen ` with a space. A one-shoe save will also accept `Damen` without the space, or any other word. Search still finds both, because it searches for the letters `Damen`. Bulk edit rejects the version without the space. One rule for all three (form, single save, bulk save) would stop a women's shoe from vanishing out of a bulk update.

**Closure and zipper can disagree.**  
The API stores the dropdown and the zipper checkbox as two fields. The product form sends both. A hand-built request can say the closure is Ösen and the zipper is yes. The order page will follow `is_zipper` for the zipper switch, and `verschlussart` for the closure dropdown.

**Deleting a shoe that is already ordered removes the photo and keeps the shoe.**  
The API deletes the main photo first, then tries to delete the row. If an order still uses the shoe, the row stays and the photo is already gone. The leather photo is never deleted. A safe order is: refuse the delete while orders exist, and only then remove both photos.

**A failed save can leave the leather photo in storage.**  
On a failed create, or on an update whose shoe id does not exist, the new main photo is removed. The new leather photo is not.

**"A shoe with this name already exists" does not match this table.**  
Update answers that when the database reports a duplicate. This table has no unique rule on the name, so two shoes may share a name, and that message is not something this schema produces.

**An employee can see a shoe and then be told it does not exist.**  
The list looks up the employee's partner. Opening one shoe does that lookup only in some login shapes. The same person can get the shoe in the list and a not-found when they open it. Both actions should use the same partner lookup.

**Search does not use the indexes that were added for search.**  
`name`, `catagoary`, and `gender` are indexed for an exact or prefix lookup. The menu search looks for the text anywhere inside the field (`ILIKE '%word%'`), which does not use those indexes. The category filter lowercases both sides, so it does not use the category index either. The indexes still help an exact match. They do not speed up the search box.
