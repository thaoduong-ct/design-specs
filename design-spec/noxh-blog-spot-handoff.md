# NOXH Hub · Blog spot — Handoff

Additive change to **NOXH Hub · Dự án tab** (existing screen). Two new sections appended below the existing project-ad list; nothing above them is modified.

| | |
|---|---|
| **Platform** | iOS + Android (single 375-wide design serves both) |
| **Vertical** | PTY (Nhà Tốt) |
| **Feature type** | New (US5 news section) + new (load-more UX for project-ad pagination) |
| **Figma** | [Vertical Handoff · 2026 — Blog spot](https://www.figma.com/design/UvLAgVP7Em1fwytMbuKHuv/Vertical-Handoff-%E2%80%A2-2026?node-id=20262-26868) · file `UvLAgVP7Em1fwytMbuKHuv` · section node `20262:26868` · frames F0 `20253:23755` (idle), F1 `20253:24115` (loading) |
| **PRDs** | [`noxh-buyer-actions-prd.md`](https://github.com/carousell/ct-product-planning/blob/main/specs/vertical/pty/noxh-buyer-actions-prd.md) US5 · [`noxh-hub-detail-prd.md`](https://github.com/carousell/ct-product-planning/blob/main/specs/vertical/pty/noxh-hub-detail-prd.md) US2 |
| **Scope sign-off** | thaoduong@chotot.vn · 2026-09-22 |
| **Sources read** | Figma REST `/nodes` (full, 892 nodes) · REST `/images` (screenshot + thumbnail) · both PRDs |
| **Not read** | Figma `variables/local` — PAT lacks `file_variables:read` scope. Every bound field is verified as bound, but variable **names** are unresolved. See [Token caveat](#token-caveat). |
| **Rename** | Not done — codebase-review round skipped in this session. See [Open items I-2](#open-items). |

---

## 1 · What to build

Two new sections, in this order, appended to the existing Dự án tab section list:

| # | spec_id | what | source |
|---|---|---|---|
| 1 | `sections/load_more_control` | Pagination control for the project-ad list. **Idle** = `Xem thêm` button. **Loading** = DS spinner (36×36). | design F0 vs F1 diff · DR-1, DR-2 |
| 2 | `sections/news_hub_noxh` | "Tin tức Nhà ở Xã hội" — horizontally scrollable news cards (thumbnail + headline + date), title + `Xem tất cả →` link. **Hide entire section when 0 articles.** | PRD US5 · design layout.6 (identical in F0 and F1) |

### Requirements

**From PRD `noxh-buyer-actions-prd.md` US5 (Tin Tức Nhà Ở Xã Hội)**
- **FR-1** — "Tin tức Nhà ở Xã hội" section on Dự án tab
- **FR-2** — Articles as horizontally scrollable cards
- **FR-3** — Each card = thumbnail + headline (truncated) + date. **PRD also lists `read time` — the DESIGN OMITS IT and designer confirmed design is correct (2026-09-22 Q1 answer). Update the PRD to match.**
- **FR-4** — Tap card → open article in **in-app WebView** (designer decision — Q3, 2026-09-22)
- **FR-5** — "Xem tất cả →" next to title → opens full article list
- **FR-6** — Articles sourced from `https://www.nhatot.com/kinh-nghiem/search/Nhà+ở+xã+hội`
- **FR-7** — If no articles available, hide the section entirely (no empty state)

**From design (new UX pattern for pagination, no PRD coverage)**
- **DR-1** — Load-more control has two states: idle = `Xem thêm` button; loading = DS spinner (36×36)
- **DR-2** — Control is centred horizontally; container has 12px vertical padding
- **DR-3** — When no more pages exist → hide the `Xem thêm` button (no empty state, no disabled state)
- **DR-4** — When load fails → show a toast; restore the `Xem thêm` button so the user can retry

### Full screen composition (context — only 2 sections need building)

The full section list on the Dự án tab, top → bottom. Existing sections are unchanged by this feature; only the two `NEW` rows have detailed section trees below.

| # | spec_id | status | figma_node (F0) | note |
|---|---|---|---|---|
| 1 | `sections/status_bar` | OUT (OS chrome) | — | skip |
| 2 | `sections/top_navigation` | keep-as-is | `20253:23849` | DS instance `Top Navigation / Main Screen`, byte-identical in F0/F1 |
| 3 | `sections/promo_banner` | keep-as-is | `20253:23851` | KIỂM TRA NGAY / Điều kiện mua NOXH row (US4 entry point) |
| 4 | `sections/tab_bar` | keep-as-is | — | Dự án / Tin đăng tabs (US2 base) |
| 5 | `sections/filter_container` | keep-as-is | `20253:23902` | Khu vực filter row |
| 6 | `sections/filter_chips_row` | keep-as-is | `20253:23904` | Region chips (Tất cả, Bình Dương (3), …) |
| 7 | `sections/pagination_indicator` | keep-as-is | `20253:23906` | Pagination dot indicator |
| 8 | `sections/project_ad_list` | keep-as-is | `20253:23757` (× 5 cards) | US2 listing — data-driven count, not modified |
| 9 | **`sections/load_more_control`** | **NEW** | F0 `20253:23908` / F1 `20253:24441` | See [§2](#2--sectionsload_more_control) |
| 10 | **`sections/news_hub_noxh`** | **NEW** | `20253:23910` (identical in F1 `20253:24270`) | See [§3](#3--sectionsnews_hub_noxh) |

---

## 2 · sections/load_more_control

**Requirement context.** Pagination control appended below the existing project-ad list. Not previously present. Two states, driven by the load-more request lifecycle.

**Acceptance criteria**
- DR-1: Idle state shows `Xem thêm` button (DS Button, M-32px, Tertiary, Pill, State=Active)
- DR-2: Container centres control horizontally, 12px vertical padding
- DR-3: When no more pages exist → entire control hidden (do not render)
- DR-4: On load fail → show toast, restore idle button
- Loading state replaces the button with a DS spinner (36×36)

**Uses** — behaviors: BH-3, BH-4, BH-5, BH-6, BH-7 · DS components: `button_m32_tertiary_pill`, `spinner_default`

### Variant `idle` — `when: hasMorePages == true && loadingMore == false`
Figma: `20253:23908` (F0 · Frame 2085668415)

```yaml
node_id: "20253:23908"
name: "Frame 2085668415"       # rename target: [SEC] load_more_control__idle
type: FRAME
size: [375, 56]
style:
  display: flex
  flex-direction: column
  gap: 4px
  padding: 12px 12px 12px 12px
  align-items: center           # counterAxisAlignItems=CENTER
  background: transparent
children:
  - node_id: "20253:24098"
    name: "Xem thêm button"
    type: INSTANCE
    ds_component: button_m32_tertiary_pill
    size: [92, 32]
    content:
      "↳ Input Text#896:1": "Xem thêm"
      "Icon Left#896:98": false     # icon hidden
    variant_props:
      Size: "M - 32px"
      Type: "🥉  Tertiary"
      State: Active
      isPill: "Yes"
    behavior: BH-3
    # DS handles internal geometry, colour, hover/pressed/disabled.
```

### Variant `loading` — `when: loadingMore == true`
Figma: `20253:24441` (F1 · Frame 2085668416)

```yaml
node_id: "20253:24441"
name: "Frame 2085668416"       # rename target: [SEC] load_more_control__loading
type: FRAME
size: [375, 60]                 # 4px taller than idle — spinner 36 vs button 32
style:
  display: flex
  flex-direction: column
  gap: 4px
  padding: 12px 12px 12px 12px
  align-items: center
  background: transparent
children:
  - node_id: "20253:24636"
    name: "Loading spinner"
    type: INSTANCE
    ds_component: spinner_default
    size: [36, 36]
    behavior: BH-4
    replaces_in_design: |
      The design draws the spinner as a raw group (Group 7 Copy 6 = Rectangle
      + Boolean + Vector + Ellipse). Designer confirmed the intent is the
      app's default DS spinner (iOS native activity indicator, Android
      ProgressBar). Do NOT re-implement the drawn shape.
```

### Hidden variant — `when: hasMorePages == false`
Section is not rendered at all. Reason: DR-3.

### Sources
- Layout & style: figma REST `/nodes` on `20253:23908` and `20253:24441` (verbatim)
- Variant props: figma REST `componentProperties` on the button; spinner has no component props (raw group in design)
- Behaviour: designer sign-off 2026-09-22 (Q3 Recommended)
- Token names: `[UNVERIFIED — file_variables scope missing]`

---

## 3 · sections/news_hub_noxh

**Requirement context.** New horizontally-scrollable news section on the Dự án tab. Fetches NOXH articles from a fixed source URL. Renders 0-or-many cards; when 0 articles are available the section is hidden entirely (no empty state).

**Acceptance criteria**
- FR-1: Section renders on Dự án tab
- FR-2: Cards are horizontally scrollable
- FR-3: Each card = thumbnail + headline (2-line truncated with ellipsis) + date. **No read time.**
- FR-4: Tap card → open article in in-app WebView
- FR-5: `"Xem tất cả →"` (Ghost button, S-24px, Pill) opens full article list
- FR-6: Article source = [news_source_url](#data-rules)
- FR-7: Hide entire section when 0 articles

**Uses** — behaviors: BH-1, BH-2 · DS components: `button_s24_ghost_pill` · data rules: `news_source_url`, `news_card_tap_url` · display rules: `news_section_hide_when_empty`

**Design note.** Figma outer container name is `Thông số` ("stats/specs"). It is actually the news feed. Rename target: `[SEC] news_hub_noxh`.

Identical in F0 (`20253:23910`) and F1 (`20253:24270`). Tree read from F0.

### Layout tree (lossless)

```yaml
node_id: "20253:23910"
name: "Thông số"                # misnamed in Figma → rename to news_hub_noxh
type: FRAME
role: section-root
size: [375, 302]
style:
  display: flex
  flex-direction: column
  gap: 8px
  padding: 12px 16px 12px 16px
  background: "#ffffff"
  border-radius: 12px           # cornerRadius: 12
  overflow: hidden              # clipsContent: true
  box-sizing: border-box
  width: 100%
  height: 302px
token_hint:
  background: "[UNVERIFIED] — expected DS: --color-surface / white"
  padding: "[UNVERIFIED] — all four sides bound to variables"
  border-radius: "[UNVERIFIED] — bound to a corner-radius variable"

children:

  # ─── Title row ───
  - node_id: "20253:23911"
    name: "Title row"              # figma: "Frame 2085667676"
    type: FRAME
    style:
      display: flex
      flex-direction: row
      gap: 8px
      align-items: stretch
      width: 100%
      height: 24px
    children:
      - node_id: "20253:23912"
        name: "Title"
        type: TEXT
        text: "Tin tức Nhà ở Xã hội"
        style:
          flex: 1                  # layoutSizingHorizontal FILL + layoutGrow 1
          font-family: "Reddit Sans"
          font-weight: 700         # Bold
          font-size: 16px
          line-height: 24px
          letter-spacing: 0
          text-align: left
          vertical-align: middle
          color: "#222222"         # bound
        token_hint:
          color: "[UNVERIFIED] — expected DS: --color-text-primary"
          font-family: "[UNVERIFIED] — expected DS: --font-family-primary"
          font-weight: "[UNVERIFIED] — expected DS: --font-weight-bold"
          font-size: "[UNVERIFIED]"
          line-height: "[UNVERIFIED] — bound"

      - node_id: "20253:23913"
        name: "Xem tất cả button"
        type: INSTANCE
        ds_component: button_s24_ghost_pill
        size: [88, 24]
        content:
          "↳ Input Text#896:1": "Xem tất cả →"
          "Icon Left#896:98": false    # icon hidden
        variant_props:
          Size: "S - 24px"
          Type: "🔗  Ghost"
          State: Active
          isPill: "Yes"
        behavior: BH-2

  # ─── Cards row (horizontally scrollable) ───
  - node_id: "20253:23914"
    name: "Cards row"              # figma: "Frame 2085667675"
    type: FRAME
    style:
      display: flex
      flex-direction: row
      gap: 8px
      width: 100%                  # 343 inside padded container
      height: 246px
      overflow-x: auto             # overflowDirection: HORIZONTAL_SCROLLING
      overflow-y: hidden
      # clipsContent: false — cards can peek beyond the container
      -webkit-overflow-scrolling: touch
    data:
      source: news_source_url      # see Data rules
      min_count_for_render: 1      # FR-7 → 0 = hide entire section
      repeat: card                 # `card` template repeated per item

    children:
      # One card template. Design draws 4 identical cards; code generates from data.
      - node_id: "20253:23915"
        template_id: card
        name: "News card"          # figma: "Main info" (× 4)
        type: FRAME
        style:
          display: flex
          flex-direction: column
          width: 232px
          height: 246px
          border: 1px solid "#e8e8e8"   # bound
          border-radius: 16px
          overflow: hidden
          box-sizing: border-box
          flex-shrink: 0            # so cards don't shrink under horizontal-scroll parent
        token_hint:
          border-color: "[UNVERIFIED] — expected DS: --color-border-subtle"
          border-radius: "[UNVERIFIED] — bound"
        behavior: BH-1
        data:
          url: data.url            # BH-1 opens this in in-app WebView

        children:
          - node_id: "20253:23916"
            name: "Thumbnail"      # figma: "Rectangle 240648306"
            type: IMAGE
            size: [232, 160]
            style:
              width: 232px
              height: 160px
              border-radius: 8px
              object-fit: cover    # scaleMode STRETCH → cover in code
              background: "#f4f4f4"    # placeholder while loading
            src: data.thumbnail_url
            placeholder_note: |
              All 4 cards in the design share the same imageRef
              7045a38c5c75f53a19cd805b2b203d55b9debe71 (sample data).
              In production each card gets its own thumbnail_url from the
              article record.

          - node_id: "20253:23917"
            name: "Item Info"
            type: FRAME
            style:
              display: flex
              flex-direction: column
              gap: 12px
              padding: 12px 12px 12px 12px
              width: 100%
              height: 86px
              box-sizing: border-box

            children:
              - node_id: "20253:23918"
                name: "Text stack"    # figma: "Div [flex]"
                type: FRAME
                style:
                  display: flex
                  flex-direction: column
                  gap: 4px
                  width: 100%
                  height: 62px

                children:
                  - node_id: "20253:23919"
                    name: "Headline"
                    type: TEXT
                    text: data.headline
                    max_lines: 2
                    truncation: end         # textTruncation: ENDING (ellipsis)
                    style:
                      width: 100%
                      font-family: "Reddit Sans"
                      font-weight: 600      # SemiBold
                      font-size: 14px
                      line-height: 20px
                      letter-spacing: 0
                      text-align: left
                      vertical-align: middle
                      color: "#222222"
                      display: -webkit-box
                      -webkit-line-clamp: 2
                      -webkit-box-orient: vertical
                      overflow: hidden
                      text-overflow: ellipsis
                    placeholder: "TP.HCM: Bổ sung nguồn cung 15.000 căn nhà ở cho công nhân, người lao động"
                    token_hint:
                      color: "[UNVERIFIED] — expected DS: --color-text-primary"
                      font-family: "[UNVERIFIED]"
                      font-weight: "[UNVERIFIED] — expected DS: --font-weight-semibold"
                      font-size: "[UNVERIFIED]"
                      line-height: "[UNVERIFIED] — bound"

                  - node_id: "20253:23920"
                    name: "Date"
                    type: TEXT
                    text: data.date_display   # format dd/MM/yyyy — see Data rules
                    style:
                      width: 100%
                      font-family: "Reddit Sans"
                      font-weight: 400       # Regular
                      font-size: 12px
                      line-height: 18px
                      letter-spacing: 0
                      text-align: left
                      vertical-align: middle
                      color: "#8c8c8c"       # bound
                    placeholder: "26/05/2026"
                    token_hint:
                      color: "[UNVERIFIED] — expected DS: --color-text-secondary or --color-text-tertiary"
                      font-family: "[UNVERIFIED]"
                      font-size: "[UNVERIFIED]"
                      line-height: "[UNVERIFIED] — bound"
```

### Sources
- Layout & style: figma REST `/nodes` on `20253:23910` (F0) — identical tree in F1 (`20253:24270`)
- Variant props: figma REST `componentProperties` on `20253:23913` (Xem tất cả button)
- Text style: figma REST `style.*` on `20253:23912`, `23919`, `23920`
- Behaviour BH-1 tap target: designer sign-off 2026-09-22 (Q3 — in-app WebView)
- Behaviour BH-2 target route: **TBD** — see [Open items I-3](#open-items)
- FR-7 hide-when-empty: PRD US5 AC-7
- Token names: `[UNVERIFIED — file_variables scope missing]`

---

## 4 · DS components

| semantic name | figma id | variant props locked | used by | notes |
|---|---|---|---|---|
| **`button_m32_tertiary_pill`** | `104:8835` | Size=`M - 32px`, Type=`🥉  Tertiary`, State=Active, isPill=Yes | `sections/load_more_control#idle` | Config per instance: text via `↳ Input Text#896:1`, icon-left visibility via `Icon Left#896:98`, icon swap via `↳ Icon Left#12667:0`. Xem thêm = text `"Xem thêm"`, Icon Left = false. |
| **`button_s24_ghost_pill`** | `104:9051` | Size=`S - 24px`, Type=`🔗  Ghost`, State=Active, isPill=Yes | `sections/news_hub_noxh#title-row.button` | Xem tất cả = text `"Xem tất cả →"`, Icon Left = false. The `→` arrow is part of the text string, not a separate icon. |
| **`spinner_default`** | `null` (NOT a DS instance in the design) | — | `sections/load_more_control#loading` | Per designer sign-off — use the app's default DS spinner (iOS native activity indicator, Android ProgressBar). Do NOT reproduce the drawn 36×36 shape group. The 36×36 is a size hint only. |

**codebase_name.ios / android:** all `null` on all three — codebase-review round was skipped in this session. Coding agent must fill in the app's actual component names (see [Open items I-2](#open-items)).

---

## 5 · Behaviors

| id | on | action | target / then | source |
|---|---|---|---|---|
| **BH-1** | `news_card_tap` | `open_in_app_webview` | `data.url` from article record | designer sign-off 2026-09-22 (Q3 Recommended) |
| **BH-2** | `xem_tat_ca_tap` | `open_route` | **TBD** — see [I-3](#open-items) | PRD US5 AC-5 |
| **BH-3** | `xem_them_tap` | `request_next_page` | target `project_ad_list_pagination`; then hide `load_more_control#idle`, show `load_more_control#loading` | design F0 → F1 diff |
| **BH-4** | `loading_more_pages` | render | `load_more_control#loading` | design F1 |
| **BH-5** | `pagination_success_with_more` | `swap_variant` | from `loading` → `idle` | designer sign-off (Q3 Recommended) |
| **BH-6** | `pagination_success_no_more` | `hide_section` | entire `load_more_control` (DR-3) | designer sign-off (Q3 Recommended) |
| **BH-7** | `pagination_failure` | `toast_and_restore` | toast text `"Không thể tải thêm dự án"` (copy TBC — I-4), duration 2000ms; then swap variant `loading` → `idle` | designer sign-off (Q3 Recommended — DR-4) |

---

## 6 · Data rules

| rule | value | source | consumed by |
|---|---|---|---|
| `news_source_url` | `https://www.nhatot.com/kinh-nghiem/search/Nhà+ở+xã+hội` | PRD US5 AC-6 | `sections/news_hub_noxh#cards-row.data.source` |
| `news_card_tap_url` | `data.url` per article (behaviour BH-1) | PRD | `sections/news_hub_noxh#card` |
| `news_card_min_count_for_render` | `1` — if source returns 0, hide entire section (FR-7). No empty state. | PRD US5 AC-7 | `sections/news_hub_noxh` |
| `news_card_date_format` | `dd/MM/yyyy` (example `"26/05/2026"`). Design copy has a trailing space — strip it in code. | design text `"26/05/2026 "` | `sections/news_hub_noxh#card.date` |
| `news_card_headline_truncation` | `max_lines: 2`, method `end-ellipsis` | design `style.maxLines=2`, `textTruncation=ENDING` | `sections/news_hub_noxh#card.headline` |
| `news_thumbnail_placeholder` | Use a placeholder image when article has no `thumbnail_url` or the fetch fails. The design shows all 4 cards with the same imageRef `7045a38c5c75f53a19cd805b2b203d55b9debe71` (sample). | design | `sections/news_hub_noxh#card.thumbnail` |

---

## 7 · Display rules

- **`news_section_hide_when_empty`** — condition: `article_count == 0` → action: `hide_section`, target `sections/news_hub_noxh`. Source: PRD US5 AC-7. Do not render an empty state, do not render the title row alone, do not reserve layout space.
- **`load_more_hide_when_no_more_pages`** — condition: `hasMorePages == false` → action: `hide_section`, target `sections/load_more_control`. Do not render a disabled button.
- **`load_more_variant_selection`** — target `sections/load_more_control`:
  - `hasMorePages == false` → variant `none` (hidden — see above)
  - `loadingMore == true` → variant `loading`
  - `hasMorePages == true AND loadingMore == false` → variant `idle`

---

## Token caveat

**Status: degraded.** The PAT used for this run had scope `File content: read-only`. `File variables: read` was not granted.

**Impact.** Every non-image `fills`, `strokes`, padding, `itemSpacing`, `rectangleCornerRadii`, `fontFamily`, `fontSize`, `fontWeight`, `lineHeight`, `letterSpacing` on every node in the two new sections IS bound to a Figma variable — verified per-node via `boundVariables` on REST `/nodes`. However, the variable's semantic NAME (`--color-text-primary`, `--space-md`, etc.) is not resolvable in this session.

**What is in the spec.** Every affected leaf carries a `token_hint` line stating `[UNVERIFIED — file_variables scope missing]` plus the expected DS category to guide the coding agent. The raw hex / raw px value is written verbatim into `style:` so nothing renders wrong even if the token substitution is deferred.

**Coding agent action.** Do **not** ship the raw hex / raw px values as constants. Replace each `token_hint` with the correct DS token from the app's stylesheet at build time. If a token cannot be found, raise a follow-up rather than baking the raw value in.

**How to upgrade this spec.** Generate a new Figma PAT with `File variables: read` scope, or run the `get_variable_defs` Figma MCP endpoint against the three frames `20253:23908`, `20253:24441`, `20253:23910`, and re-run the handoff. The token names replace every `[UNVERIFIED]` line without any structural change to the spec.

---

## Open items

None are blocking; every item has a stated `until_then` so the build can proceed.

- **I-1 · PRD update — remove `read_time` from FR-3.** PRD US5 AC-3 says each card shows `"thumbnail image, headline (truncated if too long), date, and read time"`. Design omits read_time and the designer confirmed the DESIGN is correct on 2026-09-22 (Q1). Owner: PM (buyer actions). Until then: coding agent implements without read_time per this spec.
- **I-2 · Codebase-review round was not run.** The pipeline expects a code-aware agent to map each `spec_id` to its codebase-side name per platform after scope sign-off. That round did not run in this session. Consequently `shared/ds_components.yaml#codebase_name.ios/android` are `null` and the Figma layers were NOT renamed. Owner: implementing engineer + engineering-side reviewer. Until then: coding agent picks the app's existing DS components; update names when found.
- **I-3 · "Xem tất cả →" target route not defined.** Options: (a) open the source URL directly in in-app WebView, (b) open an in-app "News list" screen that fetches from the same source. Owner: PM. Until then: coding agent uses (a) and surfaces the route as a config so it can flip to (b) without a redeploy.
- **I-4 · Load-fail toast copy needs confirmation.** BH-7 uses placeholder copy `"Không thể tải thêm dự án"`. Owner: content ops / PM.
- **I-5 · Figma variable name resolution — spec is degraded.** See [Token caveat](#token-caveat). Owner: design ops.
- **I-6 · Design owner unknown.** No `Handoff info` frame on the section canvas. Not required for build; fill at the next handoff pass.

---

## Design warnings (non-blocking hygiene)

Non-blocking findings on the design source — do NOT prevent the coding agent from building. Filed so the designer can act on them for the next iteration.

- **W-1 · 7 generic Figma names.** `Frame 2085668390`, `Frame 2085668046`, `Frame 4`, `Frame 2085668415`, `Frame 2085668416`, `Frame 2085667675`, `Frame 2085667676`. Proposed renames listed alongside each spec_id in the tables above.
- **W-2 · Duplicate layer names.** `project ad` × 5 per frame · `Main info` × 4 per frame (should be `news_card`) · `Item Info` × 4 · `Div [flex]` × 4 · `Rectangle 240648306` × 4 (Figma-imported vestigial name).
- **W-3 · News section outer container is named `Thông số`** ("stats/specs") in Figma — should be `[SEC] news_hub_noxh`. Called out in the section tree.
- **W-4 · Spinner is a raw group named `Group 7 Copy 6`** — proposed rename `spinner_loading`. Nothing in the layer tree says it is a loading spinner.
- **W-5 · No annotations on canvas.** The Chotot convention places `[B]/[R]/[E]` legend text above each frame; the "Blog spot" section has none. On this run PRD covered most of what an annotation would say. On thinner-PRD features this pattern would send the design back for a re-do.
- **W-6 · Spinner is not a DS component instance.** If the DS has a `Loading` / `Spinner` / `ActivityIndicator`, use it in the source. Coding agent uses the platform-default spinner per designer sign-off, so the produced UI is correct — but the Figma source will drift from the DS.
- **W-7 · Sample data.** All 4 news cards share the same imageRef `7045a38c5c75f53a19cd805b2b203d55b9debe71`. No impact on build (data is data-bound); flag only so no one mistakes it for a per-slot fill.

---

# Appendix A · Phase A readiness report

**Overall readiness: READY WITH CAVEATS.** Proceed to build with the caveats above surfaced as `open_items`, not as absent design.

## Audit A — Layer & Naming
Moderate defects, all fixable in rename pass. 7 generic Figma-generated names (W-1); duplicate names (W-2); wrong-semantic name `Thông số` (W-3); unlabelled state `Group 7 Copy 6` (W-4). None blocking.

## Audit B — Annotations
No annotations exist. Section has zero top-level TEXT children; no annotation frames sit alongside either NOXH HUB frame. PRD covered most of what these would have said; four INTENT questions batched in the sign-off round covered the rest.

## Audit C — Flow organisation
Pass. Two frames sit side-by-side left → right on the canvas at same Y, same width, same top structure — natural "before → after tapping Xem thêm" progression. Both frames semantically named `NOXH HUB` (correct — two states of one screen).

## Audit D — DS Compliance
Pass with two flags.
- Strong signals — every button is a DS instance with correct variant props; top nav, filters, search bar, tabs are DS instances; every typography field is a `VARIABLE_ALIAS`; every non-image fill has `boundVariables.fills`; strokes, itemSpacing, padding*, corner radii all bound where checked.
- Flags — spinner is a raw group not a DS instance (W-6); token names unverifiable due to PAT scope (I-5).

No Tier 1 blocking: no unbound raw hex on non-instance content, no wrong-variant DS usage, no absolute-positioned free colours.

---

# Appendix B · Scope Manifest (as signed)

Signed 2026-09-22 by thaoduong@chotot.vn.

### Bảng A — Requirements

| id | requirement | source | evidence | conflict? | action |
|---|---|---|---|---|---|
| FR-1 | Display "Tin tức Nhà ở Xã hội" on Dự án tab, below project ads | PRD buyer-actions US5 AC-1 · hub-detail US2 | design F0/F1 layout.6 | — | build |
| FR-2 | 4+ articles as horizontally scrollable cards | PRD US5 AC-2 | Frame 2085667675 `HORIZONTAL_SCROLLING`, 4×232 + 3×8 = 952px in 343px container | — | build |
| FR-3 | Each card = thumbnail + headline (truncated) + date **+ read time** | PRD US5 AC-3 | design has thumbnail + headline (2-line ENDING) + date; **read_time NOT drawn** | ⚠️ PRD vs design | design wins (Q1) — see I-1 |
| FR-4 | Tap card → opens article | PRD US5 AC-4 | no tap-target annotation; card is a plain FRAME | — | build (in-app WebView per Q3) |
| FR-5 | "Xem tất cả →" opens full article list | PRD US5 AC-5 | Button 20253:23913 Ghost S-24px isPill | — | build |
| FR-6 | Articles sourced from `nhatot.com/kinh-nghiem/search/Nhà+ở+xã+hội` | PRD US5 AC-6 | data-layer rule | — | reference (Data rules) |
| FR-7 | 0 articles → hide entire section | PRD US5 AC-7 | no empty state drawn | — | rule (Display rules) |
| DR-1 | Load-more UX: idle button / loading spinner | design F0 vs F1 | Button M-32px Tertiary Pill vs raw 36×36 shape group | — | build (both states) |
| DR-2 | Control centred, 12px vertical padding | design | Frame 2085668415/416 padding=[12,12,12,12] alignItems=CENTER | — | build |

### Bảng B — Behaviours (see [§5 Behaviors](#5--behaviors) for full detail)

BH-1 tap news card → open article · BH-2 tap Xem tất cả → open list · BH-3 tap Xem thêm → load next page · BH-4 loading → show spinner · BH-5 success + more → restore button · BH-6 success + no more → hide button · BH-7 fail → toast + restore.

All 7 wired to at least one node in the section trees (verified: `Behaviors declared=[BH-1..BH-7], used=[BH-1..BH-7], missing=[]`).

### Bảng C — Layout blocks

See [§1 Full screen composition](#full-screen-composition-context--only-2-sections-need-building) above. 2 sections built (`load_more_control`, `news_hub_noxh`); 7 sections keep-as-is; 1 section OUT (OS chrome).

### Bảng D — Content rules

See [§6 Data rules](#6--data-rules).

### Bảng T — Truncation & overflow

| element | rule | source |
|---|---|---|
| News card headline | maxLines 2, `textTruncation: ENDING`, font 14 SemiBold Reddit Sans, lineHeight 20 | REST `style` on `20253:23919` |
| News card date | single line, no truncation stated | REST |
| Cards row | horizontal scroll, no clip | Frame 2085667675 `HORIZONTAL_SCROLLING`, `clipsContent=false` |
| Section container `Thông số` | fixed 302px (does not grow with content) | `absoluteBoundingBox.height=302` |
| "Xem tất cả →" | fits 88px width, no truncation | text 72×18 in 88px button |
| "Xem thêm" | fits 92px width, no truncation | text 66×20 in 92px button |

---

**Attribution.** Handoff generated by Claude (Opus 4.7) via `/design-chotot:ready-to-handoff` on 2026-09-22 for thaoduong@chotot.vn.
