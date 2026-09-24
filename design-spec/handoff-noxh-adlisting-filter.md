# Handoff spec — NOXH filter on PTY Ad Listing

| | |
|---|---|
| Feature | `pty_noxh_adlisting_filter` · bundle format 2.1, single-file mode |
| PRD | `carousell/ct-product-planning` · `specs/vertical/pty/noxh-add-filter-on-adlisting/product-spec.md` (draft 2026-09-24, driver Khoa Trinh) |
| Figma | file `vjLuAM8l0t1MyVjgDX3Y4m`, section `1732:9886` "NOXH filter · PTY Adlisting" |
| Design owner | Thảo Dương |
| Platforms | iOS · Android · Web (desktop + msite). **iOS/Android reuse the Mobile frames** for the entry tile row and the filter sheet; the App frame `1748:15197` defines the destination state |
| Sources read | Figma REST `/nodes` (full subtree, re-read after rename) · `/v1/images` · Figma MCP `get_variable_defs` · bound variable IDs resolved to names in Figma. `token_hint` = the variable/style **actually bound** on that node (none → no hint); raw CSS in `style:` is ground truth |
| Assets | none shipped (.md mode). Icons are named or marked `glyph` (existing app asset); tile icons and value labels are **server data** (`data:`). **NOXH tile icon: export from Figma** node `1748:5114` (desktop) / `1748:5145` (mobile). It is final |
| Gates | coverage_check 14/14 automatic gates PASS · 54/54 display strings · 573/573 measurements match Figma |

## How to read this file
- Section 1 is the contract: requirements, acceptance criteria, behaviours, variants and display rules.
- Section 2 holds the layout trees: one fenced YAML block per section, state and overlay, per platform. `style:` is verbatim CSS from Figma. `token_hint:` is the variable bound in Figma — see OI-TOKEN-ROLE before turning a hint into code: some are semantically wrong (an Icon token on a background).
- `id:` equals the Figma layer name (layers were renamed 2026-09-24). Use `node_id` to trace back to Figma.
- `behavior:` on a node refers to §1.3. `BH-EXISTING` means an existing control whose handler is unchanged.
- Anything with `change: NEW` is the new UI. Everything else is existing UI, shown for placement context and to be kept as is.

# 1. Contract
## 1.0 Terminology
| Term | Means | Where |
|---|---|---|
| **Tile** | The NOXH entry point in the **contextual filter**: 5th, last item of the sub-category suggestion row (icon + label). Tap → Ad Listing NOXH | `sec_subcate_suggestion` · `noxh_tile_*` |
| **Chip** | The **"Loại hình căn hộ"** control in the **filter bar**. Tap → opens the value list: **popover** on desktop, **bottom sheet** on mobile (msite, iOS, Android) | `sec_filter_bar` → `ovl_apartment_type` |
| **Applied chip** | `Nhà ở xã hội ×` in the filter bar after the filter is applied; × removes it | state `noxh_applied` |

## 1.1 Requirements
| ID | Requirement | Where |
|---|---|---|
| FR-1.1 | Rename value "Chung cư" → **"Chung cư thương mại"** everywhere (filter list, selected-filter chip, ad card, ad detail). "Chung cư" must not appear anywhere | overlay `ovl_apartment_type` · server labels |
| FR-1.2 | Add value **"Nhà ở xã hội"** to param `apartment_type` (UI label **"Loại hình căn hộ"**) for subcate 1010 | overlay `ovl_apartment_type` |
| FR-1.3 | **Order: 1 Chung cư thương mại · 2 Nhà ở xã hội**, then the existing values Duplex · Penthouse · Căn hộ dịch vụ, mini · Tập thể, cư xá · Officetel (Figma, updated by the designer 2026-09-24; matches the PRD) | overlay |
| FR-1.4 | Value with 0 ads is hidden (existing inventory rule). Select/reset/selected-display behaviour unchanged | overlay |
| FR-2.1 | Entry tile **"Nhà ở xã hội"** as the **5th, last** tile of the sub-category suggestion row, on Ad Listing PTY with **no sub-category selected** | `sec_subcate_suggestion` |
| FR-2.2 | Inventory threshold: show iff NOXH count in current context (location + applied filters) **≥ 1**; 0 → not rendered, no gap; recompute on every context change; count request error → hide (fail-closed) | `BH-NOXH-TILE-VISIBILITY` |
| FR-2.3 | Tap → Ad Listing subcate 1010 + `apartment_type`=Nhà ở xã hội; keep location, ad type, other filters | `BH-NOXH-TILE-TAP` |
| DR-1 | Destination shows applied chip **"Nhà ở xã hội ×"** after "Căn hộ/Chung cư ×" in the filter bar (design requirement, not in the PRD) | state `noxh_applied` |
| FR-3 | NOXH = subcate 1010 AND apartment_type = Nhà ở xã hội; migrate all "Chung cư" ads (Chợ Tốt-posted NOXH list → NOXH; everything else → Chung cư thương mại). **Backend only, no UI** | open item OI-FR3 |

## 1.2 Acceptance criteria
- **AC-FR1-1** Subcate Căn hộ chung cư → open "Loại hình căn hộ": the list shows "Chung cư thương mại" then "Nhà ở xã hội" (then the existing values), and no "Chung cư".
- **AC-FR1-2 / 1-3** Selecting Nhà ở xã hội (or Chung cư thương mại) returns only ads with that value.
- **AC-FR1-4** A migrated ad shows the new label on the ad card and ad detail (server-resolved label, never the raw code).
- **AC-FR2-1** No subcate selected + ≥ 1 NOXH ad in context → the tile is shown at position 5.
- **AC-FR2-2** Exactly 1 NOXH ad → the tile is still shown.
- **AC-FR2-3** 0 NOXH ads → no tile and no gap (4 tiles, same spacing).
- **AC-FR2-4** Tap the tile → Ad Listing 1010 with "Loại hình căn hộ = Nhà ở xã hội" selected; location and previous filters kept.
- **AC-FR2-5** Change to a location with 0 NOXH → the tile hides; change back to ≥ 1 → it reappears.
- **AC-FR2-6** A sub-category is selected → no tile.
- **AC-EC-01** Count = 1 → both the tile and the filter value are shown; count = 0 → both hidden.
- **AC-EC-02** Count drops to 0 before the tap → the existing Ad Listing empty state.
- **AC-EC-03** Back after entering from the tile → the root listing with the previous context.
- **AC-EC-04** Old saved search / deep link with "Chung cư" → treated as Chung cư thương mại (pending PRD Q2, OI-Q2).
- **AC-EC-05** Switching ad type (Mua bán ↔ Cho thuê) resets the NOXH filter (existing logic).

## 1.3 Behaviours
```yaml
BH-NOXH-TILE-TAP:
  trigger: tap
  on_tap: Push Ad Listing with subcate=1010 (Căn hộ chung cư) + apartment_type=Nhà ở xã hội; keep location, ad type and all
    other applied filters
  navigates_to: PTY.adlisting_apartment/noxh_applied
  scope: new
  traces_to:
  - FR-2.3
  platforms:
  - ios
  - android
  - web
BH-NOXH-TILE-VISIBILITY:
  trigger: render + every context change (location, filters)
  rule: Show tile iff no sub-category selected AND NOXH count in current context >= 1. Count = 0, count request fails (fail-closed),
    or a sub-category is selected -> tile not rendered, no empty gap (row collapses to the 4 existing tiles).
  scope: new
  traces_to:
  - FR-2.2
  - PRD 5.1
BH-NOXH-CHIP-REMOVE:
  trigger: tap x on applied chip
  on_tap: Remove apartment_type filter -> Ad Listing 1010 with no apartment-type filter
  scope: keep_as_is
  traces_to:
  - PRD 5.1 Remove filter
BH-BACK:
  trigger: system back / back arrow after entering from the tile
  result: Return to root Ad Listing (no sub-category) with the previous location/filter context
  scope: keep_as_is
  traces_to:
  - AC-EC-03
BH-ADTYPE-RESET:
  trigger: switch Mua bán <-> Cho thuê
  result: apartment_type filter (incl. Nhà ở xã hội) resets — existing logic
  scope: keep_as_is
  traces_to:
  - AC-EC-05
BH-FILTER-OPEN:
  trigger: tap the "Loại hình căn hộ" chip in the filter bar
  opens: ovl_apartment_type
  scope: keep_as_is
BH-FILTER-SELECT:
  trigger: tap a value row / chip
  result: Toggle selection (desktop checkbox, mobile chip). Xoá lọc clears, Áp dụng applies — unchanged
  scope: keep_as_is
  traces_to:
  - FR-1
  figma_evidence: prototype interactions on 1732:5372-5378, 1732:9844..9881
BH-EXISTING:
  trigger: tap
  result: Existing control — handler unchanged by this PRD (keep_as_is)
  scope: keep_as_is
BH-OVERLAY-CLOSE:
  trigger: tap X / scrim / back
  result: Dismiss without applying
  scope: keep_as_is
```

## 1.4 Variants & states
| Owner | Variant | When | Tree |
|---|---|---|---|
| `sec_subcate_suggestion` | `with_noxh` | no subcate AND NOXH count ≥ 1 | layout as drawn (5 tiles) |
| `sec_subcate_suggestion` | `hidden` | count = 0 OR count error OR subcate selected | layout without `noxh_tile_*` (4 tiles, row gap 8px unchanged) |
| `sec_filter_bar` | `default` | no apartment_type applied | §2.2 |
| `sec_filter_bar` | `noxh_applied` | apartment_type = Nhà ở xã hội | §2.3 (screen state, desktop + app) |
| `ovl_apartment_type` | `default` | nothing selected | §2.4 (desktop popover, mobile bottom sheet) |

## 1.5 Display rules
| id | Element | Rule |
|---|---|---|
| D-TILE-LABEL | tile label (`*_tile_label_*`) | Server label. **Do not copy the hard line break** in the Figma sample `"Nhà ở \nxã hội"`; let it wrap naturally within the tile width (desktop 76px, mobile 56px) |
| T-TILE-LABEL | tile label | Max **2 lines, clipped** (label box is a fixed 42px desktop / 36px mobile with `overflow: hidden`), same as the existing tiles |
| T-APPLIED-CHIP | `noxh_applied_chip_label_*` | 1 line, chip hugs its content (no clamp) |
| T-VALUE-ROW | list rows / chips | DS List / Chip default |
| D-VALUES | option labels | Server-driven, in server order; client never maps codes to labels |

## 1.6 Open items
```yaml
- id: OI-ICON
  owner: Dev
  kind: resolved
  text: 'Resolved 2026-09-24 (designer): the NOXH tile icon in Figma is final. Dev exports it from Figma — desktop node noxh_tile_icon_dt
    (1748:5114), mobile node noxh_tile_icon_mw (1748:5145) — and uploads it to the sub-category suggestion config like the
    other tile icons.'
- id: OI-FR3
  owner: PM + Ops
  kind: blocking_backend
  text: PRD Q1 — source of the Chợ Tốt-posted NOXH ad list for migration
- id: OI-Q2
  owner: PM + Eng lead
  kind: before_build
  text: PRD Q2 — saved search / deep link / SEO URL with old "Chung cư" value (AC-EC-04)
- id: OI-PRD
  owner: PM (Khoa Trinh)
  kind: doc
  text: 'Update PRD wording: (1) the FR-2 entry point in the contextual filter is a TILE in the sub-category row (PRD currently
    says chip); (2) the CHIP is the "Loại hình căn hộ" control in the filter bar, which opens a popover (desktop) or bottom
    sheet (mobile); (3) param label is "Loại hình căn hộ" (PRD says Loại hình chung cư); (4) add the Figma link.'
- id: OI-TOKENS
  owner: —
  kind: resolved
  text: 'Resolved 2026-09-24: every token_hint is the variable/style actually bound on that node (REST boundVariables → names
    resolved in Figma, cross-checked with MCP get_variable_defs). Nodes without a binding carry no token_hint — raw CSS only.'
- id: OI-TOKEN-ROLE
  owner: Design (Thảo Dương)
  kind: before_qc
  text: 'Real bindings show 19 nodes with an Icon/* token on a background (incl. NEW noxh_applied_chip_*: Icon/icon-on-background
    / Icon/icon-primary — same as the existing Căn hộ/Chung cư × chip) and 76 text nodes with no text style (size/weight/line-height
    bound from different type groups, incl. noxh_tile_label_* — same as existing tiles). Frames were imported from HTML and
    auto-bound by value.'
  until_then: Build the applied chip with the EXISTING selected-filter chip component in code, and tile labels with the existing
    tile label style. Raw CSS values are correct; do not create new tokens from these names.
- id: OI-OUT
  owner: —
  kind: decision
  text: Listing body, SEO block, ad cards, sidebar, bottom nav excluded per scope confirm 2026-09-24 (unchanged by this PRD)
- id: OI-APP
  owner: —
  kind: decision
  text: iOS/Android reuse mobile-web frames for entry tile row and filter sheet (designer, 2026-09-24)
```

## 1.7 Name map
| spec_id | Figma layer | node | Code (per platform) |
|---|---|---|---|
| `PTY.adlisting_root` | `[SCR] PTY.adlisting_root — Desktop` / `— Msite` | `1732:5061` / `1732:9159` | unknown — codebase not reviewed |
| `sec_subcate_suggestion` | `sec_subcate_suggestion_dt` / `_mw` | `1732:5124` / `1732:9236` | unknown |
| `PTY.adlisting_apartment` | `[SCR] PTY.adlisting_apartment — Desktop` / `— Msite` | `1732:5222` / `1732:9404` | unknown |
| `sec_filter_bar` | `sec_filter_bar_dt` / `_mw` | `1732:5254` / `1732:9434` | unknown |
| `PTY.adlisting_apartment/noxh_applied` | `sec_filter_bar_noxh_applied_dt` / `sec_filter_bar_app` | `1748:8561` / `1748:15616` | unknown |
| `ovl_apartment_type` | `[OVL] ovl_apartment_type_popover — Desktop` / `[OVL] ovl_apartment_type_drawer — Mobile` | `1732:5361` / `1732:9372` | unknown |

# 2. Layout trees

## 2.1 Screen `PTY.adlisting_root` › section `sec_subcate_suggestion`

### web_desktop — `sec_subcate_suggestion_dt` (Figma `1732:5124`)

```yaml
id: sec_subcate_suggestion_dt
spec_id: sec_subcate_suggestion
kind: section
figma_node: 1732:5124
platform: web_desktop
requirement:
  traces_to:
  - FR-2.1
  - FR-2.2
  - FR-2.3
  context: Ad Listing PTY (category Bất động sản) with NO sub-category selected. Existing row of sub-category suggestion tiles;
    add a 5th, last tile "Nhà ở xã hội".
  acceptance:
  - AC-FR2-1 no subcate + ≥1 NOXH ad in context → tile shown
  - AC-FR2-2 exactly 1 NOXH ad → tile still shown
  - AC-FR2-3 0 NOXH ads → tile absent, no gap
  - AC-FR2-4 tap → Ad Listing 1010 + Loại hình căn hộ = Nhà ở xã hội, location & other filters kept
  - AC-FR2-5 change location to 0 NOXH → hidden; back to ≥1 → shown
  - AC-FR2-6 sub-category selected → tile not shown
  - AC-EC-01/02/03 (see README)
uses:
  behaviors:
  - BH-EXISTING
  - BH-NOXH-TILE-TAP
  - BH-NOXH-TILE-VISIBILITY
variants:
  with_noxh:
    when: no subcate selected AND NOXH count ≥ 1
    tree: = layout (5 tiles)
  hidden:
    when: NOXH count = 0 OR count error OR subcate selected
    tree: layout minus node noxh_tile_dt (4 tiles; row gap unchanged, no empty slot)
layout:
  id: sec_subcate_suggestion_dt
  node_id: 1732:5124
  tag: div
  figma_type: FRAME
  figma_name: sec_subcate_suggestion_dt
  style:
    display: flex
    flex-direction: row
    gap: 8px
    align-items: flex-start
    width: 100%
    flex-shrink: 0
    height: 102px
    overflow: hidden
    position: relative
  children:
  - id: subcate_tile_dt
    node_id: 1732:5125
    tag: div
    figma_type: FRAME
    figma_name: subcate_tile_dt
    style:
      display: flex
      flex-direction: column
      gap: 8px
      align-items: center
      padding: 0px 4px 0px 4px
      width: 84px
      flex-shrink: 0
      align-self: stretch
      min-width: 84px
    children:
    - id: subcate_tile_icon_frame_dt
      node_id: 1732:5126
      tag: div
      figma_type: FRAME
      figma_name: subcate_tile_icon_frame_dt
      style:
        display: flex
        flex-direction: row
        justify-content: center
        align-items: center
        width: 52px
        flex-shrink: 0
        height: 52px
      graphic: true
      token_hint:
        border-width: Stroke/stroke-divider
      data:
        source: server subcate-suggestion config (icon url)
        note: 'category illustration served with the config, not a DS icon. NOXH icon: export from this Figma node (final,
          OI-ICON)'
      asset_mode: none — see icon/data
    - id: subcate_tile_label_box_dt
      node_id: 1732:5144
      tag: div
      figma_type: FRAME
      figma_name: subcate_tile_label_box_dt
      style:
        display: flex
        flex-direction: column
        align-items: center
        width: 100%
        height: 42px
        overflow: hidden
      children:
      - id: subcate_tile_label_dt
        node_id: 1732:5145
        tag: p
        figma_type: TEXT
        figma_name: subcate_tile_label_dt
        style:
          width: 100%
          height: auto
          color: '#595959'
          font-family: '''Reddit Sans'''
          font-size: 14px
          font-weight: 400
          line-height: 21px
          text-align: center
        text: Căn hộ/Chung cư
        token_hint:
          color: Text/text-secondary
          letter-spacing: Display/Price/display-price-letter-spacing
          font-weight: Body/Page/body-page-font-weight
          font-family: Font family/font-family
          font-size: Tagline/Caption/tagline-caption-font-size
        data:
          source: server subcate-suggestion config (label)
          note: server-driven; client does not hardcode. Figma sample has a hard line break — do NOT copy it (content rule
            D-TILE-LABEL)
      token_hint:
        border-width: Stroke/stroke-divider
    token_hint:
      gap: Spacing/gap-x-small
      padding-left: Padding/padding-2x-small
      padding-right: Padding/padding-2x-small
      row-gap: Spacing/gap-x-small
      border-width: Stroke/stroke-divider
    behavior: BH-EXISTING
    change: existing tile, unchanged
  - id: subcate_tile_2_dt
    node_id: 1732:5146
    tag: div
    figma_type: FRAME
    figma_name: subcate_tile_2_dt
    style:
      display: flex
      flex-direction: column
      gap: 8px
      align-items: center
      padding: 0px 4px 0px 4px
      width: 84px
      flex-shrink: 0
      align-self: stretch
      min-width: 84px
    children:
    - id: subcate_tile_2_icon_frame_dt
      node_id: 1732:5147
      tag: div
      figma_type: FRAME
      figma_name: subcate_tile_2_icon_frame_dt
      style:
        display: flex
        flex-direction: row
        justify-content: center
        align-items: center
        width: 52px
        flex-shrink: 0
        height: 52px
      graphic: true
      token_hint:
        border-width: Stroke/stroke-divider
      data:
        source: server subcate-suggestion config (icon url)
        note: 'category illustration served with the config, not a DS icon. NOXH icon: export from this Figma node (final,
          OI-ICON)'
      asset_mode: none — see icon/data
    - id: subcate_tile_2_label_box_dt
      node_id: 1732:5167
      tag: div
      figma_type: FRAME
      figma_name: subcate_tile_2_label_box_dt
      style:
        display: flex
        flex-direction: column
        align-items: center
        width: 100%
        height: 21px
        overflow: hidden
      children:
      - id: subcate_tile_2_label_dt
        node_id: 1732:5168
        tag: p
        figma_type: TEXT
        figma_name: subcate_tile_2_label_dt
        style:
          width: fit-content
          flex-shrink: 0
          height: auto
          color: '#595959'
          white-space: nowrap
          font-family: '''Reddit Sans'''
          font-size: 14px
          font-weight: 400
          line-height: 21px
          text-align: center
        text: Nhà ở
        token_hint:
          color: Text/text-secondary
          letter-spacing: Display/Price/display-price-letter-spacing
          font-weight: Body/Page/body-page-font-weight
          font-family: Font family/font-family
          font-size: Tagline/Caption/tagline-caption-font-size
        data:
          source: server subcate-suggestion config (label)
          note: server-driven; client does not hardcode. Figma sample has a hard line break — do NOT copy it (content rule
            D-TILE-LABEL)
      token_hint:
        border-width: Stroke/stroke-divider
    token_hint:
      gap: Spacing/gap-x-small
      padding-left: Padding/padding-2x-small
      padding-right: Padding/padding-2x-small
      row-gap: Spacing/gap-x-small
      border-width: Stroke/stroke-divider
    behavior: BH-EXISTING
    change: existing tile, unchanged
  - id: subcate_tile_3_dt
    node_id: 1732:5169
    tag: div
    figma_type: FRAME
    figma_name: subcate_tile_3_dt
    style:
      display: flex
      flex-direction: column
      gap: 8px
      align-items: center
      padding: 0px 4px 0px 4px
      width: 84px
      flex-shrink: 0
      align-self: stretch
      min-width: 84px
    children:
    - id: subcate_tile_3_icon_frame_dt
      node_id: 1732:5170
      tag: div
      figma_type: FRAME
      figma_name: subcate_tile_3_icon_frame_dt
      style:
        display: flex
        flex-direction: row
        justify-content: center
        align-items: center
        width: 52px
        flex-shrink: 0
        height: 52px
      graphic: true
      token_hint:
        border-width: Stroke/stroke-divider
      data:
        source: server subcate-suggestion config (icon url)
        note: 'category illustration served with the config, not a DS icon. NOXH icon: export from this Figma node (final,
          OI-ICON)'
      asset_mode: none — see icon/data
    - id: subcate_tile_3_label_box_dt
      node_id: 1732:5186
      tag: div
      figma_type: FRAME
      figma_name: subcate_tile_3_label_box_dt
      style:
        display: flex
        flex-direction: column
        align-items: center
        width: 100%
        height: 21px
        overflow: hidden
      children:
      - id: subcate_tile_3_label_dt
        node_id: 1732:5187
        tag: p
        figma_type: TEXT
        figma_name: subcate_tile_3_label_dt
        style:
          width: fit-content
          flex-shrink: 0
          height: auto
          color: '#595959'
          white-space: nowrap
          font-family: '''Reddit Sans'''
          font-size: 14px
          font-weight: 400
          line-height: 21px
          text-align: center
        text: Đất
        token_hint:
          color: Text/text-secondary
          letter-spacing: Display/Price/display-price-letter-spacing
          font-weight: Body/Page/body-page-font-weight
          font-family: Font family/font-family
          font-size: Tagline/Caption/tagline-caption-font-size
        data:
          source: server subcate-suggestion config (label)
          note: server-driven; client does not hardcode. Figma sample has a hard line break — do NOT copy it (content rule
            D-TILE-LABEL)
      token_hint:
        border-width: Stroke/stroke-divider
    token_hint:
      gap: Spacing/gap-x-small
      padding-left: Padding/padding-2x-small
      padding-right: Padding/padding-2x-small
      row-gap: Spacing/gap-x-small
      border-width: Stroke/stroke-divider
    behavior: BH-EXISTING
    change: existing tile, unchanged
  - id: subcate_tile_4_dt
    node_id: 1732:5188
    tag: div
    figma_type: FRAME
    figma_name: subcate_tile_4_dt
    style:
      display: flex
      flex-direction: column
      gap: 8px
      align-items: center
      padding: 0px 4px 0px 4px
      width: 84px
      flex-shrink: 0
      align-self: stretch
      min-width: 84px
    children:
    - id: subcate_tile_4_icon_frame_dt
      node_id: 1732:5189
      tag: div
      figma_type: FRAME
      figma_name: subcate_tile_4_icon_frame_dt
      style:
        display: flex
        flex-direction: row
        justify-content: center
        align-items: center
        width: 52px
        flex-shrink: 0
        height: 52px
      graphic: true
      token_hint:
        border-width: Stroke/stroke-divider
      data:
        source: server subcate-suggestion config (icon url)
        note: 'category illustration served with the config, not a DS icon. NOXH icon: export from this Figma node (final,
          OI-ICON)'
      asset_mode: none — see icon/data
    - id: subcate_tile_4_label_box_dt
      node_id: 1732:5212
      tag: div
      figma_type: FRAME
      figma_name: subcate_tile_4_label_box_dt
      style:
        display: flex
        flex-direction: column
        align-items: center
        width: 100%
        height: 42px
        overflow: hidden
      children:
      - id: subcate_tile_4_label_dt
        node_id: 1732:5213
        tag: p
        figma_type: TEXT
        figma_name: subcate_tile_4_label_dt
        style:
          width: 100%
          height: auto
          color: '#595959'
          font-family: '''Reddit Sans'''
          font-size: 14px
          font-weight: 400
          line-height: 21px
          text-align: center
        text: Văn phòng, Mặt bằng kinh doanh
        token_hint:
          color: Text/text-secondary
          letter-spacing: Display/Price/display-price-letter-spacing
          font-weight: Body/Page/body-page-font-weight
          font-family: Font family/font-family
          font-size: Tagline/Caption/tagline-caption-font-size
        data:
          source: server subcate-suggestion config (label)
          note: server-driven; client does not hardcode. Figma sample has a hard line break — do NOT copy it (content rule
            D-TILE-LABEL)
      token_hint:
        border-width: Stroke/stroke-divider
    token_hint:
      gap: Spacing/gap-x-small
      padding-left: Padding/padding-2x-small
      padding-right: Padding/padding-2x-small
      row-gap: Spacing/gap-x-small
      border-width: Stroke/stroke-divider
    behavior: BH-EXISTING
    change: existing tile, unchanged
  - id: noxh_tile_dt
    node_id: 1732:5214
    tag: div
    figma_type: FRAME
    figma_name: noxh_tile_dt
    style:
      display: flex
      flex-direction: column
      gap: 8px
      align-items: center
      padding: 0px 4px 0px 4px
      width: 84px
      flex-shrink: 0
      align-self: stretch
      min-width: 84px
    children:
    - id: noxh_tile_icon_frame_dt
      node_id: 1732:5215
      tag: div
      figma_type: FRAME
      figma_name: noxh_tile_icon_frame_dt
      style:
        display: flex
        flex-direction: row
        justify-content: center
        align-items: center
        width: 52px
        flex-shrink: 0
        height: 52px
      graphic: true
      token_hint:
        border-width: Stroke/stroke-divider
      data:
        source: server subcate-suggestion config (icon url)
        note: 'category illustration served with the config, not a DS icon. NOXH icon: export from this Figma node (final,
          OI-ICON)'
      asset_mode: none — see icon/data
    - id: noxh_tile_label_box_dt
      node_id: 1732:5220
      tag: div
      figma_type: FRAME
      figma_name: noxh_tile_label_box_dt
      style:
        display: flex
        flex-direction: column
        align-items: center
        width: 100%
        height: 42px
        overflow: hidden
      children:
      - id: noxh_tile_label_dt
        node_id: 1732:5221
        tag: p
        figma_type: TEXT
        figma_name: noxh_tile_label_dt
        style:
          width: 100%
          height: auto
          color: '#595959'
          white-space: pre-wrap
          font-family: '''Reddit Sans'''
          font-size: 14px
          font-weight: 400
          line-height: 21px
          text-align: center
        text: "Nhà ở \nxã hội"
        token_hint:
          color: Text/text-secondary
          letter-spacing: Display/Price/display-price-letter-spacing
          font-weight: Body/Page/body-page-font-weight
          font-family: Font family/font-family
          font-size: Tagline/Caption/tagline-caption-font-size
        data:
          source: server subcate-suggestion config (label)
          note: server-driven; client does not hardcode. Figma sample has a hard line break — do NOT copy it (content rule
            D-TILE-LABEL)
      token_hint:
        border-width: Stroke/stroke-divider
    token_hint:
      gap: Spacing/gap-x-small
      padding-left: Padding/padding-2x-small
      padding-right: Padding/padding-2x-small
      row-gap: Spacing/gap-x-small
      border-width: Stroke/stroke-divider
    behavior:
    - BH-NOXH-TILE-TAP
    - BH-NOXH-TILE-VISIBILITY
    change: NEW — 5th and last tile
  measured_width_px: 1148.0
  token_hint:
    gap: Spacing/gap-x-small
    row-gap: Spacing/gap-x-small
    border-width: Stroke/stroke-divider
```

### mobile — msite; iOS/Android reuse this frame (designer, 2026-09-24) — `sec_subcate_suggestion_mw` (Figma `1732:9236`)

```yaml
id: sec_subcate_suggestion_mw
spec_id: sec_subcate_suggestion
kind: section
figma_node: 1732:9236
platform: mobile — msite; iOS/Android reuse this frame (designer, 2026-09-24)
requirement:
  traces_to:
  - FR-2.1
  - FR-2.2
  - FR-2.3
  context: Ad Listing PTY (category Bất động sản) with NO sub-category selected. Existing row of sub-category suggestion tiles;
    add a 5th, last tile "Nhà ở xã hội".
  acceptance:
  - AC-FR2-1 no subcate + ≥1 NOXH ad in context → tile shown
  - AC-FR2-2 exactly 1 NOXH ad → tile still shown
  - AC-FR2-3 0 NOXH ads → tile absent, no gap
  - AC-FR2-4 tap → Ad Listing 1010 + Loại hình căn hộ = Nhà ở xã hội, location & other filters kept
  - AC-FR2-5 change location to 0 NOXH → hidden; back to ≥1 → shown
  - AC-FR2-6 sub-category selected → tile not shown
  - AC-EC-01/02/03 (see README)
uses:
  behaviors:
  - BH-EXISTING
  - BH-NOXH-TILE-TAP
  - BH-NOXH-TILE-VISIBILITY
variants:
  with_noxh:
    when: no subcate selected AND NOXH count ≥ 1
    tree: = layout (5 tiles)
  hidden:
    when: NOXH count = 0 OR count error OR subcate selected
    tree: layout minus node noxh_tile_mw
design_warnings:
- noxh_tile_mw has no icon_frame wrapper (siblings have 36px Link > Text > Image); build it with the same wrapper as siblings
- 'noxh icon: export from Figma node noxh_tile_icon_mw (1748:5145) — final per designer (OI-ICON)'
layout:
  id: sec_subcate_suggestion_mw
  node_id: 1732:9236
  tag: div
  figma_type: FRAME
  figma_name: sec_subcate_suggestion_mw
  style:
    display: flex
    flex-direction: row
    gap: 8px
    align-items: flex-start
    width: 100%
    flex-shrink: 0
    height: 74px
    overflow: hidden
    position: relative
  children:
  - id: subcate_tile_mw
    node_id: 1732:9237
    tag: div
    figma_type: FRAME
    figma_name: subcate_tile_mw
    style:
      display: flex
      flex-direction: column
      gap: 2px
      align-items: center
      padding: 0px 4px 0px 4px
      width: 64px
      flex-shrink: 0
      align-self: stretch
      min-width: 64px
    children:
    - id: subcate_tile_icon_frame_mw
      node_id: 1732:9238
      tag: div
      figma_type: FRAME
      figma_name: subcate_tile_icon_frame_mw
      style:
        display: flex
        flex-direction: row
        justify-content: center
        align-items: center
        width: 36px
        flex-shrink: 0
        height: 36px
      graphic: true
      token_hint:
        border-width: Stroke/stroke-divider
      data:
        source: server subcate-suggestion config (icon url)
        note: 'category illustration served with the config, not a DS icon. NOXH icon: export from this Figma node (final,
          OI-ICON)'
      asset_mode: none — see icon/data
    - id: subcate_tile_label_box_mw
      node_id: 1732:9256
      tag: div
      figma_type: FRAME
      figma_name: subcate_tile_label_box_mw
      style:
        display: flex
        flex-direction: column
        align-items: center
        width: 100%
        height: 36px
        overflow: hidden
      children:
      - id: subcate_tile_label_mw
        node_id: 1732:9257
        tag: p
        figma_type: TEXT
        figma_name: subcate_tile_label_mw
        style:
          width: 100%
          height: auto
          color: '#595959'
          font-family: '''Reddit Sans'''
          font-size: 12px
          font-weight: 500
          line-height: 18px
          text-align: center
        text: Căn hộ/Chung cư
        token_hint:
          color: Text/text-secondary
          line-height: Tagline/Annotation/tagline-annotation-line-height
          letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
          font-weight: Tagline/Footext/tagline-footext-font-weight
          font-family: Font family/font-family
          font-size: Tagline/Annotation/tagline-annotation-font-size
        data:
          source: server subcate-suggestion config (label)
          note: server-driven; client does not hardcode. Figma sample has a hard line break — do NOT copy it (content rule
            D-TILE-LABEL)
      token_hint:
        border-width: Stroke/stroke-divider
    token_hint:
      gap: Spacing/gap-min
      padding-left: Padding/padding-2x-small
      padding-right: Padding/padding-2x-small
      row-gap: Spacing/gap-min
      border-width: Stroke/stroke-divider
    behavior: BH-EXISTING
    change: existing tile, unchanged
  - id: subcate_tile_2_mw
    node_id: 1732:9258
    tag: div
    figma_type: FRAME
    figma_name: subcate_tile_2_mw
    style:
      display: flex
      flex-direction: column
      gap: 2px
      align-items: center
      padding: 0px 4px 0px 4px
      width: 64px
      flex-shrink: 0
      align-self: stretch
      min-width: 64px
    children:
    - id: subcate_tile_2_icon_frame_mw
      node_id: 1732:9259
      tag: div
      figma_type: FRAME
      figma_name: subcate_tile_2_icon_frame_mw
      style:
        display: flex
        flex-direction: row
        justify-content: center
        align-items: center
        width: 36px
        flex-shrink: 0
        height: 36px
      graphic: true
      token_hint:
        border-width: Stroke/stroke-divider
      data:
        source: server subcate-suggestion config (icon url)
        note: 'category illustration served with the config, not a DS icon. NOXH icon: export from this Figma node (final,
          OI-ICON)'
      asset_mode: none — see icon/data
    - id: subcate_tile_2_label_box_mw
      node_id: 1732:9279
      tag: div
      figma_type: FRAME
      figma_name: subcate_tile_2_label_box_mw
      style:
        display: flex
        flex-direction: column
        align-items: center
        width: 100%
        height: 18px
        overflow: hidden
      children:
      - id: subcate_tile_2_label_mw
        node_id: 1732:9280
        tag: p
        figma_type: TEXT
        figma_name: subcate_tile_2_label_mw
        style:
          width: fit-content
          flex-shrink: 0
          height: auto
          color: '#595959'
          white-space: nowrap
          font-family: '''Reddit Sans'''
          font-size: 12px
          font-weight: 500
          line-height: 18px
          text-align: center
        text: Nhà ở
        token_hint:
          color: Text/text-secondary
          line-height: Tagline/Annotation/tagline-annotation-line-height
          letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
          font-weight: Tagline/Footext/tagline-footext-font-weight
          font-family: Font family/font-family
          font-size: Tagline/Annotation/tagline-annotation-font-size
        data:
          source: server subcate-suggestion config (label)
          note: server-driven; client does not hardcode. Figma sample has a hard line break — do NOT copy it (content rule
            D-TILE-LABEL)
      token_hint:
        border-width: Stroke/stroke-divider
    token_hint:
      gap: Spacing/gap-min
      padding-left: Padding/padding-2x-small
      padding-right: Padding/padding-2x-small
      row-gap: Spacing/gap-min
      border-width: Stroke/stroke-divider
    behavior: BH-EXISTING
    change: existing tile, unchanged
  - id: subcate_tile_3_mw
    node_id: 1732:9281
    tag: div
    figma_type: FRAME
    figma_name: subcate_tile_3_mw
    style:
      display: flex
      flex-direction: column
      gap: 2px
      align-items: center
      padding: 0px 4px 0px 4px
      width: 64px
      flex-shrink: 0
      align-self: stretch
      min-width: 64px
    children:
    - id: subcate_tile_3_icon_frame_mw
      node_id: 1732:9282
      tag: div
      figma_type: FRAME
      figma_name: subcate_tile_3_icon_frame_mw
      style:
        display: flex
        flex-direction: row
        justify-content: center
        align-items: center
        width: 36px
        flex-shrink: 0
        height: 36px
      graphic: true
      token_hint:
        border-width: Stroke/stroke-divider
      data:
        source: server subcate-suggestion config (icon url)
        note: 'category illustration served with the config, not a DS icon. NOXH icon: export from this Figma node (final,
          OI-ICON)'
      asset_mode: none — see icon/data
    - id: subcate_tile_3_label_box_mw
      node_id: 1732:9298
      tag: div
      figma_type: FRAME
      figma_name: subcate_tile_3_label_box_mw
      style:
        display: flex
        flex-direction: column
        align-items: center
        width: 100%
        height: 18px
        overflow: hidden
      children:
      - id: subcate_tile_3_label_mw
        node_id: 1732:9299
        tag: p
        figma_type: TEXT
        figma_name: subcate_tile_3_label_mw
        style:
          width: fit-content
          flex-shrink: 0
          height: auto
          color: '#595959'
          white-space: nowrap
          font-family: '''Reddit Sans'''
          font-size: 12px
          font-weight: 500
          line-height: 18px
          text-align: center
        text: Đất
        token_hint:
          color: Text/text-secondary
          line-height: Tagline/Annotation/tagline-annotation-line-height
          letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
          font-weight: Tagline/Footext/tagline-footext-font-weight
          font-family: Font family/font-family
          font-size: Tagline/Annotation/tagline-annotation-font-size
        data:
          source: server subcate-suggestion config (label)
          note: server-driven; client does not hardcode. Figma sample has a hard line break — do NOT copy it (content rule
            D-TILE-LABEL)
      token_hint:
        border-width: Stroke/stroke-divider
    token_hint:
      gap: Spacing/gap-min
      padding-left: Padding/padding-2x-small
      padding-right: Padding/padding-2x-small
      row-gap: Spacing/gap-min
      border-width: Stroke/stroke-divider
    behavior: BH-EXISTING
    change: existing tile, unchanged
  - id: subcate_tile_4_mw
    node_id: 1732:9300
    tag: div
    figma_type: FRAME
    figma_name: subcate_tile_4_mw
    style:
      display: flex
      flex-direction: column
      gap: 2px
      align-items: center
      padding: 0px 4px 0px 4px
      width: 64px
      flex-shrink: 0
      align-self: stretch
      min-width: 64px
    children:
    - id: subcate_tile_4_icon_frame_mw
      node_id: 1732:9301
      tag: div
      figma_type: FRAME
      figma_name: subcate_tile_4_icon_frame_mw
      style:
        display: flex
        flex-direction: row
        justify-content: center
        align-items: center
        width: 36px
        flex-shrink: 0
        height: 36px
      graphic: true
      token_hint:
        border-width: Stroke/stroke-divider
      data:
        source: server subcate-suggestion config (icon url)
        note: 'category illustration served with the config, not a DS icon. NOXH icon: export from this Figma node (final,
          OI-ICON)'
      asset_mode: none — see icon/data
    - id: subcate_tile_4_label_box_mw
      node_id: 1732:9324
      tag: div
      figma_type: FRAME
      figma_name: subcate_tile_4_label_box_mw
      style:
        display: flex
        flex-direction: column
        align-items: center
        width: 100%
        height: 36px
        overflow: hidden
      children:
      - id: subcate_tile_4_label_mw
        node_id: 1732:9325
        tag: p
        figma_type: TEXT
        figma_name: subcate_tile_4_label_mw
        style:
          width: 100%
          height: auto
          color: '#595959'
          font-family: '''Reddit Sans'''
          font-size: 12px
          font-weight: 500
          line-height: 18px
          text-align: center
        text: Văn phòng, Mặt bằng kinh doanh
        token_hint:
          color: Text/text-secondary
          line-height: Tagline/Annotation/tagline-annotation-line-height
          letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
          font-weight: Tagline/Footext/tagline-footext-font-weight
          font-family: Font family/font-family
          font-size: Tagline/Annotation/tagline-annotation-font-size
        data:
          source: server subcate-suggestion config (label)
          note: server-driven; client does not hardcode. Figma sample has a hard line break — do NOT copy it (content rule
            D-TILE-LABEL)
      token_hint:
        border-width: Stroke/stroke-divider
    token_hint:
      gap: Spacing/gap-min
      padding-left: Padding/padding-2x-small
      padding-right: Padding/padding-2x-small
      row-gap: Spacing/gap-min
      border-width: Stroke/stroke-divider
    behavior: BH-EXISTING
    change: existing tile, unchanged
  - id: noxh_tile_mw
    node_id: 1732:9328
    tag: div
    figma_type: FRAME
    figma_name: noxh_tile_mw
    style:
      display: flex
      flex-direction: column
      gap: 2px
      align-items: center
      padding: 0px 4px 0px 4px
      width: 64px
      flex-shrink: 0
      align-self: stretch
      min-width: 64px
    children:
    - id: noxh_tile_icon_mw
      node_id: 1748:5145
      tag: div
      figma_type: FRAME
      figma_name: noxh_tile_icon_mw
      style:
        width: 36px
        flex-shrink: 0
        height: 36px
        overflow: hidden
        position: relative
      graphic: true
      data:
        source: server subcate-suggestion config (icon url)
        note: 'category illustration served with the config, not a DS icon. NOXH icon: export from this Figma node (final,
          OI-ICON)'
      asset_mode: none — see icon/data
    - id: noxh_tile_label_box_mw
      node_id: 1732:9352
      tag: div
      figma_type: FRAME
      figma_name: noxh_tile_label_box_mw
      style:
        display: flex
        flex-direction: column
        align-items: center
        width: 100%
        height: 36px
        overflow: hidden
      children:
      - id: noxh_tile_label_mw
        node_id: 1732:9353
        tag: p
        figma_type: TEXT
        figma_name: noxh_tile_label_mw
        style:
          width: 100%
          height: auto
          color: '#595959'
          white-space: pre-wrap
          font-family: '''Reddit Sans'''
          font-size: 12px
          font-weight: 500
          line-height: 18px
          text-align: center
        text: "Nhà ở \nxã hội"
        token_hint:
          color: Text/text-secondary
          line-height: Tagline/Annotation/tagline-annotation-line-height
          letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
          font-weight: Tagline/Footext/tagline-footext-font-weight
          font-family: Font family/font-family
          font-size: Tagline/Annotation/tagline-annotation-font-size
        data:
          source: server subcate-suggestion config (label)
          note: server-driven; client does not hardcode. Figma sample has a hard line break — do NOT copy it (content rule
            D-TILE-LABEL)
      token_hint:
        border-width: Stroke/stroke-divider
    token_hint:
      gap: Spacing/gap-min
      padding-left: Padding/padding-2x-small
      padding-right: Padding/padding-2x-small
      row-gap: Spacing/gap-min
      border-width: Stroke/stroke-divider
    behavior:
    - BH-NOXH-TILE-TAP
    - BH-NOXH-TILE-VISIBILITY
    change: NEW — 5th and last tile
  measured_width_px: 351.0
  token_hint:
    gap: Spacing/gap-x-small
    row-gap: Spacing/gap-x-small
    border-width: Stroke/stroke-divider
```

## 2.2 Screen `PTY.adlisting_apartment` › section `sec_filter_bar` (default)

### web_desktop — `sec_filter_bar_dt` (Figma `1732:5254`)

```yaml
id: sec_filter_bar_dt
spec_id: sec_filter_bar
kind: section
figma_node: 1732:5254
platform: web_desktop
requirement:
  traces_to:
  - FR-1
  - DR-1
  context: Ad Listing subcate Căn hộ chung cư (1010). Filter bar holds the "Loại hình căn hộ" param chip that opens the value
    list.
uses:
  behaviors:
  - BH-EXISTING
  - BH-FILTER-OPEN
variants:
  default:
    when: no apartment_type applied
    tree: = layout
  noxh_applied_desktop:
    when: apartment_type = Nhà ở xã hội (web)
    file: screens/02_adlisting_apartment/states/noxh_applied.desktop.yaml
  noxh_applied_app:
    when: apartment_type = Nhà ở xã hội (app/mobile)
    file: screens/02_adlisting_apartment/states/noxh_applied.app.yaml
layout:
  id: sec_filter_bar_dt
  node_id: 1732:5254
  tag: div
  figma_type: FRAME
  figma_name: sec_filter_bar_dt
  style:
    display: flex
    flex-direction: row
    justify-content: center
    align-items: flex-start
    width: 100%
    flex-shrink: 0
    height: 52px
    max-width: 1200px
    position: relative
  children:
  - id: container
    node_id: 1732:5255
    tag: div
    figma_type: FRAME
    figma_name: Container
    style:
      display: flex
      flex-direction: row
      align-items: center
      width: 1124px
      flex-shrink: 0
      align-self: stretch
      overflow: hidden
    children:
    - id: container
      node_id: 1732:5256
      tag: div
      figma_type: FRAME
      figma_name: Container
      style:
        display: flex
        flex-direction: row
        justify-content: center
        align-items: center
        width: fit-content
        flex-shrink: 0
        height: auto
        max-height: 32px
        background: '#ffffff'
      children:
      - id: link
        node_id: 1732:5257
        tag: div
        figma_type: FRAME
        figma_name: Link
        style:
          display: flex
          flex-direction: row
          gap: 2px
          justify-content: center
          align-items: center
          padding: 4px 12px 4px 12px
          width: 67.89px
          flex-shrink: 0
          height: 32px
          background: '#222222'
          border-radius: 9999px
        children:
        - id: icon
          node_id: 1732:5258
          tag: div
          figma_type: FRAME
          figma_name: Icon
          style:
            width: 20px
            flex-shrink: 0
            height: 20px
            overflow: hidden
            position: relative
          token_hint:
            border-width: Stroke/stroke-divider
          glyph:
            kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
            vector_nodes:
            - 1732:5259
        - id: loc
          node_id: 1732:5260
          tag: p
          figma_type: TEXT
          figma_name: Lọc
          style:
            width: fit-content
            flex-shrink: 0
            height: auto
            color: '#ffffff'
            white-space: nowrap
            font-family: '''Reddit Sans'''
            font-size: 14px
            font-weight: 500
            line-height: 20px
            text-align: center
          text: Lọc
          token_hint:
            color: Text/text-blank
            line-height: Tagline/Caption/tagline-caption-line-height
            letter-spacing: Body/Caption/body-caption-letter-spacing
            font-weight: Tagline/Footext/tagline-footext-font-weight
            font-family: Font family/font-family
            font-size: Tagline/Caption/tagline-caption-font-size
        token_hint:
          gap: Spacing/gap-min
          padding-left: Padding/padding-small
          padding-top: Padding/padding-2x-small
          padding-right: Padding/padding-small
          padding-bottom: Padding/padding-2x-small
          row-gap: Spacing/gap-min
          border-width: Stroke/stroke-divider
          background: Icon/icon-on-background
        behavior: BH-EXISTING
      token_hint:
        border-width: Stroke/stroke-divider
        background: Background/Light/background-primary
    - id: container_2
      node_id: 1732:5261
      tag: div
      figma_type: FRAME
      figma_name: Container
      style:
        display: flex
        flex-direction: column
        align-items: flex-start
        width: 950px
        flex-shrink: 0
        height: 52px
        position: relative
      children:
      - id: container
        node_id: 1732:5262
        tag: div
        figma_type: FRAME
        figma_name: Container
        style:
          display: flex
          flex-direction: row
          align-items: flex-start
          width: 100%
          height: 52px
          overflow: hidden
          position: relative
        children:
        - id: container_scroll_content
          node_id: 1732:5263
          tag: div
          figma_type: FRAME
          figma_name: Container:scroll-content
          style:
            display: flex
            flex-direction: row
            gap: 8px
            align-items: flex-start
            padding: 10px 0px 10px 0px
            width: 100%
            height: 52px
            position: absolute
            left: -508px
            top: 0px
          children:
          - id: text
            node_id: 1732:5264
            tag: div
            figma_type: FRAME
            figma_name: Text
            style:
              width: 0px
              flex-shrink: 0
              align-self: stretch
            token_hint:
              border-width: Stroke/stroke-divider
          - id: link
            node_id: 1732:5265
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 100.11px
              flex-shrink: 0
              height: 32px
              background: '#222222'
              border-radius: 9999px
            children:
            - id: mua_ban
              node_id: 1732:5266
              tag: p
              figma_type: TEXT
              figma_name: Mua bán
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#ffffff'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: Mua bán
              token_hint:
                color: Text/text-blank
                line-height: Tagline/Caption/tagline-caption-line-height
                letter-spacing: Body/Caption/body-caption-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            - id: icon
              node_id: 1732:5267
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:5269
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Icon/icon-on-background
            behavior: BH-EXISTING
          - id: link_2
            node_id: 1732:5270
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 152.84px
              flex-shrink: 0
              align-self: stretch
              background: '#222222'
              border-radius: 9999px
            children:
            - id: can_ho_chung_cu
              node_id: 1732:5271
              tag: p
              figma_type: TEXT
              figma_name: Căn hộ/Chung cư
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#ffffff'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: Căn hộ/Chung cư
              token_hint:
                color: Text/text-blank
                line-height: Tagline/Caption/tagline-caption-line-height
                letter-spacing: Body/Caption/body-caption-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            - id: button
              node_id: 1732:5273
              tag: div
              figma_type: FRAME
              figma_name: Button
              style:
                display: flex
                flex-direction: row
                justify-content: center
                align-items: center
                padding: 12px
                width: 38px
                flex-shrink: 0
                height: 38px
                position: absolute
                left: 114.92px
                top: -3px
                border-radius: 2px
              children:
              - id: icon
                node_id: 1732:5274
                tag: div
                figma_type: FRAME
                figma_name: Icon
                style:
                  width: 100%
                  height: 14px
                  overflow: hidden
                  position: relative
                  background: '#8c8c8c'
                  border-radius: 14px
                measured_width_px: 14.0
                token_hint:
                  border-width: Stroke/stroke-divider
                  background: Icon/icon-tertiary
                glyph:
                  kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                  vector_nodes:
                  - 1732:5275
              detached_from:
                component: Button
                note: 'Figma đặt tên `Button` nhưng node là FRAME, KHÔNG còn là INSTANCE của `Button` — đã detach. DỰNG THEO
                  CÂY dưới đây, đừng thay bằng component `Button` của DS: hai bên có thể đã khác nhau.'
              token_hint:
                padding-left: Padding/padding-small
                padding-top: Padding/padding-small
                padding-right: Padding/padding-small
                padding-bottom: Padding/padding-small
                border-width: Stroke/stroke-divider
              behavior: BH-EXISTING
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Icon/icon-on-background
            behavior: BH-EXISTING
          - id: link_3
            node_id: 1732:5276
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 113.48px
              flex-shrink: 0
              align-self: stretch
              background: '#222222'
              border-radius: 9999px
            children:
            - id: chu_dau_tu
              node_id: 1732:5277
              tag: p
              figma_type: TEXT
              figma_name: Chủ đầu tư
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#ffffff'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: Chủ đầu tư
              token_hint:
                color: Text/text-blank
                line-height: Tagline/Caption/tagline-caption-line-height
                letter-spacing: Body/Caption/body-caption-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            - id: button
              node_id: 1732:5279
              tag: div
              figma_type: FRAME
              figma_name: Button
              style:
                display: flex
                flex-direction: row
                justify-content: center
                align-items: center
                padding: 12px
                width: 38px
                flex-shrink: 0
                height: 38px
                position: absolute
                left: 75.74px
                top: -3px
                border-radius: 2px
              children:
              - id: icon
                node_id: 1732:5280
                tag: div
                figma_type: FRAME
                figma_name: Icon
                style:
                  width: 100%
                  height: 14px
                  overflow: hidden
                  position: relative
                  background: '#8c8c8c'
                  border-radius: 14px
                measured_width_px: 14.0
                token_hint:
                  border-width: Stroke/stroke-divider
                  background: Icon/icon-tertiary
                glyph:
                  kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                  vector_nodes:
                  - 1732:5281
              detached_from:
                component: Button
                note: 'Figma đặt tên `Button` nhưng node là FRAME, KHÔNG còn là INSTANCE của `Button` — đã detach. DỰNG THEO
                  CÂY dưới đây, đừng thay bằng component `Button` của DS: hai bên có thể đã khác nhau.'
              token_hint:
                padding-left: Padding/padding-small
                padding-top: Padding/padding-small
                padding-right: Padding/padding-small
                padding-bottom: Padding/padding-small
                border-width: Stroke/stroke-divider
              behavior: BH-EXISTING
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Icon/icon-on-background
            behavior: BH-EXISTING
          - id: link_4
            node_id: 1732:5282
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 94.3px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1732:5283
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: gia_ban
                node_id: 1732:5284
                tag: p
                figma_type: TEXT
                figma_name: Giá bán
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Giá bán
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Caption/tagline-caption-line-height
                  letter-spacing: Body/Caption/body-caption-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1732:5285
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:5287
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_5
            node_id: 1732:5288
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 82.62px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1732:5289
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: du_an
                node_id: 1732:5290
                tag: p
                figma_type: TEXT
                figma_name: Dự án
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Dự án
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Caption/tagline-caption-line-height
                  letter-spacing: Body/Caption/body-caption-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1732:5291
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:5293
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_6
            node_id: 1732:5294
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 133.67px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1732:5295
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: so_phong_ngu
                node_id: 1732:5296
                tag: p
                figma_type: TEXT
                figma_name: Số phòng ngủ
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Số phòng ngủ
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Caption/tagline-caption-line-height
                  letter-spacing: Body/Caption/body-caption-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1732:5297
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:5299
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_7
            node_id: 1732:5300
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 153.3px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1732:5301
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: so_phong_ve_sinh
                node_id: 1732:5302
                tag: p
                figma_type: TEXT
                figma_name: Số phòng vệ sinh
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Số phòng vệ sinh
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Caption/tagline-caption-line-height
                  letter-spacing: Body/Caption/body-caption-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1732:5303
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:5305
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_8
            node_id: 1732:5306
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 101.11px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1732:5307
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: dien_tich
                node_id: 1732:5308
                tag: p
                figma_type: TEXT
                figma_name: Diện tích
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Diện tích
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Caption/tagline-caption-line-height
                  letter-spacing: Body/Caption/body-caption-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1732:5309
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:5311
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_9
            node_id: 1732:5312
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 141.44px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1732:5313
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: don_gia_tr_m2
                node_id: 1732:5314
                tag: p
                figma_type: TEXT
                figma_name: Đơn giá (tr/m2)
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Đơn giá (tr/m2)
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Caption/tagline-caption-line-height
                  letter-spacing: Body/Caption/body-caption-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1732:5315
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:5317
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_10
            node_id: 1732:5318
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 146.83px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1732:5319
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: loai_hinh_can_ho
                node_id: 1732:5320
                tag: p
                figma_type: TEXT
                figma_name: Loại hình căn hộ
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Loại hình căn hộ
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Caption/tagline-caption-line-height
                  letter-spacing: Body/Caption/body-caption-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1732:5321
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:5323
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-FILTER-OPEN
          - id: link_11
            node_id: 1732:5324
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 150.05px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1732:5325
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: huong_ban_cong
                node_id: 1732:5326
                tag: p
                figma_type: TEXT
                figma_name: Hướng ban công
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Hướng ban công
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Caption/tagline-caption-line-height
                  letter-spacing: Body/Caption/body-caption-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1732:5327
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:5329
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          measured_width_px: 950.0
          token_hint:
            gap: Spacing/gap-x-small
            row-gap: Spacing/gap-x-small
            border-width: Stroke/stroke-divider
        measured_width_px: 950.0
        token_hint:
          border-width: Stroke/stroke-divider
      - id: button_next
        node_id: 1732:5330
        tag: div
        figma_type: FRAME
        figma_name: Button - Next
        style:
          display: flex
          flex-direction: row
          align-items: center
          width: 28px
          flex-shrink: 0
          height: 28px
          position: absolute
          left: 962px
          top: 12px
        children:
        - id: icon
          node_id: 1732:5331
          tag: div
          figma_type: FRAME
          figma_name: Icon
          style:
            width: 14px
            flex-shrink: 0
            height: 14px
            overflow: hidden
            position: relative
          token_hint:
            border-width: Stroke/stroke-divider
          glyph:
            kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
            vector_nodes:
            - 1732:5333
        token_hint:
          border-width: Stroke/stroke-divider
        behavior: BH-EXISTING
      token_hint:
        border-width: Stroke/stroke-divider
    token_hint:
      border-width: Stroke/stroke-divider
  - id: button
    node_id: 1732:5334
    tag: div
    figma_type: FRAME
    figma_name: Button
    style:
      display: flex
      flex-direction: row
      align-items: center
      width: 52px
      flex-shrink: 0
      height: 32px
      box-shadow: inset 0 0 0 1px rgba(0, 0, 0, 0.0)
      border-radius: 8px
    children:
    - id: xoa_loc
      node_id: 1732:5335
      tag: p
      figma_type: TEXT
      figma_name: Xoá lọc
      style:
        width: fit-content
        flex-shrink: 0
        height: auto
        color: '#222222'
        white-space: nowrap
        font-family: '''Reddit Sans'''
        font-size: 14px
        font-weight: 700
        line-height: 20px
        text-align: center
      text: Xoá lọc
      token_hint:
        color: Text/text-primary
        line-height: Tagline/Caption/tagline-caption-line-height
        letter-spacing: Body/Caption/body-caption-letter-spacing
        font-weight: Display/Section/display-section-font-weight
        font-family: Font family/font-family
        font-size: Tagline/Caption/tagline-caption-font-size
    detached_from:
      component: Button
      note: 'Figma đặt tên `Button` nhưng node là FRAME, KHÔNG còn là INSTANCE của `Button` — đã detach. DỰNG THEO CÂY dưới
        đây, đừng thay bằng component `Button` của DS: hai bên có thể đã khác nhau.'
    token_hint:
      border-width: Stroke/stroke-divider
      border-radius: Radius/radius-card-small
    behavior: BH-EXISTING
  measured_width_px: 1176.0
  token_hint:
    border-width: Stroke/stroke-divider
```

### mobile — msite; iOS/Android reuse — `sec_filter_bar_mw` (Figma `1732:9434`)

```yaml
id: sec_filter_bar_mw
spec_id: sec_filter_bar
kind: section
figma_node: 1732:9434
platform: mobile — msite; iOS/Android reuse
requirement:
  traces_to:
  - FR-1
  - DR-1
  context: Ad Listing subcate Căn hộ chung cư (1010). Filter bar holds the "Loại hình căn hộ" param chip that opens the value
    list.
uses:
  behaviors:
  - BH-EXISTING
  - BH-FILTER-OPEN
variants:
  default:
    when: no apartment_type applied
    tree: = layout
  noxh_applied_desktop:
    when: apartment_type = Nhà ở xã hội (web)
    file: screens/02_adlisting_apartment/states/noxh_applied.desktop.yaml
  noxh_applied_app:
    when: apartment_type = Nhà ở xã hội (app/mobile)
    file: screens/02_adlisting_apartment/states/noxh_applied.app.yaml
layout:
  id: sec_filter_bar_mw
  node_id: 1732:9434
  tag: div
  figma_type: FRAME
  figma_name: sec_filter_bar_mw
  style:
    display: flex
    flex-direction: column
    align-items: flex-start
    width: 100%
    height: 84px
    min-height: 72px
    background: '#ffffff'
    position: relative
  children:
  - id: container
    node_id: 1732:9435
    tag: div
    figma_type: FRAME
    figma_name: Container
    style:
      display: flex
      flex-direction: column
      gap: 8px
      justify-content: center
      align-items: flex-start
      padding: 0px 16px 8px 16px
      width: 375px
      flex-shrink: 0
      height: auto
    children:
    - id: container
      node_id: 1732:9436
      tag: div
      figma_type: FRAME
      figma_name: Container
      style:
        display: flex
        flex-direction: row
        gap: 16px
        justify-content: space-between
        align-items: center
        width: 100%
        height: 36px
      children:
      - id: text
        node_id: 1732:9437
        tag: div
        figma_type: FRAME
        figma_name: Text
        style:
          display: flex
          flex-direction: row
          justify-content: center
          align-items: center
          padding: 0px 0px 0px 4px
          width: 188.52px
          flex-shrink: 0
          height: 20px
        children:
        - id: khu_vuc
          node_id: 1732:9439
          tag: p
          figma_type: TEXT
          figma_name: 'Khu vực:'
          style:
            width: fit-content
            flex-shrink: 0
            height: auto
            color: '#222222'
            white-space: nowrap
            font-family: '''Reddit Sans'''
            font-size: 14px
            font-weight: 700
            line-height: 20px
          text: Khu vực:   
          token_hint:
            color: Text/text-primary
            line-height: Tagline/Section/tagline-section-line-height
            letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
            font-weight: Label/Annotation/label-annotation-font-weight
            font-family: Font family/font-family
            font-size: Tagline/Caption/tagline-caption-font-size
        - id: paragraph
          node_id: 1732:9440
          tag: div
          figma_type: FRAME
          figma_name: Paragraph
          style:
            display: flex
            flex-direction: column
            align-items: flex-start
            width: 97.52px
            flex-shrink: 0
            height: 20px
            max-width: 192px
            overflow: hidden
          children:
          - id: tp_ho_chi_minh
            node_id: 1732:9441
            tag: p
            figma_type: TEXT
            figma_name: Tp Hồ Chí Minh
            style:
              width: fit-content
              flex-shrink: 0
              height: auto
              color: '#fa6819'
              white-space: nowrap
              font-family: '''Reddit Sans'''
              font-size: 14px
              font-weight: 700
              line-height: 20px
            text: Tp Hồ Chí Minh
            token_hint:
              color: Text/text-brand
              line-height: Tagline/Section/tagline-section-line-height
              letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
              font-weight: Label/Annotation/label-annotation-font-weight
              font-family: Font family/font-family
              font-size: Tagline/Caption/tagline-caption-font-size
          token_hint:
            border-width: Stroke/stroke-divider
        - id: icon
          node_id: 1732:9442
          tag: div
          figma_type: FRAME
          figma_name: Icon
          style:
            width: 20px
            flex-shrink: 0
            height: 20px
            overflow: hidden
            position: relative
          token_hint:
            border-width: Stroke/stroke-divider
          glyph:
            kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
            vector_nodes:
            - 1732:9444
        token_hint:
          padding-left: Padding/padding-2x-small
          border-width: Stroke/stroke-divider
      - id: button
        node_id: 1732:9445
        tag: div
        figma_type: FRAME
        figma_name: Button
        style:
          display: flex
          flex-direction: row
          align-items: center
          width: fit-content
          flex-shrink: 0
          height: 32px
          box-shadow: inset 0 0 0 1px rgba(0, 0, 0, 0.0)
          border-radius: 8px
        children:
        - id: xoa_loc
          node_id: 1732:9446
          tag: p
          figma_type: TEXT
          figma_name: Xoá lọc
          style:
            width: fit-content
            flex-shrink: 0
            height: auto
            color: '#222222'
            white-space: nowrap
            font-family: '''Reddit Sans'''
            font-size: 14px
            font-weight: 700
            line-height: 20px
            text-align: center
          text: Xoá lọc
          token_hint:
            color: Text/text-primary
            line-height: Tagline/Section/tagline-section-line-height
            letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
            font-weight: Label/Annotation/label-annotation-font-weight
            font-family: Font family/font-family
            font-size: Tagline/Caption/tagline-caption-font-size
        detached_from:
          component: Button
          note: 'Figma đặt tên `Button` nhưng node là FRAME, KHÔNG còn là INSTANCE của `Button` — đã detach. DỰNG THEO CÂY
            dưới đây, đừng thay bằng component `Button` của DS: hai bên có thể đã khác nhau.'
        token_hint:
          border-width: Stroke/stroke-divider
          border-radius: Radius/radius-card-small
        behavior: BH-EXISTING
      token_hint:
        row-gap: Spacing/gap-medium
        border-width: Stroke/stroke-divider
    - id: container_2
      node_id: 1732:9751
      tag: div
      figma_type: FRAME
      figma_name: Container
      style:
        display: flex
        flex-direction: row
        align-items: center
        width: 100%
        height: 32px
        overflow: hidden
        position: relative
      children:
      - id: container
        node_id: 1732:9758
        tag: div
        figma_type: FRAME
        figma_name: Container
        style:
          display: flex
          flex-direction: row
          align-items: flex-start
          width: 950px
          flex-shrink: 0
          height: 52px
          overflow: hidden
          position: relative
        children:
        - id: container_scroll_content
          node_id: 1732:9759
          tag: div
          figma_type: FRAME
          figma_name: Container:scroll-content
          style:
            display: flex
            flex-direction: row
            gap: 8px
            align-items: flex-start
            padding: 10px 0px 10px 0px
            width: 100%
            height: 52px
            position: absolute
            left: -508px
            top: 0px
          children:
          - id: text
            node_id: 1732:9760
            tag: div
            figma_type: FRAME
            figma_name: Text
            style:
              width: 0px
              flex-shrink: 0
              align-self: stretch
            token_hint:
              border-width: Stroke/stroke-divider
          - id: link
            node_id: 1732:9761
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 100.11px
              flex-shrink: 0
              height: 32px
              background: '#222222'
              border-radius: 9999px
            children:
            - id: mua_ban
              node_id: 1732:9762
              tag: p
              figma_type: TEXT
              figma_name: Mua bán
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#ffffff'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: Mua bán
              token_hint:
                color: Text/text-blank
                line-height: Tagline/Caption/tagline-caption-line-height
                letter-spacing: Body/Caption/body-caption-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            - id: icon
              node_id: 1732:9763
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:9765
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Icon/icon-on-background
            behavior: BH-EXISTING
          - id: link_2
            node_id: 1732:9766
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 152.84px
              flex-shrink: 0
              align-self: stretch
              background: '#222222'
              border-radius: 9999px
            children:
            - id: can_ho_chung_cu
              node_id: 1732:9767
              tag: p
              figma_type: TEXT
              figma_name: Căn hộ/Chung cư
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#ffffff'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: Căn hộ/Chung cư
              token_hint:
                color: Text/text-blank
                line-height: Tagline/Caption/tagline-caption-line-height
                letter-spacing: Body/Caption/body-caption-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            - id: button
              node_id: 1732:9769
              tag: div
              figma_type: FRAME
              figma_name: Button
              style:
                display: flex
                flex-direction: row
                justify-content: center
                align-items: center
                padding: 12px
                width: 38px
                flex-shrink: 0
                height: 38px
                position: absolute
                left: 114.92px
                top: -3px
                border-radius: 2px
              children:
              - id: icon
                node_id: 1732:9770
                tag: div
                figma_type: FRAME
                figma_name: Icon
                style:
                  width: 100%
                  height: 14px
                  overflow: hidden
                  position: relative
                  background: '#8c8c8c'
                  border-radius: 14px
                measured_width_px: 14.0
                token_hint:
                  border-width: Stroke/stroke-divider
                  background: Icon/icon-tertiary
                glyph:
                  kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                  vector_nodes:
                  - 1732:9771
              detached_from:
                component: Button
                note: 'Figma đặt tên `Button` nhưng node là FRAME, KHÔNG còn là INSTANCE của `Button` — đã detach. DỰNG THEO
                  CÂY dưới đây, đừng thay bằng component `Button` của DS: hai bên có thể đã khác nhau.'
              token_hint:
                padding-left: Padding/padding-small
                padding-top: Padding/padding-small
                padding-right: Padding/padding-small
                padding-bottom: Padding/padding-small
                border-width: Stroke/stroke-divider
              behavior: BH-EXISTING
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Icon/icon-on-background
            behavior: BH-EXISTING
          - id: link_3
            node_id: 1732:9772
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 113.48px
              flex-shrink: 0
              align-self: stretch
              background: '#222222'
              border-radius: 9999px
            children:
            - id: chu_dau_tu
              node_id: 1732:9773
              tag: p
              figma_type: TEXT
              figma_name: Chủ đầu tư
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#ffffff'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: Chủ đầu tư
              token_hint:
                color: Text/text-blank
                line-height: Tagline/Caption/tagline-caption-line-height
                letter-spacing: Body/Caption/body-caption-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            - id: button
              node_id: 1732:9775
              tag: div
              figma_type: FRAME
              figma_name: Button
              style:
                display: flex
                flex-direction: row
                justify-content: center
                align-items: center
                padding: 12px
                width: 38px
                flex-shrink: 0
                height: 38px
                position: absolute
                left: 75.74px
                top: -3px
                border-radius: 2px
              children:
              - id: icon
                node_id: 1732:9776
                tag: div
                figma_type: FRAME
                figma_name: Icon
                style:
                  width: 100%
                  height: 14px
                  overflow: hidden
                  position: relative
                  background: '#8c8c8c'
                  border-radius: 14px
                measured_width_px: 14.0
                token_hint:
                  border-width: Stroke/stroke-divider
                  background: Icon/icon-tertiary
                glyph:
                  kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                  vector_nodes:
                  - 1732:9777
              detached_from:
                component: Button
                note: 'Figma đặt tên `Button` nhưng node là FRAME, KHÔNG còn là INSTANCE của `Button` — đã detach. DỰNG THEO
                  CÂY dưới đây, đừng thay bằng component `Button` của DS: hai bên có thể đã khác nhau.'
              token_hint:
                padding-left: Padding/padding-small
                padding-top: Padding/padding-small
                padding-right: Padding/padding-small
                padding-bottom: Padding/padding-small
                border-width: Stroke/stroke-divider
              behavior: BH-EXISTING
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Icon/icon-on-background
            behavior: BH-EXISTING
          - id: link_4
            node_id: 1732:9778
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 94.3px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1732:9779
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: gia_ban
                node_id: 1732:9780
                tag: p
                figma_type: TEXT
                figma_name: Giá bán
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Giá bán
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Caption/tagline-caption-line-height
                  letter-spacing: Body/Caption/body-caption-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1732:9781
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:9783
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_5
            node_id: 1732:9784
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 82.62px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1732:9785
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: du_an
                node_id: 1732:9786
                tag: p
                figma_type: TEXT
                figma_name: Dự án
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Dự án
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Caption/tagline-caption-line-height
                  letter-spacing: Body/Caption/body-caption-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1732:9787
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:9789
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_6
            node_id: 1732:9790
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 133.67px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1732:9791
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: so_phong_ngu
                node_id: 1732:9792
                tag: p
                figma_type: TEXT
                figma_name: Số phòng ngủ
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Số phòng ngủ
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Caption/tagline-caption-line-height
                  letter-spacing: Body/Caption/body-caption-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1732:9793
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:9795
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_7
            node_id: 1732:9796
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 153.3px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1732:9797
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: so_phong_ve_sinh
                node_id: 1732:9798
                tag: p
                figma_type: TEXT
                figma_name: Số phòng vệ sinh
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Số phòng vệ sinh
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Caption/tagline-caption-line-height
                  letter-spacing: Body/Caption/body-caption-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1732:9799
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:9801
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_8
            node_id: 1732:9802
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 101.11px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1732:9803
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: dien_tich
                node_id: 1732:9804
                tag: p
                figma_type: TEXT
                figma_name: Diện tích
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Diện tích
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Caption/tagline-caption-line-height
                  letter-spacing: Body/Caption/body-caption-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1732:9805
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:9807
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_9
            node_id: 1732:9808
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 141.44px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1732:9809
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: don_gia_tr_m2
                node_id: 1732:9810
                tag: p
                figma_type: TEXT
                figma_name: Đơn giá (tr/m2)
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Đơn giá (tr/m2)
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Caption/tagline-caption-line-height
                  letter-spacing: Body/Caption/body-caption-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1732:9811
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:9813
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_10
            node_id: 1732:9814
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 146.83px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1732:9815
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: loai_hinh_can_ho
                node_id: 1732:9816
                tag: p
                figma_type: TEXT
                figma_name: Loại hình căn hộ
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Loại hình căn hộ
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Caption/tagline-caption-line-height
                  letter-spacing: Body/Caption/body-caption-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1732:9817
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:9819
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-FILTER-OPEN
          - id: link_11
            node_id: 1732:9820
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 150.05px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1732:9821
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: huong_ban_cong
                node_id: 1732:9822
                tag: p
                figma_type: TEXT
                figma_name: Hướng ban công
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Hướng ban công
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Caption/tagline-caption-line-height
                  letter-spacing: Body/Caption/body-caption-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1732:9823
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1732:9825
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          measured_width_px: 950.0
          token_hint:
            gap: Spacing/gap-x-small
            row-gap: Spacing/gap-x-small
            border-width: Stroke/stroke-divider
        token_hint:
          border-width: Stroke/stroke-divider
      - id: container_2
        node_id: 1732:9752
        tag: div
        figma_type: FRAME
        figma_name: Container
        style:
          display: flex
          flex-direction: row
          justify-content: center
          align-items: center
          padding: 0px 8px 0px 0px
          width: fit-content
          flex-shrink: 0
          height: auto
          max-height: 32px
          background: '#ffffff'
        children:
        - id: container
          node_id: 1748:15191
          tag: div
          figma_type: FRAME
          figma_name: Container
          style:
            display: flex
            flex-direction: row
            justify-content: center
            align-items: center
            width: fit-content
            flex-shrink: 0
            height: auto
            max-height: 32px
            background: '#ffffff'
          children:
          - id: link
            node_id: 1748:15192
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 67.89px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: icon
              node_id: 1748:15193
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1748:15194
            - id: loc
              node_id: 1748:15195
              tag: p
              figma_type: TEXT
              figma_name: Lọc
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#222222'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: Lọc
              token_hint:
                color: Text/text-primary
                line-height: Tagline/Section/tagline-section-line-height
                letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          token_hint:
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
        token_hint:
          border-width: Stroke/stroke-divider
          background: Background/Light/background-primary
      token_hint:
        border-width: Stroke/stroke-divider
    token_hint:
      gap: Spacing/gap-x-small
      padding-left: Padding/padding-medium
      padding-right: Padding/padding-medium
      padding-bottom: Padding/padding-x-small
      row-gap: Spacing/gap-x-small
      border-width: Stroke/stroke-divider
  token_hint:
    border-width: Stroke/stroke-divider
    background: Background/Light/background-primary
```

## 2.3 State `PTY.adlisting_apartment/noxh_applied`

### web_desktop — `sec_filter_bar_noxh_applied_dt` (Figma `1748:8561`)

```yaml
id: sec_filter_bar_noxh_applied_dt
spec_id: PTY.adlisting_apartment/noxh_applied
kind: screen_state
platform: web_desktop
figma_node: 1748:8561
requirement:
  traces_to:
  - FR-2.3
  - DR-1
  - PRD 5.1
  context: 'Destination after tapping the NOXH tile: subcate chip "Căn hộ/Chung cư ×" + applied chip "Nhà ở xã hội ×". Rest
    of the listing page is existing and unchanged (OUT of this spec).'
uses:
  behaviors:
  - BH-EXISTING
  - BH-NOXH-CHIP-REMOVE
related_behaviors:
- BH-BACK
- BH-ADTYPE-RESET
layout:
  id: sec_filter_bar_noxh_applied_dt
  node_id: 1748:8561
  tag: div
  figma_type: FRAME
  figma_name: sec_filter_bar_noxh_applied_dt
  style:
    display: flex
    flex-direction: column
    align-items: flex-start
    padding: 16px 20px 16px 20px
    width: 100%
    flex-shrink: 0
    height: auto
    background: '#ffffff'
    border-radius: 12px
    position: relative
  children:
  - id: container
    node_id: 1748:8562
    tag: div
    figma_type: FRAME
    figma_name: Container
    style:
      display: flex
      flex-direction: column
      gap: 8px
      align-items: flex-start
      width: 100%
      height: auto
    children:
    - id: numbered_list
      node_id: 1748:8564
      tag: div
      figma_type: FRAME
      figma_name: Numbered List
      style:
        width: 1125.19px
        flex-shrink: 0
        height: 21px
        overflow: hidden
        position: relative
      children:
      - id: nha_tot
        node_id: 1748:8565
        tag: p
        figma_type: TEXT
        figma_name: Nhà Tốt
        style:
          width: 43px
          position: absolute
          left: 0px
          top: 2px
          height: 18px
          color: '#8c8c8c'
          white-space: nowrap
          font-family: '''Reddit Sans'''
          font-size: 12px
          font-weight: 400
          line-height: 18px
        text: Nhà Tốt
        token_hint:
          color: Text/text-tertiary
          line-height: Tagline/Annotation/tagline-annotation-line-height
          letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
          font-weight: Body/Caption/body-caption-font-weight
          font-family: Font family/font-family
          font-size: Tagline/Annotation/tagline-annotation-font-size
      - id: breadcrumb_separator_dt
        node_id: 1748:8566
        tag: p
        figma_type: TEXT
        figma_name: breadcrumb_separator_dt
        style:
          width: 5px
          position: absolute
          left: 46.44px
          top: 2px
          height: 18px
          color: '#8c8c8c'
          white-space: nowrap
          font-family: '''Reddit Sans'''
          font-size: 12px
          font-weight: 400
          line-height: 18px
        text: /
        token_hint:
          color: Text/text-tertiary
          line-height: Tagline/Annotation/tagline-annotation-line-height
          letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
          font-weight: Body/Caption/body-caption-font-weight
          font-family: Font family/font-family
          font-size: Tagline/Annotation/tagline-annotation-font-size
      - id: mua_ban_can_ho_chung_cu
        node_id: 1748:8567
        tag: p
        figma_type: TEXT
        figma_name: Mua bán Căn hộ Chung cư
        style:
          width: 144px
          position: absolute
          left: 59.03px
          top: 1px
          height: 18px
          color: '#222222'
          white-space: nowrap
          font-family: '''Reddit Sans'''
          font-size: 12px
          font-weight: 700
          line-height: 18px
        text: Mua bán Căn hộ Chung cư
        token_hint:
          color: Text/text-primary
          line-height: Tagline/Annotation/tagline-annotation-line-height
          letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
          font-weight: Label/Annotation/label-annotation-font-weight
          font-family: Font family/font-family
          font-size: Tagline/Annotation/tagline-annotation-font-size
      token_hint:
        border-width: Stroke/stroke-divider
    - id: container
      node_id: 1748:8568
      tag: div
      figma_type: FRAME
      figma_name: Container
      style:
        display: flex
        flex-direction: column
        align-items: flex-start
        width: 100%
        height: auto
      children:
      - id: container
        node_id: 1748:8569
        tag: div
        figma_type: FRAME
        figma_name: Container
        style:
          display: flex
          flex-direction: row
          gap: 20px
          align-items: center
          width: 100%
          height: auto
        children:
        - id: mua_ban_13_036_can_ho_chung_cu_toan_quoc_thang
          node_id: 1748:8571
          tag: p
          figma_type: TEXT
          figma_name: Mua Bán 13.036 Căn Hộ Chung Cư Toàn Quốc Tháng 09/
          style:
            width: fit-content
            flex-shrink: 0
            height: auto
            color: '#222222'
            white-space: nowrap
            font-family: '''Reddit Sans'''
            font-size: 16px
            font-weight: 700
            line-height: 24px
          text: Mua Bán 13.036 Căn Hộ Chung Cư Toàn Quốc Tháng 09/2026
          token_hint:
            color: Text/text-primary
            line-height: Label/Page/label-page-line-height
            letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
            font-weight: Label/Annotation/label-annotation-font-weight
            font-family: Font family/font-family
            font-size: Tagline/Section/tagline-section-font-size
        - id: button
          node_id: 1748:8573
          tag: div
          figma_type: FRAME
          figma_name: Button
          style:
            display: flex
            flex-direction: row
            align-items: center
            padding: 0px 12px 0px 12px
            width: fit-content
            flex-shrink: 0
            height: 32px
            min-height: 32px
            background: '#ffffff'
            box-shadow: 'inset 0 0 0 1px #dadada'
            border-radius: 9999px
          children:
          - id: icon
            node_id: 1748:8575
            tag: div
            figma_type: FRAME
            figma_name: Icon
            style:
              width: 20px
              flex-shrink: 0
              height: 20px
              overflow: hidden
              position: absolute
              left: 9px
              top: 6px
            token_hint:
              border-width: Stroke/stroke-divider
            glyph:
              kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
              vector_nodes:
              - 1748:8576
          - id: luu_tim_kiem
            node_id: 1748:8577
            tag: p
            figma_type: TEXT
            figma_name: Lưu tìm kiếm
            style:
              width: fit-content
              flex-shrink: 0
              height: auto
              color: '#222222'
              white-space: nowrap
              font-family: '''Reddit Sans'''
              font-size: 14px
              font-weight: 700
              line-height: 20px
              text-align: center
            text: Lưu tìm kiếm
            token_hint:
              color: Text/text-primary
              line-height: Tagline/Section/tagline-section-line-height
              letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
              font-weight: Label/Annotation/label-annotation-font-weight
              font-family: Font family/font-family
              font-size: Tagline/Caption/tagline-caption-font-size
          detached_from:
            component: Button
            note: 'Figma đặt tên `Button` nhưng node là FRAME, KHÔNG còn là INSTANCE của `Button` — đã detach. DỰNG THEO CÂY
              dưới đây, đừng thay bằng component `Button` của DS: hai bên có thể đã khác nhau.'
          token_hint:
            padding-left: Padding/padding-small
            padding-right: Padding/padding-small
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
          behavior: BH-EXISTING
        token_hint:
          gap: Spacing/gap-large-20
          row-gap: Spacing/gap-large-20
          border-width: Stroke/stroke-divider
      - id: container_2
        node_id: 1748:8578
        tag: div
        figma_type: FRAME
        figma_name: Container
        style:
          width: 100%
          height: 18px
          position: relative
        children:
        - id: container
          node_id: 1748:8579
          tag: div
          figma_type: FRAME
          figma_name: Container
          style:
            display: flex
            flex-direction: column
            align-items: flex-start
            width: 1037.92px
            height: 18px
            position: absolute
            left: 0px
            top: 0px
            overflow: hidden
            background: '#ffffff'
          children:
          - id: mua_ban_can_ho_chung_cu_toan_quoc_hien_rat_da_
            node_id: 1748:8581
            tag: p
            figma_type: TEXT
            figma_name: Mua bán căn hộ chung cư toàn quốc hiện rất đa dạng
            style:
              width: 100%
              height: auto
              color: '#595959'
              font-family: '''Reddit Sans'''
              font-size: 12px
              font-weight: 700
              line-height: 18px
            spans:
            - text: Mua bán căn hộ chung cư toàn quốc
              font-weight: 700
              font-size: 12px
              color: '#595959'
            - text: ' hiện rất đa dạng, với hơn 13.036 '
              font-weight: 400
              font-size: 12px
              color: '#595959'
            - text: căn hộ chung cư toàn quốc
              font-weight: 700
              font-size: 12px
              color: '#595959'
            - text: ' đang được chào bán mới mỗi ngày, người mua có thể dễ dàng tìm thấy '
              font-weight: 400
              font-size: 12px
              color: '#595959'
            - text: dự án chung cư
              font-weight: 700
              font-size: 12px
              color: '#595959'
            - text: ' phù hợp với ngân sách và mục tiêu sử dụng. '
              font-weight: 400
              font-size: 12px
              color: '#595959'
            - text: Giá mua bán căn hộ chung cư toàn quốc
              font-weight: 700
              font-size: 12px
              color: '#595959'
            - text: ' dao động đang cập nhật, với mức trung bình khoảng đang cập nhật. Giá đang cập nhật và biến động đang
                cập nhật phản ánh xu hướng thị trường hiện tại, trong đó các đô thị lớn thường có mức '
              font-weight: 400
              font-size: 12px
              color: '#595959'
            - text: giá chung cư
              font-weight: 700
              font-size: 12px
              color: '#595959'
            - text: ' và biến động cao hơn do sức cầu lớn.'
              font-weight: 400
              font-size: 12px
              color: '#595959'
            text_plain: Mua bán căn hộ chung cư toàn quốc hiện rất đa dạng, với hơn 13.036 căn hộ chung cư toàn quốc đang
              được chào bán mới mỗi ngày, người mua có thể dễ dàng tìm thấy dự án chung cư phù hợp với ngân sách và mục tiêu
              sử dụng. Giá mua bán căn hộ chung cư toàn quốc dao động đang cập nhật, với mức trung bình khoảng đang cập nhật.
              Giá đang cập nhật và biến động đang cập nhật phản ánh xu hướng thị trường hiện tại, trong đó các đô thị lớn
              thường có mức giá chung cư và biến động cao hơn do sức cầu lớn.
            token_hint:
              color: Text/text-secondary
              line-height: Tagline/Annotation/tagline-annotation-line-height
              letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
              font-weight:
              - Body/Caption/body-caption-font-weight
              - Label/Annotation/label-annotation-font-weight
              font-family: Font family/font-family
              font-size: Tagline/Annotation/tagline-annotation-font-size
          - id: paragraph
            node_id: 1748:8582
            tag: div
            figma_type: FRAME
            figma_name: Paragraph
            style:
              display: flex
              flex-direction: column
              align-items: flex-start
              padding: 8px 0px 8px 0px
              width: 100%
              height: auto
            children:
            - id: can_ho_chung_cu_toan_quoc_la_phan_khuc_bat_don
              node_id: 1748:8583
              tag: p
              figma_type: TEXT
              figma_name: Căn hộ chung cư toàn quốc là phân khúc bất động sả
              style:
                width: 100%
                height: auto
                color: '#595959'
                font-family: '''Reddit Sans'''
                font-size: 12px
                font-weight: 700
                line-height: 18px
              spans:
              - text: Căn hộ chung cư toàn quốc
                font-weight: 700
                font-size: 12px
                color: '#595959'
              - text: ' là phân khúc bất động sản trọng điểm của thị trường nhà ở Việt Nam, phục vụ nhu cầu an cư, đầu tư
                  và khai thác cho thuê của đa dạng nhóm khách hàng từ thành thị tới ven đô. Trên quy mô toàn quốc, '
                font-weight: 400
                font-size: 12px
                color: '#595959'
              - text: căn hộ chung cư
                font-weight: 700
                font-size: 12px
                color: '#595959'
              - text: ' đã trở thành lựa chọn ưu tiên vì thiết kế tối ưu, tiện ích đầy đủ, an ninh đảm bảo và khả năng kết
                  nối hạ tầng nhanh chóng. Từ căn hộ studio cho người độc thân tới căn hộ 2–3 phòng ngủ cho gia đình, '
                font-weight: 400
                font-size: 12px
                color: '#595959'
              - text: thị trường chung cư
                font-weight: 700
                font-size: 12px
                color: '#595959'
              - text: ' cung cấp giải pháp nhà ở linh hoạt phù hợp với nhu cầu ở thực và đầu tư dài hạn.'
                font-weight: 400
                font-size: 12px
                color: '#595959'
              text_plain: Căn hộ chung cư toàn quốc là phân khúc bất động sản trọng điểm của thị trường nhà ở Việt Nam, phục
                vụ nhu cầu an cư, đầu tư và khai thác cho thuê của đa dạng nhóm khách hàng từ thành thị tới ven đô. Trên quy
                mô toàn quốc, căn hộ chung cư đã trở thành lựa chọn ưu tiên vì thiết kế tối ưu, tiện ích đầy đủ, an ninh đảm
                bảo và khả năng kết nối hạ tầng nhanh chóng. Từ căn hộ studio cho người độc thân tới căn hộ 2–3 phòng ngủ
                cho gia đình, thị trường chung cư cung cấp giải pháp nhà ở linh hoạt phù hợp với nhu cầu ở thực và đầu tư
                dài hạn.
              token_hint:
                color: Text/text-secondary
                line-height: Tagline/Annotation/tagline-annotation-line-height
                letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                font-weight:
                - Body/Caption/body-caption-font-weight
                - Label/Annotation/label-annotation-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Annotation/tagline-annotation-font-size
            token_hint:
              padding-top: Padding/padding-x-small
              padding-bottom: Padding/padding-x-small
              border-width: Stroke/stroke-divider
          token_hint:
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
        - id: button
          node_id: 1748:8584
          tag: div
          figma_type: FRAME
          figma_name: Button
          style:
            display: flex
            flex-direction: column
            justify-content: center
            align-items: center
            width: 57px
            height: 18px
            position: absolute
            left: 1039.92px
            top: 0px
            border-radius: 2px
          children:
          - id: xem_them
            node_id: 1748:8585
            tag: p
            figma_type: TEXT
            figma_name: Xem thêm
            style:
              width: fit-content
              flex-shrink: 0
              height: auto
              color: '#595959'
              white-space: nowrap
              font-family: '''Reddit Sans'''
              font-size: 12px
              font-weight: 700
              line-height: 18px
              text-align: center
            text: Xem thêm
            token_hint:
              color: Text/text-secondary
              line-height: Tagline/Annotation/tagline-annotation-line-height
              letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
              font-weight: Label/Annotation/label-annotation-font-weight
              font-family: Font family/font-family
              font-size: Tagline/Annotation/tagline-annotation-font-size
          detached_from:
            component: Button
            note: 'Figma đặt tên `Button` nhưng node là FRAME, KHÔNG còn là INSTANCE của `Button` — đã detach. DỰNG THEO CÂY
              dưới đây, đừng thay bằng component `Button` của DS: hai bên có thể đã khác nhau.'
          token_hint:
            border-width: Stroke/stroke-divider
          behavior: BH-EXISTING
        token_hint:
          border-width: Stroke/stroke-divider
      token_hint:
        border-width: Stroke/stroke-divider
    - id: container_2
      node_id: 1748:8586
      tag: div
      figma_type: FRAME
      figma_name: Container
      style:
        width: 100%
        height: 0px
      token_hint:
        border-width: Stroke/stroke-divider
    token_hint:
      gap: Spacing/gap-x-small
      row-gap: Spacing/gap-x-small
      border-width: Stroke/stroke-divider
  - id: container_2
    node_id: 1748:8587
    tag: div
    figma_type: FRAME
    figma_name: Container
    style:
      display: flex
      flex-direction: row
      justify-content: center
      align-items: flex-start
      width: 100%
      height: 52px
      max-width: 1200px
    children:
    - id: container
      node_id: 1748:8588
      tag: div
      figma_type: FRAME
      figma_name: Container
      style:
        display: flex
        flex-direction: row
        align-items: center
        width: 1108px
        flex-shrink: 0
        align-self: stretch
        overflow: hidden
      children:
      - id: container
        node_id: 1748:8589
        tag: div
        figma_type: FRAME
        figma_name: Container
        style:
          display: flex
          flex-direction: row
          justify-content: center
          align-items: center
          width: fit-content
          flex-shrink: 0
          height: auto
          max-height: 32px
          background: '#ffffff'
        children:
        - id: link
          node_id: 1748:8590
          tag: div
          figma_type: FRAME
          figma_name: Link
          style:
            display: flex
            flex-direction: row
            gap: 2px
            justify-content: center
            align-items: center
            padding: 4px 12px 4px 12px
            width: 67.89px
            flex-shrink: 0
            height: 32px
            background: '#222222'
            border-radius: 9999px
          children:
          - id: icon
            node_id: 1748:8591
            tag: div
            figma_type: FRAME
            figma_name: Icon
            style:
              width: 20px
              flex-shrink: 0
              height: 20px
              overflow: hidden
              position: relative
            token_hint:
              border-width: Stroke/stroke-divider
            glyph:
              kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
              vector_nodes:
              - 1748:8592
          - id: loc
            node_id: 1748:8593
            tag: p
            figma_type: TEXT
            figma_name: Lọc
            style:
              width: fit-content
              flex-shrink: 0
              height: auto
              color: '#ffffff'
              white-space: nowrap
              font-family: '''Reddit Sans'''
              font-size: 14px
              font-weight: 500
              line-height: 20px
              text-align: center
            text: Lọc
            token_hint:
              color: Text/text-blank
              line-height: Tagline/Section/tagline-section-line-height
              letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
              font-weight: Tagline/Footext/tagline-footext-font-weight
              font-family: Font family/font-family
              font-size: Tagline/Caption/tagline-caption-font-size
          token_hint:
            gap: Spacing/gap-min
            padding-left: Padding/padding-small
            padding-top: Padding/padding-2x-small
            padding-right: Padding/padding-small
            padding-bottom: Padding/padding-2x-small
            row-gap: Spacing/gap-min
            border-width: Stroke/stroke-divider
            background: Icon/icon-on-background
          behavior: BH-EXISTING
        token_hint:
          border-width: Stroke/stroke-divider
          background: Background/Light/background-primary
      - id: container_2
        node_id: 1748:8594
        tag: div
        figma_type: FRAME
        figma_name: Container
        style:
          display: flex
          flex-direction: column
          align-items: flex-start
          width: 950px
          flex-shrink: 0
          height: 52px
          position: relative
        children:
        - id: container
          node_id: 1748:8595
          tag: div
          figma_type: FRAME
          figma_name: Container
          style:
            display: flex
            flex-direction: row
            gap: 8px
            align-items: flex-start
            padding: 10px 0px 10px 0px
            width: 100%
            height: 52px
            overflow: hidden
          children:
          - id: text
            node_id: 1748:8596
            tag: div
            figma_type: FRAME
            figma_name: Text
            style:
              width: 0px
              flex-shrink: 0
              align-self: stretch
            token_hint:
              border-width: Stroke/stroke-divider
          - id: link
            node_id: 1748:8597
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 100.11px
              flex-shrink: 0
              height: 32px
              background: '#222222'
              border-radius: 9999px
            children:
            - id: mua_ban
              node_id: 1748:8598
              tag: p
              figma_type: TEXT
              figma_name: Mua bán
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#ffffff'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: Mua bán
              token_hint:
                color: Text/text-blank
                line-height: Tagline/Section/tagline-section-line-height
                letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            - id: icon
              node_id: 1748:8599
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1748:8601
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Icon/icon-on-background
            behavior: BH-EXISTING
          - id: link_2
            node_id: 1748:8602
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 152.84px
              flex-shrink: 0
              align-self: stretch
              background: '#222222'
              border-radius: 9999px
            children:
            - id: can_ho_chung_cu
              node_id: 1748:8603
              tag: p
              figma_type: TEXT
              figma_name: Căn hộ/Chung cư
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#ffffff'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: Căn hộ/Chung cư
              token_hint:
                color: Text/text-blank
                line-height: Tagline/Section/tagline-section-line-height
                letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            - id: button
              node_id: 1748:8605
              tag: div
              figma_type: FRAME
              figma_name: Button
              style:
                display: flex
                flex-direction: row
                justify-content: center
                align-items: center
                padding: 12px
                width: 38px
                flex-shrink: 0
                height: 38px
                position: absolute
                left: 114.92px
                top: -3px
                border-radius: 2px
              children:
              - id: icon
                node_id: 1748:8606
                tag: div
                figma_type: FRAME
                figma_name: Icon
                style:
                  width: 100%
                  height: 14px
                  overflow: hidden
                  position: relative
                  background: '#8c8c8c'
                  border-radius: 14px
                measured_width_px: 14.0
                token_hint:
                  border-width: Stroke/stroke-divider
                  background: Icon/icon-tertiary
                glyph:
                  kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                  vector_nodes:
                  - 1748:8607
              detached_from:
                component: Button
                note: 'Figma đặt tên `Button` nhưng node là FRAME, KHÔNG còn là INSTANCE của `Button` — đã detach. DỰNG THEO
                  CÂY dưới đây, đừng thay bằng component `Button` của DS: hai bên có thể đã khác nhau.'
              token_hint:
                padding-left: Padding/padding-small
                padding-top: Padding/padding-small
                padding-right: Padding/padding-small
                padding-bottom: Padding/padding-small
                border-width: Stroke/stroke-divider
              behavior: BH-EXISTING
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Icon/icon-on-background
            behavior: BH-EXISTING
          - id: noxh_applied_chip_slot_dt
            node_id: 1748:8608
            tag: div
            figma_type: FRAME
            figma_name: noxh_applied_chip_slot_dt
            style:
              display: flex
              flex-direction: column
              gap: 4px
              align-items: flex-start
              width: fit-content
              flex-shrink: 0
              height: 32px
            children:
            - id: noxh_applied_chip_dt
              node_id: 1748:8609
              tag: div
              figma_type: FRAME
              figma_name: noxh_applied_chip_dt
              style:
                display: flex
                flex-direction: row
                gap: 2px
                justify-content: center
                align-items: center
                padding: 4px 12px 4px 12px
                flex: '1'
                width: 100%
                height: 100%
                background: '#222222'
                border-radius: 9999px
              children:
              - id: noxh_applied_chip_label_dt
                node_id: 1748:8610
                tag: p
                figma_type: TEXT
                figma_name: noxh_applied_chip_label_dt
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#ffffff'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Nhà ở xã hội
                token_hint:
                  color: Text/text-blank
                  line-height: Tagline/Section/tagline-section-line-height
                  letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
                not_a_control: true
              - id: button
                node_id: 1748:8612
                tag: div
                figma_type: FRAME
                figma_name: Button
                style:
                  display: flex
                  flex-direction: row
                  justify-content: center
                  align-items: center
                  padding: 12px
                  width: 38px
                  flex-shrink: 0
                  height: 38px
                  position: absolute
                  left: 85px
                  top: -3px
                  border-radius: 2px
                children:
                - id: icon
                  node_id: 1748:8613
                  tag: div
                  figma_type: FRAME
                  figma_name: Icon
                  style:
                    width: 100%
                    height: 14px
                    overflow: hidden
                    position: relative
                    background: '#8c8c8c'
                    border-radius: 14px
                  measured_width_px: 14.0
                  token_hint:
                    border-width: Stroke/stroke-divider
                    background: Icon/icon-tertiary
                  glyph:
                    kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                    vector_nodes:
                    - 1748:8614
                detached_from:
                  component: Button
                  note: 'Figma đặt tên `Button` nhưng node là FRAME, KHÔNG còn là INSTANCE của `Button` — đã detach. DỰNG
                    THEO CÂY dưới đây, đừng thay bằng component `Button` của DS: hai bên có thể đã khác nhau.'
                token_hint:
                  padding-left: Padding/padding-small
                  padding-top: Padding/padding-small
                  padding-right: Padding/padding-small
                  padding-bottom: Padding/padding-small
                  border-width: Stroke/stroke-divider
              token_hint:
                gap: Spacing/gap-min
                padding-left: Padding/padding-small
                padding-top: Padding/padding-2x-small
                padding-right: Padding/padding-small
                padding-bottom: Padding/padding-2x-small
                row-gap: Spacing/gap-min
                border-width: Stroke/stroke-divider
                background: Icon/icon-on-background
              behavior: BH-NOXH-CHIP-REMOVE
              change: NEW state — applied NOXH filter chip
              data:
                source: selected apartment_type value label (server)
                note: chip shows whichever apartment_type value is applied; here Nhà ở xã hội
          - id: link_3
            node_id: 1748:8615
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 94.3px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1748:8616
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: gia_ban
                node_id: 1748:8617
                tag: p
                figma_type: TEXT
                figma_name: Giá bán
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Giá bán
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Section/tagline-section-line-height
                  letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1748:8618
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1748:8620
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_4
            node_id: 1748:8621
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 82.62px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1748:8622
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: du_an
                node_id: 1748:8623
                tag: p
                figma_type: TEXT
                figma_name: Dự án
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Dự án
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Section/tagline-section-line-height
                  letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1748:8624
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1748:8626
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_5
            node_id: 1748:8627
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 133.67px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1748:8628
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: so_phong_ngu
                node_id: 1748:8629
                tag: p
                figma_type: TEXT
                figma_name: Số phòng ngủ
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Số phòng ngủ
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Section/tagline-section-line-height
                  letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1748:8630
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1748:8632
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_6
            node_id: 1748:8633
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 153.3px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1748:8634
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: so_phong_ve_sinh
                node_id: 1748:8635
                tag: p
                figma_type: TEXT
                figma_name: Số phòng vệ sinh
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Số phòng vệ sinh
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Section/tagline-section-line-height
                  letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1748:8636
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1748:8638
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_7
            node_id: 1748:8639
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 101.11px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1748:8640
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: dien_tich
                node_id: 1748:8641
                tag: p
                figma_type: TEXT
                figma_name: Diện tích
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Diện tích
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Section/tagline-section-line-height
                  letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1748:8642
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1748:8644
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_8
            node_id: 1748:8645
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 141.44px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1748:8646
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: don_gia_tr_m2
                node_id: 1748:8647
                tag: p
                figma_type: TEXT
                figma_name: Đơn giá (tr/m2)
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Đơn giá (tr/m2)
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Section/tagline-section-line-height
                  letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1748:8648
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1748:8650
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_9
            node_id: 1748:8651
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 150.05px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1748:8652
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: huong_ban_cong
                node_id: 1748:8653
                tag: p
                figma_type: TEXT
                figma_name: Hướng ban công
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Hướng ban công
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Section/tagline-section-line-height
                  letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1748:8654
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1748:8656
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          - id: link_10
            node_id: 1748:8657
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              gap: 2px
              justify-content: center
              align-items: center
              padding: 4px 12px 4px 12px
              width: 102.72px
              flex-shrink: 0
              height: 32px
              background: '#f4f4f4'
              border-radius: 9999px
            children:
            - id: text
              node_id: 1748:8658
              tag: div
              figma_type: FRAME
              figma_name: Text
              style:
                display: flex
                flex-direction: column
                align-items: center
                width: fit-content
                flex-shrink: 0
                height: auto
              children:
              - id: dang_boi
                node_id: 1748:8659
                tag: p
                figma_type: TEXT
                figma_name: Đăng bởi
                style:
                  width: fit-content
                  flex-shrink: 0
                  height: auto
                  color: '#222222'
                  white-space: nowrap
                  font-family: '''Reddit Sans'''
                  font-size: 14px
                  font-weight: 500
                  line-height: 20px
                  text-align: center
                text: Đăng bởi
                token_hint:
                  color: Text/text-primary
                  line-height: Tagline/Section/tagline-section-line-height
                  letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                  font-weight: Tagline/Footext/tagline-footext-font-weight
                  font-family: Font family/font-family
                  font-size: Tagline/Caption/tagline-caption-font-size
              token_hint:
                border-width: Stroke/stroke-divider
            - id: icon
              node_id: 1748:8660
              tag: div
              figma_type: FRAME
              figma_name: Icon
              style:
                width: 20px
                flex-shrink: 0
                height: 20px
                overflow: hidden
                position: relative
              token_hint:
                border-width: Stroke/stroke-divider
              glyph:
                kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
                vector_nodes:
                - 1748:8662
            token_hint:
              gap: Spacing/gap-min
              padding-left: Padding/padding-small
              padding-top: Padding/padding-2x-small
              padding-right: Padding/padding-small
              padding-bottom: Padding/padding-2x-small
              row-gap: Spacing/gap-min
              border-width: Stroke/stroke-divider
              background: Skeleton/skeleton-background
            behavior: BH-EXISTING
          measured_width_px: 950.0
          token_hint:
            gap: Spacing/gap-x-small
            row-gap: Spacing/gap-x-small
            border-width: Stroke/stroke-divider
        - id: button_next
          node_id: 1748:8663
          tag: div
          figma_type: FRAME
          figma_name: Button - Next
          style:
            display: flex
            flex-direction: row
            align-items: center
            width: 28px
            flex-shrink: 0
            height: 28px
            position: absolute
            left: 962px
            top: 12px
          children:
          - id: icon
            node_id: 1748:8664
            tag: div
            figma_type: FRAME
            figma_name: Icon
            style:
              width: 14px
              flex-shrink: 0
              height: 14px
              overflow: hidden
              position: relative
            token_hint:
              border-width: Stroke/stroke-divider
            glyph:
              kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
              vector_nodes:
              - 1748:8666
          token_hint:
            border-width: Stroke/stroke-divider
          behavior: BH-EXISTING
        token_hint:
          border-width: Stroke/stroke-divider
      token_hint:
        border-width: Stroke/stroke-divider
    - id: button
      node_id: 1748:8667
      tag: div
      figma_type: FRAME
      figma_name: Button
      style:
        display: flex
        flex-direction: row
        align-items: center
        width: 52px
        flex-shrink: 0
        height: 32px
        box-shadow: inset 0 0 0 1px rgba(0, 0, 0, 0.0)
        border-radius: 8px
      children:
      - id: xoa_loc
        node_id: 1748:8668
        tag: p
        figma_type: TEXT
        figma_name: Xoá lọc
        style:
          width: fit-content
          flex-shrink: 0
          height: auto
          color: '#222222'
          white-space: nowrap
          font-family: '''Reddit Sans'''
          font-size: 14px
          font-weight: 700
          line-height: 20px
          text-align: center
        text: Xoá lọc
        token_hint:
          color: Text/text-primary
          line-height: Tagline/Section/tagline-section-line-height
          letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
          font-weight: Label/Annotation/label-annotation-font-weight
          font-family: Font family/font-family
          font-size: Tagline/Caption/tagline-caption-font-size
      detached_from:
        component: Button
        note: 'Figma đặt tên `Button` nhưng node là FRAME, KHÔNG còn là INSTANCE của `Button` — đã detach. DỰNG THEO CÂY dưới
          đây, đừng thay bằng component `Button` của DS: hai bên có thể đã khác nhau.'
      token_hint:
        border-width: Stroke/stroke-divider
        border-radius: Radius/radius-card-small
      behavior: BH-EXISTING
    token_hint:
      border-width: Stroke/stroke-divider
  - id: container_3
    node_id: 1748:8669
    tag: div
    figma_type: FRAME
    figma_name: Container
    style:
      display: flex
      flex-direction: column
      align-items: flex-start
      width: 100%
      height: 96px
      overflow: hidden
    children:
    - id: container
      node_id: 1748:8670
      tag: div
      figma_type: FRAME
      figma_name: Container
      style:
        display: flex
        flex-direction: row
        gap: 12px
        align-items: center
        width: 100%
        height: auto
      children:
      - id: khu_vuc
        node_id: 1748:8672
        tag: p
        figma_type: TEXT
        figma_name: 'Khu vực:'
        style:
          width: fit-content
          flex-shrink: 0
          height: auto
          color: '#595959'
          white-space: nowrap
          font-family: '''Reddit Sans'''
          font-size: 14px
          font-weight: 500
          line-height: 20px
        text: 'Khu vực: '
        token_hint:
          color: Text/text-secondary
          line-height: Tagline/Section/tagline-section-line-height
          letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
          font-weight: Tagline/Footext/tagline-footext-font-weight
          font-family: Font family/font-family
          font-size: Tagline/Caption/tagline-caption-font-size
      - id: container
        node_id: 1748:8673
        tag: div
        figma_type: FRAME
        figma_name: Container
        style:
          display: flex
          flex-direction: row
          gap: 8px
          align-items: flex-start
          padding: 8px 12px 8px 0px
          width: 1093.97px
          flex-shrink: 0
          height: 48px
          overflow: hidden
        children:
        - id: link
          node_id: 1748:8674
          tag: div
          figma_type: FRAME
          figma_name: Link
          style:
            display: flex
            flex-direction: row
            gap: 2px
            justify-content: center
            align-items: center
            padding: 4px 12px 4px 12px
            width: 121.22px
            flex-shrink: 0
            height: 32px
            min-width: 32px
            min-height: 32px
            background: '#ffffff'
            box-shadow: 'inset 0 0 0 1px #dadada'
            border-radius: 9999px
          children:
          - id: link
            node_id: 1748:8675
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              align-items: center
              width: fit-content
              flex-shrink: 0
              height: auto
            children:
            - id: tp_ho_chi_minh
              node_id: 1748:8676
              tag: p
              figma_type: TEXT
              figma_name: Tp Hồ Chí Minh
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#000000'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: Tp Hồ Chí Minh
              token_hint:
                line-height: Tagline/Section/tagline-section-line-height
                letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            token_hint:
              border-width: Stroke/stroke-divider
            behavior: BH-EXISTING
          token_hint:
            gap: Spacing/gap-min
            padding-left: Padding/padding-small
            padding-top: Padding/padding-2x-small
            padding-right: Padding/padding-small
            padding-bottom: Padding/padding-2x-small
            row-gap: Spacing/gap-min
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
          behavior: BH-EXISTING
        - id: link_2
          node_id: 1748:8677
          tag: div
          figma_type: FRAME
          figma_name: Link
          style:
            display: flex
            flex-direction: row
            gap: 2px
            justify-content: center
            align-items: center
            padding: 4px 12px 4px 12px
            width: 234.16px
            flex-shrink: 0
            height: 32px
            min-width: 32px
            min-height: 32px
            background: '#ffffff'
            box-shadow: 'inset 0 0 0 1px #dadada'
            border-radius: 9999px
          children:
          - id: link
            node_id: 1748:8678
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              align-items: center
              width: fit-content
              flex-shrink: 0
              height: auto
            children:
            - id: binh_duong_tp_ho_chi_minh_moi
              node_id: 1748:8679
              tag: p
              figma_type: TEXT
              figma_name: Bình Dương (TP Hồ Chí Minh mới)
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#000000'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: Bình Dương (TP Hồ Chí Minh mới)
              token_hint:
                line-height: Tagline/Section/tagline-section-line-height
                letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            token_hint:
              border-width: Stroke/stroke-divider
            behavior: BH-EXISTING
          token_hint:
            gap: Spacing/gap-min
            padding-left: Padding/padding-small
            padding-top: Padding/padding-2x-small
            padding-right: Padding/padding-small
            padding-bottom: Padding/padding-2x-small
            row-gap: Spacing/gap-min
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
          behavior: BH-EXISTING
        - id: link_3
          node_id: 1748:8680
          tag: div
          figma_type: FRAME
          figma_name: Link
          style:
            display: flex
            flex-direction: row
            gap: 2px
            justify-content: center
            align-items: center
            padding: 4px 12px 4px 12px
            width: 270.98px
            flex-shrink: 0
            height: 32px
            min-width: 32px
            min-height: 32px
            background: '#ffffff'
            box-shadow: 'inset 0 0 0 1px #dadada'
            border-radius: 9999px
          children:
          - id: link
            node_id: 1748:8681
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              align-items: center
              width: fit-content
              flex-shrink: 0
              height: auto
            children:
            - id: ba_ria_vung_tau_tp_ho_chi_minh_moi
              node_id: 1748:8682
              tag: p
              figma_type: TEXT
              figma_name: Bà Rịa - Vũng Tàu (TP Hồ Chí Minh mới)
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#000000'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: Bà Rịa - Vũng Tàu (TP Hồ Chí Minh mới)
              token_hint:
                line-height: Tagline/Section/tagline-section-line-height
                letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            token_hint:
              border-width: Stroke/stroke-divider
            behavior: BH-EXISTING
          token_hint:
            gap: Spacing/gap-min
            padding-left: Padding/padding-small
            padding-top: Padding/padding-2x-small
            padding-right: Padding/padding-small
            padding-bottom: Padding/padding-2x-small
            row-gap: Spacing/gap-min
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
          behavior: BH-EXISTING
        - id: link_4
          node_id: 1748:8683
          tag: div
          figma_type: FRAME
          figma_name: Link
          style:
            display: flex
            flex-direction: row
            gap: 2px
            justify-content: center
            align-items: center
            padding: 4px 12px 4px 12px
            width: 68.73px
            flex-shrink: 0
            height: 32px
            min-width: 32px
            min-height: 32px
            background: '#ffffff'
            box-shadow: 'inset 0 0 0 1px #dadada'
            border-radius: 9999px
          children:
          - id: link
            node_id: 1748:8684
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              align-items: center
              width: fit-content
              flex-shrink: 0
              height: auto
            children:
            - id: ha_noi
              node_id: 1748:8685
              tag: p
              figma_type: TEXT
              figma_name: Hà Nội
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#000000'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: Hà Nội
              token_hint:
                line-height: Tagline/Section/tagline-section-line-height
                letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            token_hint:
              border-width: Stroke/stroke-divider
            behavior: BH-EXISTING
          token_hint:
            gap: Spacing/gap-min
            padding-left: Padding/padding-small
            padding-top: Padding/padding-2x-small
            padding-right: Padding/padding-small
            padding-bottom: Padding/padding-2x-small
            row-gap: Spacing/gap-min
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
          behavior: BH-EXISTING
        - id: link_5
          node_id: 1748:8686
          tag: div
          figma_type: FRAME
          figma_name: Link
          style:
            display: flex
            flex-direction: row
            gap: 2px
            justify-content: center
            align-items: center
            padding: 4px 12px 4px 12px
            width: 80.77px
            flex-shrink: 0
            height: 32px
            min-width: 32px
            min-height: 32px
            background: '#ffffff'
            box-shadow: 'inset 0 0 0 1px #dadada'
            border-radius: 9999px
          children:
          - id: link
            node_id: 1748:8687
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              align-items: center
              width: fit-content
              flex-shrink: 0
              height: auto
            children:
            - id: da_nang
              node_id: 1748:8688
              tag: p
              figma_type: TEXT
              figma_name: Đà Nẵng
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#000000'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: Đà Nẵng
              token_hint:
                line-height: Tagline/Section/tagline-section-line-height
                letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            token_hint:
              border-width: Stroke/stroke-divider
            behavior: BH-EXISTING
          token_hint:
            gap: Spacing/gap-min
            padding-left: Padding/padding-small
            padding-top: Padding/padding-2x-small
            padding-right: Padding/padding-small
            padding-bottom: Padding/padding-2x-small
            row-gap: Spacing/gap-min
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
          behavior: BH-EXISTING
        - id: link_6
          node_id: 1748:8689
          tag: div
          figma_type: FRAME
          figma_name: Link
          style:
            display: flex
            flex-direction: row
            gap: 2px
            justify-content: center
            align-items: center
            padding: 4px 12px 4px 12px
            width: 93.34px
            flex-shrink: 0
            height: 32px
            min-width: 32px
            min-height: 32px
            background: '#ffffff'
            box-shadow: 'inset 0 0 0 1px #dadada'
            border-radius: 9999px
          children:
          - id: image_nearby
            node_id: 1748:8690
            tag: div
            figma_type: FRAME
            figma_name: Image (nearby)
            style:
              width: 19.3px
              flex-shrink: 0
              height: 20px
              overflow: hidden
              position: relative
            token_hint:
              border-width: Stroke/stroke-divider
            glyph:
              kind: 'non-DS vector glyph (existing app asset: chevron / clear / filter)'
              vector_nodes:
              - 1748:8691
          - id: gan_toi
            node_id: 1748:8692
            tag: p
            figma_type: TEXT
            figma_name: Gần tôi
            style:
              width: fit-content
              flex-shrink: 0
              height: auto
              color: '#222222'
              white-space: nowrap
              font-family: '''Reddit Sans'''
              font-size: 14px
              font-weight: 500
              line-height: 20px
              text-align: center
            text: Gần tôi
            token_hint:
              color: Text/text-primary
              line-height: Tagline/Section/tagline-section-line-height
              letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
              font-weight: Tagline/Footext/tagline-footext-font-weight
              font-family: Font family/font-family
              font-size: Tagline/Caption/tagline-caption-font-size
          token_hint:
            gap: Spacing/gap-min
            padding-left: Padding/padding-small
            padding-top: Padding/padding-2x-small
            padding-right: Padding/padding-small
            padding-bottom: Padding/padding-2x-small
            row-gap: Spacing/gap-min
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
          behavior: BH-EXISTING
        token_hint:
          gap: Spacing/gap-x-small
          padding-top: Padding/padding-x-small
          padding-right: Padding/padding-small
          padding-bottom: Padding/padding-x-small
          row-gap: Spacing/gap-x-small
          border-width: Stroke/stroke-divider
      token_hint:
        gap: Spacing/gap-small
        row-gap: Spacing/gap-small
        border-width: Stroke/stroke-divider
    - id: container_2
      node_id: 1748:8693
      tag: div
      figma_type: FRAME
      figma_name: Container
      style:
        display: flex
        flex-direction: row
        gap: 12px
        align-items: center
        width: 100%
        height: auto
      children:
      - id: gia_ban
        node_id: 1748:8695
        tag: p
        figma_type: TEXT
        figma_name: 'Giá bán:'
        style:
          width: fit-content
          flex-shrink: 0
          height: auto
          color: '#595959'
          white-space: nowrap
          font-family: '''Reddit Sans'''
          font-size: 14px
          font-weight: 500
          line-height: 20px
        text: 'Giá bán:'
        token_hint:
          color: Text/text-secondary
          line-height: Tagline/Section/tagline-section-line-height
          letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
          font-weight: Tagline/Footext/tagline-footext-font-weight
          font-family: Font family/font-family
          font-size: Tagline/Caption/tagline-caption-font-size
      - id: container
        node_id: 1748:8696
        tag: div
        figma_type: FRAME
        figma_name: Container
        style:
          display: flex
          flex-direction: row
          gap: 8px
          align-items: flex-start
          padding: 8px 12px 8px 0px
          width: 1096.3px
          flex-shrink: 0
          height: 48px
          overflow: hidden
        children:
        - id: link
          node_id: 1748:8697
          tag: div
          figma_type: FRAME
          figma_name: Link
          style:
            display: flex
            flex-direction: row
            gap: 2px
            justify-content: center
            align-items: center
            padding: 4px 12px 4px 12px
            width: 80.72px
            flex-shrink: 0
            height: 32px
            min-width: 32px
            min-height: 32px
            background: '#ffffff'
            box-shadow: 'inset 0 0 0 1px #dadada'
            border-radius: 9999px
          children:
          - id: link
            node_id: 1748:8698
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              align-items: center
              width: fit-content
              flex-shrink: 0
              height: auto
            children:
            - id: duoi_1_ty
              node_id: 1748:8699
              tag: p
              figma_type: TEXT
              figma_name: Dưới 1 tỷ
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#000000'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: Dưới 1 tỷ
              token_hint:
                line-height: Tagline/Section/tagline-section-line-height
                letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            token_hint:
              border-width: Stroke/stroke-divider
            behavior: BH-EXISTING
          token_hint:
            gap: Spacing/gap-min
            padding-left: Padding/padding-small
            padding-top: Padding/padding-2x-small
            padding-right: Padding/padding-small
            padding-bottom: Padding/padding-2x-small
            row-gap: Spacing/gap-min
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
          behavior: BH-EXISTING
        - id: link_2
          node_id: 1748:8700
          tag: div
          figma_type: FRAME
          figma_name: Link
          style:
            display: flex
            flex-direction: row
            gap: 2px
            justify-content: center
            align-items: center
            padding: 4px 12px 4px 12px
            width: 68.08px
            flex-shrink: 0
            height: 32px
            min-width: 32px
            min-height: 32px
            background: '#ffffff'
            box-shadow: 'inset 0 0 0 1px #dadada'
            border-radius: 9999px
          children:
          - id: link
            node_id: 1748:8701
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              align-items: center
              width: fit-content
              flex-shrink: 0
              height: auto
            children:
            - id: 1_2_ty
              node_id: 1748:8702
              tag: p
              figma_type: TEXT
              figma_name: 1 - 2 tỷ
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#000000'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: 1 - 2 tỷ
              token_hint:
                line-height: Tagline/Section/tagline-section-line-height
                letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            token_hint:
              border-width: Stroke/stroke-divider
            behavior: BH-EXISTING
          token_hint:
            gap: Spacing/gap-min
            padding-left: Padding/padding-small
            padding-top: Padding/padding-2x-small
            padding-right: Padding/padding-small
            padding-bottom: Padding/padding-2x-small
            row-gap: Spacing/gap-min
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
          behavior: BH-EXISTING
        - id: link_3
          node_id: 1748:8703
          tag: div
          figma_type: FRAME
          figma_name: Link
          style:
            display: flex
            flex-direction: row
            gap: 2px
            justify-content: center
            align-items: center
            padding: 4px 12px 4px 12px
            width: 70.53px
            flex-shrink: 0
            height: 32px
            min-width: 32px
            min-height: 32px
            background: '#ffffff'
            box-shadow: 'inset 0 0 0 1px #dadada'
            border-radius: 9999px
          children:
          - id: link
            node_id: 1748:8704
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              align-items: center
              width: fit-content
              flex-shrink: 0
              height: auto
            children:
            - id: 2_3_ty
              node_id: 1748:8705
              tag: p
              figma_type: TEXT
              figma_name: 2 - 3 tỷ
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#000000'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: 2 - 3 tỷ
              token_hint:
                line-height: Tagline/Section/tagline-section-line-height
                letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            token_hint:
              border-width: Stroke/stroke-divider
            behavior: BH-EXISTING
          token_hint:
            gap: Spacing/gap-min
            padding-left: Padding/padding-small
            padding-top: Padding/padding-2x-small
            padding-right: Padding/padding-small
            padding-bottom: Padding/padding-2x-small
            row-gap: Spacing/gap-min
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
          behavior: BH-EXISTING
        - id: link_4
          node_id: 1748:8706
          tag: div
          figma_type: FRAME
          figma_name: Link
          style:
            display: flex
            flex-direction: row
            gap: 2px
            justify-content: center
            align-items: center
            padding: 4px 12px 4px 12px
            width: 70.62px
            flex-shrink: 0
            height: 32px
            min-width: 32px
            min-height: 32px
            background: '#ffffff'
            box-shadow: 'inset 0 0 0 1px #dadada'
            border-radius: 9999px
          children:
          - id: link
            node_id: 1748:8707
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              align-items: center
              width: fit-content
              flex-shrink: 0
              height: auto
            children:
            - id: 3_5_ty
              node_id: 1748:8708
              tag: p
              figma_type: TEXT
              figma_name: 3 - 5 tỷ
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#000000'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: 3 - 5 tỷ
              token_hint:
                line-height: Tagline/Section/tagline-section-line-height
                letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            token_hint:
              border-width: Stroke/stroke-divider
            behavior: BH-EXISTING
          token_hint:
            gap: Spacing/gap-min
            padding-left: Padding/padding-small
            padding-top: Padding/padding-2x-small
            padding-right: Padding/padding-small
            padding-bottom: Padding/padding-2x-small
            row-gap: Spacing/gap-min
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
          behavior: BH-EXISTING
        - id: link_5
          node_id: 1748:8709
          tag: div
          figma_type: FRAME
          figma_name: Link
          style:
            display: flex
            flex-direction: row
            gap: 2px
            justify-content: center
            align-items: center
            padding: 4px 12px 4px 12px
            width: 70.05px
            flex-shrink: 0
            height: 32px
            min-width: 32px
            min-height: 32px
            background: '#ffffff'
            box-shadow: 'inset 0 0 0 1px #dadada'
            border-radius: 9999px
          children:
          - id: link
            node_id: 1748:8710
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              align-items: center
              width: fit-content
              flex-shrink: 0
              height: auto
            children:
            - id: 5_7_ty
              node_id: 1748:8711
              tag: p
              figma_type: TEXT
              figma_name: 5 - 7 tỷ
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#000000'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: 5 - 7 tỷ
              token_hint:
                line-height: Tagline/Section/tagline-section-line-height
                letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            token_hint:
              border-width: Stroke/stroke-divider
            behavior: BH-EXISTING
          token_hint:
            gap: Spacing/gap-min
            padding-left: Padding/padding-small
            padding-top: Padding/padding-2x-small
            padding-right: Padding/padding-small
            padding-bottom: Padding/padding-2x-small
            row-gap: Spacing/gap-min
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
          behavior: BH-EXISTING
        - id: link_6
          node_id: 1748:8712
          tag: div
          figma_type: FRAME
          figma_name: Link
          style:
            display: flex
            flex-direction: row
            gap: 2px
            justify-content: center
            align-items: center
            padding: 4px 12px 4px 12px
            width: 76.64px
            flex-shrink: 0
            height: 32px
            min-width: 32px
            min-height: 32px
            background: '#ffffff'
            box-shadow: 'inset 0 0 0 1px #dadada'
            border-radius: 9999px
          children:
          - id: link
            node_id: 1748:8713
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              align-items: center
              width: fit-content
              flex-shrink: 0
              height: auto
            children:
            - id: 7_10_ty
              node_id: 1748:8714
              tag: p
              figma_type: TEXT
              figma_name: 7 - 10 tỷ
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#000000'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: 7 - 10 tỷ
              token_hint:
                line-height: Tagline/Section/tagline-section-line-height
                letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            token_hint:
              border-width: Stroke/stroke-divider
            behavior: BH-EXISTING
          token_hint:
            gap: Spacing/gap-min
            padding-left: Padding/padding-small
            padding-top: Padding/padding-2x-small
            padding-right: Padding/padding-small
            padding-bottom: Padding/padding-2x-small
            row-gap: Spacing/gap-min
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
          behavior: BH-EXISTING
        - id: link_7
          node_id: 1748:8715
          tag: div
          figma_type: FRAME
          figma_name: Link
          style:
            display: flex
            flex-direction: row
            gap: 2px
            justify-content: center
            align-items: center
            padding: 4px 12px 4px 12px
            width: 88.3px
            flex-shrink: 0
            height: 32px
            min-width: 32px
            min-height: 32px
            background: '#ffffff'
            box-shadow: 'inset 0 0 0 1px #dadada'
            border-radius: 9999px
          children:
          - id: link
            node_id: 1748:8716
            tag: div
            figma_type: FRAME
            figma_name: Link
            style:
              display: flex
              flex-direction: row
              align-items: center
              width: fit-content
              flex-shrink: 0
              height: auto
            children:
            - id: tren_10_ty
              node_id: 1748:8717
              tag: p
              figma_type: TEXT
              figma_name: Trên 10 tỷ
              style:
                width: fit-content
                flex-shrink: 0
                height: auto
                color: '#000000'
                white-space: nowrap
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 500
                line-height: 20px
                text-align: center
              text: Trên 10 tỷ
              token_hint:
                line-height: Tagline/Section/tagline-section-line-height
                letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
                font-weight: Tagline/Footext/tagline-footext-font-weight
                font-family: Font family/font-family
                font-size: Tagline/Caption/tagline-caption-font-size
            token_hint:
              border-width: Stroke/stroke-divider
            behavior: BH-EXISTING
          token_hint:
            gap: Spacing/gap-min
            padding-left: Padding/padding-small
            padding-top: Padding/padding-2x-small
            padding-right: Padding/padding-small
            padding-bottom: Padding/padding-2x-small
            row-gap: Spacing/gap-min
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
          behavior: BH-EXISTING
        token_hint:
          gap: Spacing/gap-x-small
          padding-top: Padding/padding-x-small
          padding-right: Padding/padding-small
          padding-bottom: Padding/padding-x-small
          row-gap: Spacing/gap-x-small
          border-width: Stroke/stroke-divider
      token_hint:
        gap: Spacing/gap-small
        row-gap: Spacing/gap-small
        border-width: Stroke/stroke-divider
    token_hint:
      border-width: Stroke/stroke-divider
  measured_width_px: 1200.0
  token_hint:
    padding-left: Padding/padding-large
    padding-top: Padding/padding-medium
    padding-right: Padding/padding-large
    padding-bottom: Padding/padding-medium
    border-width: Stroke/stroke-divider
    border-radius: Radius/radius-card
    background: Background/Light/background-primary
```

### app (iOS/Android) — `sec_filter_bar_app` (Figma `1748:15616`)

```yaml
id: sec_filter_bar_app
spec_id: PTY.adlisting_apartment/noxh_applied
kind: screen_state
platform: app (iOS/Android)
figma_node: 1748:15616
requirement:
  traces_to:
  - FR-2.3
  - DR-1
  - PRD 5.1
  context: 'Destination after tapping the NOXH tile: subcate chip "Căn hộ/Chung cư ×" + applied chip "Nhà ở xã hội ×". Rest
    of the listing page is existing and unchanged (OUT of this spec).'
uses:
  behaviors:
  - BH-EXISTING
  - BH-NOXH-CHIP-REMOVE
related_behaviors:
- BH-BACK
- BH-ADTYPE-RESET
layout:
  id: sec_filter_bar_app
  node_id: 1748:15616
  tag: div
  figma_type: FRAME
  figma_name: sec_filter_bar_app
  style:
    display: flex
    flex-direction: column
    gap: 8px
    align-items: flex-start
    padding: 0px 16px 8px 16px
    width: 100%
    height: auto
    position: relative
  children:
  - id: filter_container
    node_id: 1748:15617
    tag: div
    figma_type: FRAME
    figma_name: Filter Container
    style:
      display: flex
      flex-direction: row
      gap: 16px
      align-items: center
      width: 100%
      height: 36px
    children:
    - id: location_filter
      node_id: 1748:15618
      tag: div
      figma_type: FRAME
      figma_name: Location Filter
      style:
        display: flex
        flex-direction: row
        gap: 4px
        align-items: center
        flex: '1'
        width: 100%
        align-self: stretch
      children:
      - id: location_icon
        node_id: 1748:15619
        tag: div
        figma_type: INSTANCE
        figma_name: Location Icon
        style:
          width: 20px
          flex-shrink: 0
          height: 20px
          overflow: hidden
          position: relative
        ds_component:
          figma_component: Location Icon
        icon: Location-fill
        asset_mode: none — see icon/data
        content:
          icon_only_instance: Location Icon
          note: no text; renders the DS component as-is
      - id: location_info
        node_id: 1748:15620
        tag: div
        figma_type: FRAME
        figma_name: Location Info
        style:
          display: flex
          flex-direction: row
          gap: 4px
          align-items: center
          flex: '1'
          width: 100%
          height: auto
        children:
        - id: location_label
          node_id: 1748:15621
          tag: p
          figma_type: TEXT
          figma_name: Location Label
          style:
            width: fit-content
            flex-shrink: 0
            height: auto
            color: '#222222'
            white-space: nowrap
            font-family: '''Reddit Sans'''
            font-size: 14px
            font-weight: 700
            line-height: 20px
            align-self: center
          text: 'Khu vực:'
          token_hint:
            color: Text/text-primary
            line-height: Label/Section/label-section-line-height
            letter-spacing: Label/Section/label-section-letter-spacing
            font-weight: Label/Section/label-section-font-weight
            font-family: Label/Section/label-section-font-family
            font-size: Label/Section/label-section-font-size
            typography: Label/label - section
        - id: location_address
          node_id: 1748:15622
          tag: p
          figma_type: TEXT
          figma_name: Location Address
          style:
            width: fit-content
            flex-shrink: 0
            height: auto
            color: '#fa6819'
            white-space: nowrap
            overflow: hidden
            text-overflow: ellipsis
            font-family: '''Reddit Sans'''
            font-size: 14px
            font-weight: 700
            line-height: 20px
            display: -webkit-box
            -webkit-line-clamp: 1
            -webkit-box-orient: vertical
            align-self: center
          text: Toàn quốc
          token_hint:
            color: Text/text-brand
            line-height: Label/Section/label-section-line-height
            letter-spacing: Label/Section/label-section-letter-spacing
            font-weight: Label/Section/label-section-font-weight
            font-family: Label/Section/label-section-font-family
            font-size: Label/Section/label-section-font-size
            typography: Label/label - section
        - id: dismisible
          node_id: 1748:15623
          tag: div
          figma_type: INSTANCE
          figma_name: dismisible
          style:
            width: 20px
            flex-shrink: 0
            height: 20px
            overflow: hidden
            position: relative
          ds_component:
            figma_component: dismisible
          icon: Chevrondown-outline
          asset_mode: none — see icon/data
          content:
            icon_only_instance: dismisible
            note: no text; renders the DS component as-is
    - id: reset_filter
      node_id: 1748:15624
      tag: div
      figma_type: FRAME
      figma_name: Reset filter
      style:
        display: flex
        flex-direction: row
        gap: 30px
        justify-content: center
        align-items: center
        width: fit-content
        flex-shrink: 0
        align-self: stretch
      children:
      - id: reset_filter
        node_id: 1748:15625
        tag: p
        figma_type: TEXT
        figma_name: Reset Filter
        style:
          width: fit-content
          flex-shrink: 0
          height: auto
          color: '#222222'
          white-space: nowrap
          font-family: '''Reddit Sans'''
          font-size: 14px
          font-weight: 600
          line-height: 20px
          align-self: center
        text: Xóa lọc
        token_hint:
          color: Text/text-primary
          line-height: Header/Caption/header-caption-line-height
          letter-spacing: Header/Caption/header-caption-letter-spacing
          font-weight: Header/Caption/header-caption-font-weight
          font-family: Header/Caption/header-caption-font-family
          font-size: Header/Caption/header-caption-font-size
          typography: Header/header-caption
      behavior: BH-EXISTING
  - id: filter_rows_app
    node_id: 1748:16280
    tag: div
    figma_type: FRAME
    figma_name: filter_rows_app
    style:
      display: flex
      flex-direction: column
      gap: 8px
      align-items: flex-start
      width: 100%
      height: auto
    children:
    - id: filter_tags_app
      node_id: 1748:16281
      tag: div
      figma_type: FRAME
      figma_name: filter_tags_app
      style:
        display: flex
        flex-direction: row
        gap: 4px
        align-items: center
        width: 100%
        height: auto
        overflow-x: auto
        scrollbar-width: none
      children:
      - id: chip
        node_id: 1748:16282
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 4px
          justify-content: center
          align-items: center
          padding: 6px 12px 6px 12px
          width: fit-content
          flex-shrink: 0
          height: 32px
          background: '#222222'
          border-radius: 999px
        ds_component:
          props:
            right_icon: false
            change_l_icon: 6:1559
            text: false
            left_icon: true
            size: Medium 32px
            style: Fill
            select: 'Yes'
            state: Default
          figma_component: Chip
        icon: Size=Medium 32px, Style=Fill, Select=Yes, State=Default
        token_hint:
          gap: Spacing/gap-2x-small-4
          border-width: Stroke/stroke-action
          border-radius: Radius/radius-pill
          background: Background/Solid/background-inverted
        asset_mode: none — see icon/data
        behavior: BH-EXISTING
        content:
          icon_only_instance: Chip
          note: no text; renders the DS component as-is
      - id: chip_2
        node_id: 1748:16283
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 4px
          justify-content: center
          align-items: center
          padding: 6px 12px 6px 12px
          width: 103.0px
          flex-shrink: 0
          height: 32.0px
          background: '#222222'
          border-radius: 999px
        ds_component:
          props:
            right_icon: true
            change_l_icon: 6:1559
            text: true
            left_icon: false
            size: Medium 32px
            style: Fill
            select: 'Yes'
            state: Default
          figma_component: Chip
        content:
          labels:
          - node_id: I1748:16283;44:8427
            text: Mua bán
            box:
            - 12
            - 6
            - 55
            - 20
            style:
              font-family: '''Reddit Sans'''
              font-size: 14px
              font-weight: 600
              line-height: 20px
              color: '#ffffff'
            token_hint:
              typography: Header/header-caption
              color: Text/text-blank
          glyphs:
          - node_id: I1748:16283;44:8428
            box:
            - 71
            - 6
            - 20
            - 20
            icon: Chevrondown-outline
        token_hint:
          gap: Spacing/gap-2x-small-4
          border-width: Stroke/stroke-action
          border-radius: Radius/radius-pill
          background: Background/Solid/background-inverted
        behavior: BH-EXISTING
      - id: link
        node_id: 1748:16284
        tag: div
        figma_type: FRAME
        figma_name: Link
        style:
          display: flex
          flex-direction: row
          gap: 2px
          justify-content: center
          align-items: center
          padding: 4px 12px 4px 12px
          width: 152.84px
          flex-shrink: 0
          height: 32px
          background: '#222222'
          border-radius: 9999px
        children:
        - id: can_ho_chung_cu
          node_id: 1748:16285
          tag: p
          figma_type: TEXT
          figma_name: Căn hộ/Chung cư
          style:
            width: fit-content
            flex-shrink: 0
            height: auto
            color: '#ffffff'
            white-space: nowrap
            font-family: '''Reddit Sans'''
            font-size: 14px
            font-weight: 500
            line-height: 20px
            text-align: center
          text: Căn hộ/Chung cư
          token_hint:
            color: Text/text-blank
            line-height: Tagline/Section/tagline-section-line-height
            letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
            font-weight: Tagline/Footext/tagline-footext-font-weight
            font-family: Font family/font-family
            font-size: Tagline/Caption/tagline-caption-font-size
        - id: close_outline
          node_id: 1748:16286
          tag: div
          figma_type: INSTANCE
          figma_name: Close-outline
          style:
            width: 20px
            flex-shrink: 0
            height: 20px
            position: relative
          ds_component:
            figma_component: Close-outline
          icon: Close-outline
          asset_mode: none — see icon/data
          content:
            icon_only_instance: Close-outline
            note: no text; renders the DS component as-is
        token_hint:
          gap: Spacing/gap-min
          padding-left: Padding/padding-small
          padding-top: Padding/padding-2x-small
          padding-right: Padding/padding-small
          padding-bottom: Padding/padding-2x-small
          row-gap: Spacing/gap-min
          border-width: Stroke/stroke-divider
          background: Icon/icon-primary
        behavior: BH-EXISTING
      - id: noxh_applied_chip_slot_app
        node_id: 1748:16287
        tag: div
        figma_type: FRAME
        figma_name: noxh_applied_chip_slot_app
        style:
          display: flex
          flex-direction: column
          gap: 4px
          align-items: flex-start
          width: fit-content
          flex-shrink: 0
          height: 32px
        children:
        - id: noxh_applied_chip_app
          node_id: 1748:16288
          tag: div
          figma_type: FRAME
          figma_name: noxh_applied_chip_app
          style:
            display: flex
            flex-direction: row
            gap: 2px
            justify-content: center
            align-items: center
            padding: 4px 12px 4px 12px
            flex: '1'
            width: 100%
            height: 100%
            background: '#222222'
            border-radius: 9999px
          children:
          - id: noxh_applied_chip_label_app
            node_id: 1748:16289
            tag: p
            figma_type: TEXT
            figma_name: noxh_applied_chip_label_app
            style:
              width: fit-content
              flex-shrink: 0
              height: auto
              color: '#ffffff'
              white-space: nowrap
              font-family: '''Reddit Sans'''
              font-size: 14px
              font-weight: 500
              line-height: 20px
              text-align: center
            text: Nhà ở xã hội
            token_hint:
              color: Text/text-blank
              line-height: Tagline/Section/tagline-section-line-height
              letter-spacing: Tagline/Footext/tagline-footext-letter-spacing
              font-weight: Tagline/Footext/tagline-footext-font-weight
              font-family: Font family/font-family
              font-size: Tagline/Caption/tagline-caption-font-size
            not_a_control: true
          - id: close_outline
            node_id: 1748:16290
            tag: div
            figma_type: INSTANCE
            figma_name: Close-outline
            style:
              width: 20px
              flex-shrink: 0
              height: 20px
              position: relative
            ds_component:
              figma_component: Close-outline
            icon: Close-outline
            asset_mode: none — see icon/data
            content:
              icon_only_instance: Close-outline
              note: no text; renders the DS component as-is
          token_hint:
            gap: Spacing/gap-min
            padding-left: Padding/padding-small
            padding-top: Padding/padding-2x-small
            padding-right: Padding/padding-small
            padding-bottom: Padding/padding-2x-small
            row-gap: Spacing/gap-min
            border-width: Stroke/stroke-divider
            background: Icon/icon-primary
          behavior: BH-NOXH-CHIP-REMOVE
          change: NEW state — applied NOXH filter chip
          data:
            source: selected apartment_type value label (server)
            note: chip shows whichever apartment_type value is applied; here Nhà ở xã hội
      - id: chip_3
        node_id: 1748:16291
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 4px
          justify-content: center
          align-items: center
          padding: 6px 12px 6px 12px
          width: 137.0px
          flex-shrink: 0
          height: 32.0px
          background: '#f4f4f4'
          border-radius: 999px
        ds_component:
          props:
            text: true
            change_l_icon: 3:1312
            right_icon: true
            left_icon: false
            size: Medium 32px
            style: Fill
            select: 'No'
            state: Default
          figma_component: Chip
        content:
          labels:
          - node_id: I1748:16291;44:8407
            text: Số phòng ngủ
            box:
            - 12
            - 6
            - 89
            - 20
            style:
              font-family: '''Reddit Sans'''
              font-size: 14px
              font-weight: 600
              line-height: 20px
              color: '#222222'
            token_hint:
              typography: Header/header-caption
              color: Text/text-primary
          glyphs:
          - node_id: I1748:16291;44:8408
            box:
            - 105
            - 6
            - 20
            - 20
            icon: Chevrondown-outline
        token_hint:
          gap: Spacing/gap-2x-small-4
          padding-left: Padding/padding-small-12
          padding-right: Padding/padding-small-12
          border-radius: Radius/radius-pill
          background: Background/Light/background-secondary
        behavior: BH-EXISTING
      - id: chip_4
        node_id: 1748:16292
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 4px
          justify-content: center
          align-items: center
          padding: 6px 12px 6px 12px
          width: 85.0px
          flex-shrink: 0
          height: 32.0px
          background: '#f4f4f4'
          border-radius: 999px
        ds_component:
          props:
            text: true
            change_l_icon: 3:1312
            right_icon: true
            left_icon: false
            size: Medium 32px
            style: Fill
            select: 'No'
            state: Default
          figma_component: Chip
        content:
          labels:
          - node_id: I1748:16292;44:8407
            text: Dự án
            box:
            - 12
            - 6
            - 37
            - 20
            style:
              font-family: '''Reddit Sans'''
              font-size: 14px
              font-weight: 600
              line-height: 20px
              color: '#222222'
            token_hint:
              typography: Header/header-caption
              color: Text/text-primary
          glyphs:
          - node_id: I1748:16292;44:8408
            box:
            - 53
            - 6
            - 20
            - 20
            icon: Chevrondown-outline
        token_hint:
          gap: Spacing/gap-2x-small-4
          padding-left: Padding/padding-small-12
          padding-right: Padding/padding-small-12
          border-radius: Radius/radius-pill
          background: Background/Light/background-secondary
        behavior: BH-EXISTING
      - id: chip_5
        node_id: 1748:16293
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 4px
          justify-content: center
          align-items: center
          padding: 6px 12px 6px 12px
          width: 157.0px
          flex-shrink: 0
          height: 32.0px
          background: '#f4f4f4'
          border-radius: 999px
        ds_component:
          props:
            text: true
            change_l_icon: 3:1312
            right_icon: true
            left_icon: false
            size: Medium 32px
            style: Fill
            select: 'No'
            state: Default
          figma_component: Chip
        content:
          labels:
          - node_id: I1748:16293;44:8407
            text: Số phòng vệ sinh
            box:
            - 12
            - 6
            - 109
            - 20
            style:
              font-family: '''Reddit Sans'''
              font-size: 14px
              font-weight: 600
              line-height: 20px
              color: '#222222'
            token_hint:
              typography: Header/header-caption
              color: Text/text-primary
          glyphs:
          - node_id: I1748:16293;44:8408
            box:
            - 125
            - 6
            - 20
            - 20
            icon: Chevrondown-outline
        token_hint:
          gap: Spacing/gap-2x-small-4
          padding-left: Padding/padding-small-12
          padding-right: Padding/padding-small-12
          border-radius: Radius/radius-pill
          background: Background/Light/background-secondary
        behavior: BH-EXISTING
      - id: chip_6
        node_id: 1748:16294
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 4px
          justify-content: center
          align-items: center
          padding: 6px 12px 6px 12px
          width: 117.0px
          flex-shrink: 0
          height: 32.0px
          background: '#f4f4f4'
          border-radius: 999px
        ds_component:
          props:
            text: true
            change_l_icon: 3:1312
            right_icon: true
            left_icon: false
            size: Medium 32px
            style: Fill
            select: 'No'
            state: Default
          figma_component: Chip
        content:
          labels:
          - node_id: I1748:16294;44:8407
            text: Hướng cửa
            box:
            - 12
            - 6
            - 69
            - 20
            style:
              font-family: '''Reddit Sans'''
              font-size: 14px
              font-weight: 600
              line-height: 20px
              color: '#222222'
            token_hint:
              typography: Header/header-caption
              color: Text/text-primary
          glyphs:
          - node_id: I1748:16294;44:8408
            box:
            - 85
            - 6
            - 20
            - 20
            icon: Chevrondown-outline
        token_hint:
          gap: Spacing/gap-2x-small-4
          padding-left: Padding/padding-small-12
          padding-right: Padding/padding-small-12
          border-radius: Radius/radius-pill
          background: Background/Light/background-secondary
        behavior: BH-EXISTING
    - id: quick_filter_row_khu_vuc
      node_id: 1748:16295
      tag: div
      figma_type: FRAME
      figma_name: Quick Filter Row / Khu vực
      style:
        display: flex
        flex-direction: row
        gap: 8px
        align-items: center
        width: 100%
        height: auto
        overflow-x: auto
        scrollbar-width: none
      children:
      - id: label
        node_id: 1748:16296
        tag: p
        figma_type: TEXT
        figma_name: Label
        style:
          width: fit-content
          flex-shrink: 0
          height: auto
          color: '#595959'
          white-space: nowrap
          font-family: '''Reddit Sans'''
          font-size: 12px
          font-weight: 500
          line-height: 18px
          align-self: center
        text: 'Khu vực:'
        token_hint:
          color: Text/text-secondary
          line-height: Tagline/Annotation/tagline-annotation-line-height
          letter-spacing: Tagline/Annotation/tagline-annotation-letter-spacing
          font-weight: Tagline/Annotation/tagline-annotation-font-weight
          font-family: Font family/font-family
          font-size: Tagline/Annotation/tagline-annotation-font-size
          typography: Tagline/tagline-annotation
      - id: chip
        node_id: 1748:16297
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 4px
          justify-content: center
          align-items: center
          padding: 6px 12px 6px 12px
          width: 124.0px
          flex-shrink: 0
          height: 32.0px
          box-shadow: 'inset 0 0 0 1px #dddddd'
          border-radius: 999px
        ds_component:
          props:
            text: true
            right_icon: false
            change_l_icon: 3:4249
            left_icon: false
            size: Medium 32px
            style: Outline
            select: 'No'
            state: Default
          figma_component: Chip
        content:
          labels:
          - node_id: I1748:16297;44:8417
            text: Tp. Hồ Chí Minh
            box:
            - 12
            - 6
            - 100
            - 20
            style:
              font-family: '''Reddit Sans'''
              font-size: 14px
              font-weight: 600
              line-height: 20px
              color: '#595959'
            token_hint:
              typography: Header/header-caption
              color: Text/text-secondary
        token_hint:
          gap: Spacing/gap-2x-small-4
          border-width: Stroke/stroke-divider
          border-radius: Radius/radius-pill
          border-color: Border/border-regular
        behavior: BH-EXISTING
      - id: chip_2
        node_id: 1748:16298
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 4px
          justify-content: center
          align-items: center
          padding: 6px 12px 6px 12px
          width: 68.0px
          flex-shrink: 0
          height: 32.0px
          box-shadow: 'inset 0 0 0 1px #dddddd'
          border-radius: 999px
        ds_component:
          props:
            text: true
            right_icon: false
            change_l_icon: 3:4249
            left_icon: false
            size: Medium 32px
            style: Outline
            select: 'No'
            state: Default
          figma_component: Chip
        content:
          labels:
          - node_id: I1748:16298;44:8417
            text: Hà Nội
            box:
            - 12
            - 6
            - 44
            - 20
            style:
              font-family: '''Reddit Sans'''
              font-size: 14px
              font-weight: 600
              line-height: 20px
              color: '#595959'
            token_hint:
              typography: Header/header-caption
              color: Text/text-secondary
        token_hint:
          gap: Spacing/gap-2x-small-4
          border-width: Stroke/stroke-divider
          border-radius: Radius/radius-pill
          border-color: Border/border-regular
        behavior: BH-EXISTING
      - id: chip_3
        node_id: 1748:16299
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 4px
          justify-content: center
          align-items: center
          padding: 6px 12px 6px 12px
          width: 80.0px
          flex-shrink: 0
          height: 32.0px
          box-shadow: 'inset 0 0 0 1px #dddddd'
          border-radius: 999px
        ds_component:
          props:
            text: true
            right_icon: false
            change_l_icon: 3:4249
            left_icon: false
            size: Medium 32px
            style: Outline
            select: 'No'
            state: Default
          figma_component: Chip
        content:
          labels:
          - node_id: I1748:16299;44:8417
            text: Đà Nẵng
            box:
            - 12
            - 6
            - 56
            - 20
            style:
              font-family: '''Reddit Sans'''
              font-size: 14px
              font-weight: 600
              line-height: 20px
              color: '#595959'
            token_hint:
              typography: Header/header-caption
              color: Text/text-secondary
        token_hint:
          gap: Spacing/gap-2x-small-4
          border-width: Stroke/stroke-divider
          border-radius: Radius/radius-pill
          border-color: Border/border-regular
        behavior: BH-EXISTING
      - id: chip_4
        node_id: 1748:16300
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 4px
          justify-content: center
          align-items: center
          padding: 6px 12px 6px 12px
          width: 76.0px
          flex-shrink: 0
          height: 32.0px
          box-shadow: 'inset 0 0 0 1px #dddddd'
          border-radius: 999px
        ds_component:
          props:
            change_l_icon: 3:1312
            text: true
            right_icon: false
            left_icon: false
            size: Medium 32px
            style: Outline
            select: 'No'
            state: Default
          figma_component: Chip
        content:
          labels:
          - node_id: I1748:16300;44:8417
            text: Cần Thơ
            box:
            - 12
            - 6
            - 52
            - 20
            style:
              font-family: '''Reddit Sans'''
              font-size: 14px
              font-weight: 600
              line-height: 20px
              color: '#595959'
            token_hint:
              typography: Header/header-caption
              color: Text/text-secondary
        token_hint:
          gap: Spacing/gap-2x-small-4
          border-width: Stroke/stroke-divider
          border-radius: Radius/radius-pill
          border-color: Border/border-regular
        behavior: BH-EXISTING
      - id: chip_5
        node_id: 1748:16301
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 4px
          justify-content: center
          align-items: center
          padding: 6px 12px 6px 12px
          width: 95.0px
          flex-shrink: 0
          height: 32.0px
          box-shadow: 'inset 0 0 0 1px #dddddd'
          border-radius: 999px
        ds_component:
          props:
            text: true
            right_icon: false
            change_l_icon: 66:65444
            left_icon: true
            size: Medium 32px
            style: Outline
            select: 'No'
            state: Default
          figma_component: Chip
        content:
          labels:
          - node_id: I1748:16301;44:8417
            text: Gần tôi
            box:
            - 36
            - 6
            - 47
            - 20
            style:
              font-family: '''Reddit Sans'''
              font-size: 14px
              font-weight: 600
              line-height: 20px
              color: '#595959'
            token_hint:
              typography: Header/header-caption
              color: Text/text-secondary
          glyphs:
          - node_id: I1748:16301;44:8416
            box:
            - 12
            - 6
            - 20
            - 20
            icon: CurrentLocation-outline
        token_hint:
          gap: Spacing/gap-2x-small-4
          border-width: Stroke/stroke-divider
          border-radius: Radius/radius-pill
          border-color: Border/border-regular
        behavior: BH-EXISTING
```

## 2.4 Overlay `ovl_apartment_type`

### web_desktop — `ovl_apartment_type_popover_desktop` (Figma `1732:5361`)

```yaml
id: ovl_apartment_type_popover_desktop
spec_id: ovl_apartment_type
kind: overlay
type: popover
platform: web_desktop
figma_node: 1732:5361
hosts:
- PTY.adlisting_apartment
triggered_by: BH-FILTER-OPEN
scrim: none
dismiss:
- close_outline
- outside click
- back
requirement:
  traces_to:
  - FR-1.1
  - FR-1.2
  - FR-1.3
  - FR-1.5
  context: 'Value list of param apartment_type ("Loại hình căn hộ"). Values come from server config, in server order. Current
    list and ORDER per Figma (designer-updated 2026-09-24, matches PRD FR-1 order): 1 Chung cư thương mại, 2 Nhà ở xã hội,
    then existing: Duplex, Penthouse, Căn hộ dịch vụ mini, Tập thể cư xá, Officetel. "Chung cư" must not appear anywhere.'
  acceptance:
  - AC-FR1-1 list shows Nhà ở xã hội and Chung cư thương mại (no "Chung cư")
  - AC-FR1-2 select NOXH → only NOXH ads
  - AC-FR1-3 select CCTM → only CCTM ads
  - Value with 0 ads hidden (existing inventory rule)
uses:
  behaviors:
  - BH-EXISTING
  - BH-FILTER-SELECT
  - BH-OVERLAY-CLOSE
variants:
  default:
    when: nothing selected
    tree: = layout
layout:
  id: ovl_apartment_type_popover_desktop
  node_id: 1732:5361
  tag: div
  figma_type: FRAME
  figma_name: '[OVL] ovl_apartment_type_popover — Desktop'
  style:
    display: flex
    flex-direction: column
    align-items: center
    width: 100%
    flex-shrink: 0
    height: 497px
    overflow: hidden
    background: '#ffffff'
    border-radius: 20px
    box-shadow: 0px 4px 16px 0px rgba(34, 34, 34, 0.12)
    position: relative
  children:
  - id: drawer_header_dt
    node_id: 1732:5362
    tag: div
    figma_type: FRAME
    figma_name: drawer_header_dt
    style:
      display: flex
      flex-direction: row
      gap: 8px
      justify-content: center
      align-items: center
      padding: 12px 20px 12px 20px
      width: 100%
      height: 48px
      background: '#ffffff'
      border-bottom: '1px solid #e8e8e8'
    children:
    - id: close_outline
      node_id: 1732:5364
      tag: div
      figma_type: INSTANCE
      figma_name: Close-outline
      style:
        width: 24px
        flex-shrink: 0
        height: 24px
        position: relative
      ds_component:
        figma_component: Close-outline
      icon: Close-outline
      asset_mode: none — see icon/data
      behavior: BH-OVERLAY-CLOSE
    - id: header_title_dt
      node_id: 1732:5365
      tag: p
      figma_type: TEXT
      figma_name: header_title_dt
      style:
        flex: '1'
        width: 100%
        height: auto
        color: '#222222'
        white-space: nowrap
        font-family: '''Reddit Sans'''
        font-size: 16px
        font-weight: 600
        line-height: 24px
        text-align: center
        align-self: center
      text: Loại hình căn hộ
      token_hint:
        color: Text/text-primary
        line-height: Header/Section/header-section-line-height
        letter-spacing: Header/Section/header-section-letter-spacing
        font-weight: Header/Section/header-section-font-weight
        font-family: Header/Section/header-section-font-family
        font-size: Header/Section/header-section-font-size
        typography: Header/header-section
    - id: drawer_left_button
      node_id: 1732:5366
      tag: div
      figma_type: FRAME
      figma_name: Drawer Left Button
      style:
        width: 24px
        flex-shrink: 0
        height: 24px
    token_hint:
      gap: Spacing/gap-x-small-8
      padding-left: Padding/padding-large-20
      padding-top: Padding/padding-small-12
      padding-right: Padding/padding-large-20
      padding-bottom: Padding/padding-small-12
      border-width: Stroke/stroke-divider
      background: Background/Light/background-primary
      border-color: Border/border-thin
  - id: body_content_dt
    node_id: 1732:5367
    tag: div
    figma_type: FRAME
    figma_name: body_content_dt
    style:
      display: flex
      flex-direction: column
      gap: 16px
      align-items: flex-start
      padding: 16px 0px 16px 0px
      flex: '1'
      width: 100%
      height: 100%
    children:
    - id: apartment_type_body_dt
      node_id: 1732:5368
      tag: div
      figma_type: FRAME
      figma_name: apartment_type_body_dt
      style:
        display: flex
        flex-direction: column
        gap: 16px
        align-items: flex-start
        width: 100%
        height: auto
      children:
      - id: apartment_type_search_slot_dt
        node_id: 1732:5369
        tag: div
        figma_type: FRAME
        figma_name: apartment_type_search_slot_dt
        style:
          display: flex
          flex-direction: column
          gap: 4px
          align-items: flex-start
          padding: 0px 16px 0px 16px
          width: 100%
          height: auto
        children:
        - id: search_bar
          node_id: 1732:5370
          tag: div
          figma_type: INSTANCE
          figma_name: Search Bar
          style:
            display: flex
            flex-direction: row
            gap: 8px
            align-items: center
            padding: 4px 4px 4px 12px
            width: 100%
            height: 40.0px
            overflow: hidden
            background: '#f4f4f4'
            border-radius: 99px
          ds_component:
            props:
              mua_b_n: false
              state: Active
            figma_component: Search Bar
          content:
            labels:
            - node_id: I1732:5370;12618:41211
              text: Tìm kiếm loại hình căn hộ
              box:
              - 40
              - 10
              - 299
              - 20
              style:
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 600
                line-height: 20px
                color: '#595959'
              token_hint:
                typography: Header/header-caption
                color: Text/text-secondary
            glyphs:
            - node_id: I1732:5370;12618:41210
              box:
              - 12
              - 10
              - 20
              - 20
              icon: Search-outline
          measured_width_px: 343.0
          token_hint:
            background: Background/Light/background-secondary
          behavior: BH-EXISTING
      - id: apartment_type_options_dt
        node_id: 1732:5371
        tag: div
        figma_type: FRAME
        figma_name: apartment_type_options_dt
        style:
          display: flex
          flex-direction: column
          align-items: flex-start
          width: 100%
          height: auto
        children:
        - id: list
          node_id: 1732:5373
          tag: div
          figma_type: INSTANCE
          figma_name: List
          style:
            display: flex
            flex-direction: column
            gap: 8px
            justify-content: center
            align-items: flex-start
            padding: 12px 16px 12px 16px
            width: 100%
            height: 48.0px
            background: '#ffffff'
            border-bottom: '1px solid #f4f4f4'
          ds_component:
            props:
              change_icon: 1732:5047
              add_description: false
              add_icon: false
              type: ✅ Checkbox
              size: Large - 48px
              state: Default
              left_right: 'Yes'
            figma_component: List
          content:
            labels:
            - node_id: I1732:5373;1545:3451
              text: Chung cư thương mại
              box:
              - 16
              - 12
              - 311
              - 24
              style:
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 400
                line-height: 20px
                color: '#222222'
              token_hint:
                typography: Body/body-section
                color: Text/text-primary
            glyphs:
            - node_id: I1732:5373;1545:3452
              box:
              - 335
              - 12
              - 24
              - 24
              icon: State=Default, Bulk Selection=No, Unselect / Select=No
            surfaces:
            - node_id: I1732:5373;1545:3452;558:6231
              box:
              - 337
              - 14
              - 20
              - 20
              style:
                background: '#ffffff'
                border: '2px solid #dddddd'
                border-radius: 4px
              token_hint:
                background: Background/Light/background-primary
                border-color: Border/border-regular
          measured_width_px: 375.0
          token_hint:
            gap: Spacing/gap-x-small-8
            padding-left: Padding/padding-medium-16
            padding-top: Padding/padding-small-12
            padding-right: Padding/padding-medium-16
            padding-bottom: Padding/padding-small-12
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
            border-color: Border/border-divider
          behavior: BH-FILTER-SELECT
          data:
            source: server filter config for param apartment_type, in server order
            note: 'Order: Chung cư thương mại, Nhà ở xã hội, Duplex, Penthouse, Căn hộ dịch vụ mini, Tập thể cư xá, Officetel
              (Figma, designer-updated 2026-09-24)'
        - id: list_2
          node_id: 1732:5372
          tag: div
          figma_type: INSTANCE
          figma_name: List
          style:
            display: flex
            flex-direction: column
            gap: 8px
            justify-content: center
            align-items: flex-start
            padding: 12px 16px 12px 16px
            width: 100%
            height: 48.0px
            background: '#ffffff'
            border-bottom: '1px solid #f4f4f4'
          ds_component:
            props:
              change_icon: 1732:5047
              add_description: false
              add_icon: false
              type: ✅ Checkbox
              size: Large - 48px
              state: Default
              left_right: 'Yes'
            figma_component: List
          content:
            labels:
            - node_id: I1732:5372;1545:3451
              text: Nhà ở xã hội
              box:
              - 16
              - 12
              - 311
              - 24
              style:
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 400
                line-height: 20px
                color: '#222222'
              token_hint:
                typography: Body/body-section
                color: Text/text-primary
            glyphs:
            - node_id: I1732:5372;1545:3452
              box:
              - 335
              - 12
              - 24
              - 24
              icon: State=Default, Bulk Selection=No, Unselect / Select=No
            surfaces:
            - node_id: I1732:5372;1545:3452;558:6231
              box:
              - 337
              - 14
              - 20
              - 20
              style:
                background: '#ffffff'
                border: '2px solid #dddddd'
                border-radius: 4px
              token_hint:
                background: Background/Light/background-primary
                border-color: Border/border-regular
          measured_width_px: 375.0
          token_hint:
            gap: Spacing/gap-x-small-8
            padding-left: Padding/padding-medium-16
            padding-top: Padding/padding-small-12
            padding-right: Padding/padding-medium-16
            padding-bottom: Padding/padding-small-12
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
            border-color: Border/border-divider
          behavior: BH-FILTER-SELECT
          data:
            source: server filter config for param apartment_type, in server order
            note: 'Order: Chung cư thương mại, Nhà ở xã hội, Duplex, Penthouse, Căn hộ dịch vụ mini, Tập thể cư xá, Officetel
              (Figma, designer-updated 2026-09-24)'
        - id: list_3
          node_id: 1732:5374
          tag: div
          figma_type: INSTANCE
          figma_name: List
          style:
            display: flex
            flex-direction: column
            gap: 8px
            justify-content: center
            align-items: flex-start
            padding: 12px 16px 12px 16px
            width: 100%
            height: 48.0px
            background: '#ffffff'
            border-bottom: '1px solid #f4f4f4'
          ds_component:
            props:
              change_icon: 1732:5047
              add_description: false
              add_icon: false
              type: ✅ Checkbox
              size: Large - 48px
              state: Default
              left_right: 'Yes'
            figma_component: List
          content:
            labels:
            - node_id: I1732:5374;1545:3451
              text: Duplex
              box:
              - 16
              - 12
              - 311
              - 24
              style:
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 400
                line-height: 20px
                color: '#222222'
              token_hint:
                typography: Body/body-section
                color: Text/text-primary
            glyphs:
            - node_id: I1732:5374;1545:3452
              box:
              - 335
              - 12
              - 24
              - 24
              icon: State=Default, Bulk Selection=No, Unselect / Select=No
            surfaces:
            - node_id: I1732:5374;1545:3452;558:6231
              box:
              - 337
              - 14
              - 20
              - 20
              style:
                background: '#ffffff'
                border: '2px solid #dddddd'
                border-radius: 4px
              token_hint:
                background: Background/Light/background-primary
                border-color: Border/border-regular
          measured_width_px: 375.0
          token_hint:
            gap: Spacing/gap-x-small-8
            padding-left: Padding/padding-medium-16
            padding-top: Padding/padding-small-12
            padding-right: Padding/padding-medium-16
            padding-bottom: Padding/padding-small-12
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
            border-color: Border/border-divider
          behavior: BH-FILTER-SELECT
          data:
            source: server filter config for param apartment_type, in server order
            note: 'Order: Chung cư thương mại, Nhà ở xã hội, Duplex, Penthouse, Căn hộ dịch vụ mini, Tập thể cư xá, Officetel
              (Figma, designer-updated 2026-09-24)'
        - id: list_4
          node_id: 1732:5375
          tag: div
          figma_type: INSTANCE
          figma_name: List
          style:
            display: flex
            flex-direction: column
            gap: 8px
            justify-content: center
            align-items: flex-start
            padding: 12px 16px 12px 16px
            width: 100%
            height: 48.0px
            background: '#ffffff'
            border-bottom: '1px solid #f4f4f4'
          ds_component:
            props:
              change_icon: 1732:5047
              add_description: false
              add_icon: false
              type: ✅ Checkbox
              size: Large - 48px
              state: Default
              left_right: 'Yes'
            figma_component: List
          content:
            labels:
            - node_id: I1732:5375;1545:3451
              text: Penthouse
              box:
              - 16
              - 12
              - 311
              - 24
              style:
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 400
                line-height: 20px
                color: '#222222'
              token_hint:
                typography: Body/body-section
                color: Text/text-primary
            glyphs:
            - node_id: I1732:5375;1545:3452
              box:
              - 335
              - 12
              - 24
              - 24
              icon: State=Default, Bulk Selection=No, Unselect / Select=No
            surfaces:
            - node_id: I1732:5375;1545:3452;558:6231
              box:
              - 337
              - 14
              - 20
              - 20
              style:
                background: '#ffffff'
                border: '2px solid #dddddd'
                border-radius: 4px
              token_hint:
                background: Background/Light/background-primary
                border-color: Border/border-regular
          measured_width_px: 375.0
          token_hint:
            gap: Spacing/gap-x-small-8
            padding-left: Padding/padding-medium-16
            padding-top: Padding/padding-small-12
            padding-right: Padding/padding-medium-16
            padding-bottom: Padding/padding-small-12
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
            border-color: Border/border-divider
          behavior: BH-FILTER-SELECT
          data:
            source: server filter config for param apartment_type, in server order
            note: 'Order: Chung cư thương mại, Nhà ở xã hội, Duplex, Penthouse, Căn hộ dịch vụ mini, Tập thể cư xá, Officetel
              (Figma, designer-updated 2026-09-24)'
        - id: list_5
          node_id: 1732:5376
          tag: div
          figma_type: INSTANCE
          figma_name: List
          style:
            display: flex
            flex-direction: column
            gap: 8px
            justify-content: center
            align-items: flex-start
            padding: 12px 16px 12px 16px
            width: 100%
            height: 48.0px
            background: '#ffffff'
            border-bottom: '1px solid #f4f4f4'
          ds_component:
            props:
              change_icon: 1732:5047
              add_description: false
              add_icon: false
              type: ✅ Checkbox
              size: Large - 48px
              state: Default
              left_right: 'Yes'
            figma_component: List
          content:
            labels:
            - node_id: I1732:5376;1545:3451
              text: Căn hộ dịch vụ, mini
              box:
              - 16
              - 12
              - 311
              - 24
              style:
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 400
                line-height: 20px
                color: '#222222'
              token_hint:
                typography: Body/body-section
                color: Text/text-primary
            glyphs:
            - node_id: I1732:5376;1545:3452
              box:
              - 335
              - 12
              - 24
              - 24
              icon: State=Default, Bulk Selection=No, Unselect / Select=No
            surfaces:
            - node_id: I1732:5376;1545:3452;558:6231
              box:
              - 337
              - 14
              - 20
              - 20
              style:
                background: '#ffffff'
                border: '2px solid #dddddd'
                border-radius: 4px
              token_hint:
                background: Background/Light/background-primary
                border-color: Border/border-regular
          measured_width_px: 375.0
          token_hint:
            gap: Spacing/gap-x-small-8
            padding-left: Padding/padding-medium-16
            padding-top: Padding/padding-small-12
            padding-right: Padding/padding-medium-16
            padding-bottom: Padding/padding-small-12
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
            border-color: Border/border-divider
          behavior: BH-FILTER-SELECT
          data:
            source: server filter config for param apartment_type, in server order
            note: 'Order: Chung cư thương mại, Nhà ở xã hội, Duplex, Penthouse, Căn hộ dịch vụ mini, Tập thể cư xá, Officetel
              (Figma, designer-updated 2026-09-24)'
        - id: list_6
          node_id: 1732:5377
          tag: div
          figma_type: INSTANCE
          figma_name: List
          style:
            display: flex
            flex-direction: column
            gap: 8px
            justify-content: center
            align-items: flex-start
            padding: 12px 16px 12px 16px
            width: 100%
            height: 48.0px
            background: '#ffffff'
            border-bottom: '1px solid #f4f4f4'
          ds_component:
            props:
              change_icon: 1732:5047
              add_description: false
              add_icon: false
              type: ✅ Checkbox
              size: Large - 48px
              state: Default
              left_right: 'Yes'
            figma_component: List
          content:
            labels:
            - node_id: I1732:5377;1545:3451
              text: Tập thể, cư xá
              box:
              - 16
              - 12
              - 311
              - 24
              style:
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 400
                line-height: 20px
                color: '#222222'
              token_hint:
                typography: Body/body-section
                color: Text/text-primary
            glyphs:
            - node_id: I1732:5377;1545:3452
              box:
              - 335
              - 12
              - 24
              - 24
              icon: State=Default, Bulk Selection=No, Unselect / Select=No
            surfaces:
            - node_id: I1732:5377;1545:3452;558:6231
              box:
              - 337
              - 14
              - 20
              - 20
              style:
                background: '#ffffff'
                border: '2px solid #dddddd'
                border-radius: 4px
              token_hint:
                background: Background/Light/background-primary
                border-color: Border/border-regular
          measured_width_px: 375.0
          token_hint:
            gap: Spacing/gap-x-small-8
            padding-left: Padding/padding-medium-16
            padding-top: Padding/padding-small-12
            padding-right: Padding/padding-medium-16
            padding-bottom: Padding/padding-small-12
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
            border-color: Border/border-divider
          behavior: BH-FILTER-SELECT
          data:
            source: server filter config for param apartment_type, in server order
            note: 'Order: Chung cư thương mại, Nhà ở xã hội, Duplex, Penthouse, Căn hộ dịch vụ mini, Tập thể cư xá, Officetel
              (Figma, designer-updated 2026-09-24)'
        - id: list_7
          node_id: 1732:5378
          tag: div
          figma_type: INSTANCE
          figma_name: List
          style:
            display: flex
            flex-direction: column
            gap: 8px
            justify-content: center
            align-items: flex-start
            padding: 12px 16px 12px 16px
            width: 100%
            height: 48.0px
            background: '#ffffff'
            border-bottom: '1px solid #f4f4f4'
          ds_component:
            props:
              change_icon: 1732:5047
              add_description: false
              add_icon: false
              type: ✅ Checkbox
              size: Large - 48px
              state: Default
              left_right: 'Yes'
            figma_component: List
          content:
            labels:
            - node_id: I1732:5378;1545:3451
              text: Officetel
              box:
              - 16
              - 12
              - 311
              - 24
              style:
                font-family: '''Reddit Sans'''
                font-size: 14px
                font-weight: 400
                line-height: 20px
                color: '#222222'
              token_hint:
                typography: Body/body-section
                color: Text/text-primary
            glyphs:
            - node_id: I1732:5378;1545:3452
              box:
              - 335
              - 12
              - 24
              - 24
              icon: State=Default, Bulk Selection=No, Unselect / Select=No
            surfaces:
            - node_id: I1732:5378;1545:3452;558:6231
              box:
              - 337
              - 14
              - 20
              - 20
              style:
                background: '#ffffff'
                border: '2px solid #dddddd'
                border-radius: 4px
              token_hint:
                background: Background/Light/background-primary
                border-color: Border/border-regular
          measured_width_px: 375.0
          token_hint:
            gap: Spacing/gap-x-small-8
            padding-left: Padding/padding-medium-16
            padding-top: Padding/padding-small-12
            padding-right: Padding/padding-medium-16
            padding-bottom: Padding/padding-small-12
            border-width: Stroke/stroke-divider
            background: Background/Light/background-primary
            border-color: Border/border-divider
          behavior: BH-FILTER-SELECT
          data:
            source: server filter config for param apartment_type, in server order
            note: 'Order: Chung cư thương mại, Nhà ở xã hội, Duplex, Penthouse, Căn hộ dịch vụ mini, Tập thể cư xá, Officetel
              (Figma, designer-updated 2026-09-24)'
    token_hint:
      gap: Spacing/gap-medium-16
      padding-top: Padding/padding-medium-16
      padding-bottom: Padding/padding-medium-16
  - id: action_bar_dt
    node_id: 1732:5379
    tag: div
    figma_type: FRAME
    figma_name: action_bar_dt
    style:
      display: flex
      flex-direction: row
      gap: 8px
      justify-content: center
      align-items: center
      padding: 12px
      width: 100%
      height: auto
      background: '#ffffff'
      border-top: '1px solid #e8e8e8'
    children:
    - id: button
      node_id: 1732:5380
      tag: div
      figma_type: INSTANCE
      figma_name: Button
      style:
        display: flex
        flex-direction: row
        gap: 4px
        justify-content: center
        align-items: center
        padding: 4px 12px 4px 12px
        flex: '1'
        width: 171.5px
        height: 32.0px
        min-height: 32px
        max-height: 32px
        background: '#ffffff'
        box-shadow: 'inset 0 0 0 1px #dddddd'
        border-radius: 8px
        flex-shrink: 0
      ds_component:
        props:
          icon_left: 843:175310
          input_text: Xoá lọc
          size: M - 32px
          type: 🥉  Tertiary
          state: Active
          ispill: 'No'
        figma_component: Button
      content:
        labels:
        - node_id: I1732:5380;42:31643
          text: Xoá lọc
          box:
          - 61.75
          - 6
          - 48
          - 20
          style:
            font-family: '''Reddit Sans'''
            font-size: 14px
            font-weight: 700
            line-height: 20px
            color: '#222222'
          token_hint:
            typography: Label/label - section
            color: Text/text-primary
      token_hint:
        gap: Spacing/gap-2x-small-4
        padding-left: Padding/padding-small-12
        padding-top: Padding/padding-2x-small-4
        padding-right: Padding/padding-small-12
        padding-bottom: Padding/padding-2x-small-4
        border-width: Stroke/stroke-divider
        border-radius: Radius/radius-card-small
        background: Button/Solid/button-blank
        border-color: Border/border-regular
      behavior: BH-FILTER-SELECT
    - id: button_2
      node_id: 1732:5381
      tag: div
      figma_type: INSTANCE
      figma_name: Button
      style:
        display: flex
        flex-direction: row
        gap: 4px
        justify-content: center
        align-items: center
        padding: 4px 12px 4px 12px
        flex: '1'
        width: 171.5px
        height: 32.0px
        min-height: 32px
        max-height: 32px
        background: '#c0c0c0'
        border-radius: 8px
        flex-shrink: 0
      ds_component:
        props:
          icon_left: false
          input_text: Áp dụng
          size: M - 32px
          type: 🥇  Primary
          state: Disabled
          ispill: 'No'
        figma_component: Button
      content:
        labels:
        - node_id: I1732:5381;42:31539
          text: Áp dụng
          box:
          - 57.75
          - 6
          - 56
          - 20
          style:
            font-family: '''Reddit Sans'''
            font-size: 14px
            font-weight: 700
            line-height: 20px
            color: '#ffffff'
          token_hint:
            typography: Label/label - section
            color: Text/text-blank
      token_hint:
        gap: Spacing/gap-2x-small-4
        padding-left: Padding/padding-small-12
        padding-top: Padding/padding-2x-small-4
        padding-right: Padding/padding-small-12
        padding-bottom: Padding/padding-2x-small-4
        border-width: Stroke/stroke-action
        border-radius: Radius/radius-card-small
        background: Button/Solid/button-disabled
      behavior: BH-FILTER-SELECT
    token_hint:
      gap: Spacing/gap-x-small-8
      border-width: Stroke/stroke-divider
      background: Background/Light/background-primary
      border-color: Border/border-thin
  measured_width_px: 375.0
  token_hint:
    background: Background/Light/background-primary
    box-shadow: shadow-floating
anchor:
  node: filter chip "Loại hình căn hộ"
  placement: bottom-start
  offset: measure from Figma frame
```

### mobile — msite; iOS/Android reuse — `ovl_apartment_type_drawer_mobile` (Figma `1732:9372`)

```yaml
id: ovl_apartment_type_drawer_mobile
spec_id: ovl_apartment_type
kind: overlay
type: bottom_sheet
platform: mobile — msite; iOS/Android reuse
figma_node: 1732:9372
hosts:
- PTY.adlisting_apartment
triggered_by: BH-FILTER-OPEN
scrim: dim
dismiss:
- close_outline
- scrim tap
- back
requirement:
  traces_to:
  - FR-1.1
  - FR-1.2
  - FR-1.3
  - FR-1.5
  context: 'Value list of param apartment_type ("Loại hình căn hộ"). Values come from server config, in server order. Current
    list and ORDER per Figma (designer-updated 2026-09-24, matches PRD FR-1 order): 1 Chung cư thương mại, 2 Nhà ở xã hội,
    then existing: Duplex, Penthouse, Căn hộ dịch vụ mini, Tập thể cư xá, Officetel. "Chung cư" must not appear anywhere.'
  acceptance:
  - AC-FR1-1 list shows Nhà ở xã hội and Chung cư thương mại (no "Chung cư")
  - AC-FR1-2 select NOXH → only NOXH ads
  - AC-FR1-3 select CCTM → only CCTM ads
  - Value with 0 ads hidden (existing inventory rule)
uses:
  behaviors:
  - BH-FILTER-SELECT
  - BH-OVERLAY-CLOSE
variants:
  default:
    when: nothing selected
    tree: = layout
layout:
  id: ovl_apartment_type_drawer_mobile
  node_id: 1732:9372
  tag: div
  figma_type: FRAME
  figma_name: '[OVL] ovl_apartment_type_drawer — Mobile'
  style:
    display: flex
    flex-direction: column
    align-items: center
    width: 100%
    flex-shrink: 0
    height: auto
    overflow: hidden
    background: '#ffffff'
    border-radius: 20px 20px 0px 0px
    box-shadow: 0px 4px 16px 0px rgba(34, 34, 34, 0.12)
    position: relative
  children:
  - id: drawer_header
    node_id: 1732:9373
    tag: div
    figma_type: FRAME
    figma_name: Drawer Header
    style:
      display: flex
      flex-direction: row
      gap: 8px
      justify-content: center
      align-items: center
      padding: 12px 20px 12px 20px
      width: 100%
      height: 48px
      background: '#ffffff'
      border-bottom: '1px solid #e8e8e8'
    children:
    - id: close_outline
      node_id: 1732:9375
      tag: div
      figma_type: INSTANCE
      figma_name: Close-outline
      style:
        width: 24px
        flex-shrink: 0
        height: 24px
        position: relative
      ds_component:
        figma_component: Close-outline
      icon: Close-outline
      asset_mode: none — see icon/data
      behavior: BH-OVERLAY-CLOSE
    - id: header_title_mw
      node_id: 1732:9376
      tag: p
      figma_type: TEXT
      figma_name: header_title_mw
      style:
        flex: '1'
        width: 100%
        height: auto
        color: '#222222'
        white-space: nowrap
        font-family: '''Reddit Sans'''
        font-size: 16px
        font-weight: 600
        line-height: 24px
        text-align: center
        align-self: center
      text: Loại hình căn hộ
      token_hint:
        color: Text/text-primary
        line-height: Header/Section/header-section-line-height
        letter-spacing: Header/Section/header-section-letter-spacing
        font-weight: Header/Section/header-section-font-weight
        font-family: Header/Section/header-section-font-family
        font-size: Header/Section/header-section-font-size
        typography: Header/header-section
    - id: drawer_left_button
      node_id: 1732:9377
      tag: div
      figma_type: FRAME
      figma_name: Drawer Left Button
      style:
        width: 24px
        flex-shrink: 0
        height: 24px
    token_hint:
      gap: Spacing/gap-x-small-8
      padding-left: Padding/padding-large-20
      padding-top: Padding/padding-small-12
      padding-right: Padding/padding-large-20
      padding-bottom: Padding/padding-small-12
      border-width: Stroke/stroke-divider
      background: Background/Light/background-primary
      border-color: Border/border-thin
  - id: body_content_mw
    node_id: 1732:9378
    tag: div
    figma_type: FRAME
    figma_name: body_content_mw
    style:
      display: flex
      flex-direction: column
      gap: 8px
      align-items: flex-start
      padding: 16px 20px 16px 20px
      width: 100%
      height: auto
    children:
    - id: apartment_type_title_mw
      node_id: 1732:9834
      tag: p
      figma_type: TEXT
      figma_name: apartment_type_title_mw
      style:
        width: 100%
        height: auto
        color: '#222222'
        white-space: nowrap
        font-family: '''Reddit Sans'''
        font-size: 14px
        font-weight: 600
        line-height: 20px
      text: Loại hình căn hộ
      token_hint:
        color: Text/text-primary
        line-height: Header/Caption/header-caption-line-height
        letter-spacing: Header/Caption/header-caption-letter-spacing
        font-weight: Header/Caption/header-caption-font-weight
        font-family: Header/Caption/header-caption-font-family
        font-size: Header/Caption/header-caption-font-size
        typography: Header/header-caption
    - id: apartment_type_options_mw
      node_id: 1732:9856
      tag: div
      figma_type: FRAME
      figma_name: apartment_type_options_mw
      style:
        display: flex
        flex-direction: row
        flex-wrap: wrap
        gap: 4px
        align-items: flex-start
        width: 100%
        height: auto
      children:
      - id: chip
        node_id: 1732:9851
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 3px
          justify-content: center
          align-items: center
          padding: 3px 10px 3px 10px
          width: 137.0px
          flex-shrink: 0
          height: 28.0px
          background: '#f4f4f4'
          border-radius: 999px
        ds_component:
          props:
            text: true
            change_l_icon: 875:185710
            right_icon: false
            left_icon: false
            size: Small 24px
            style: Fill
            select: 'No'
            state: Default
          figma_component: Chip
        content:
          labels:
          - node_id: I1732:9851;12236:833
            text: Chung cư thương mại
            box:
            - 10
            - 5
            - 117
            - 18
            style:
              font-family: '''Reddit Sans'''
              font-size: 12px
              font-weight: 600
              line-height: 18px
              color: '#222222'
            token_hint:
              typography: Header/header-annotation
              color: Text/text-primary
        token_hint:
          border-width: Stroke/stroke-action
          border-radius: Radius/radius-pill
          background: Background/Light/background-secondary
        behavior: BH-FILTER-SELECT
        data:
          source: server filter config for param apartment_type, in server order
          note: 'Order: Chung cư thương mại, Nhà ở xã hội, Duplex, Penthouse, Căn hộ dịch vụ mini, Tập thể cư xá, Officetel
            (Figma, designer-updated 2026-09-24)'
      - id: chip_2
        node_id: 1732:9844
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 3px
          justify-content: center
          align-items: center
          padding: 3px 10px 3px 10px
          width: 88.0px
          flex-shrink: 0
          height: 28.0px
          background: '#f4f4f4'
          border-radius: 999px
        ds_component:
          props:
            text: true
            change_l_icon: 875:185710
            right_icon: false
            left_icon: false
            size: Small 24px
            style: Fill
            select: 'No'
            state: Default
          figma_component: Chip
        content:
          labels:
          - node_id: I1732:9844;12236:833
            text: Nhà ở xã hội
            box:
            - 10
            - 5
            - 68
            - 18
            style:
              font-family: '''Reddit Sans'''
              font-size: 12px
              font-weight: 600
              line-height: 18px
              color: '#222222'
            token_hint:
              typography: Header/header-annotation
              color: Text/text-primary
        token_hint:
          border-width: Stroke/stroke-action
          border-radius: Radius/radius-pill
          background: Background/Light/background-secondary
        behavior: BH-FILTER-SELECT
        data:
          source: server filter config for param apartment_type, in server order
          note: 'Order: Chung cư thương mại, Nhà ở xã hội, Duplex, Penthouse, Căn hộ dịch vụ mini, Tập thể cư xá, Officetel
            (Figma, designer-updated 2026-09-24)'
      - id: chip_3
        node_id: 1732:9857
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 3px
          justify-content: center
          align-items: center
          padding: 3px 10px 3px 10px
          width: 59.0px
          flex-shrink: 0
          height: 28.0px
          background: '#f4f4f4'
          border-radius: 999px
        ds_component:
          props:
            text: true
            change_l_icon: 875:185710
            right_icon: false
            left_icon: false
            size: Small 24px
            style: Fill
            select: 'No'
            state: Default
          figma_component: Chip
        content:
          labels:
          - node_id: I1732:9857;12236:833
            text: Duplex
            box:
            - 10
            - 5
            - 39
            - 18
            style:
              font-family: '''Reddit Sans'''
              font-size: 12px
              font-weight: 600
              line-height: 18px
              color: '#222222'
            token_hint:
              typography: Header/header-annotation
              color: Text/text-primary
        token_hint:
          border-width: Stroke/stroke-action
          border-radius: Radius/radius-pill
          background: Background/Light/background-secondary
        behavior: BH-FILTER-SELECT
        data:
          source: server filter config for param apartment_type, in server order
          note: 'Order: Chung cư thương mại, Nhà ở xã hội, Duplex, Penthouse, Căn hộ dịch vụ mini, Tập thể cư xá, Officetel
            (Figma, designer-updated 2026-09-24)'
      - id: chip_4
        node_id: 1732:9862
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 3px
          justify-content: center
          align-items: center
          padding: 3px 10px 3px 10px
          width: 77.0px
          flex-shrink: 0
          height: 28.0px
          background: '#f4f4f4'
          border-radius: 999px
        ds_component:
          props:
            text: true
            change_l_icon: 875:185710
            right_icon: false
            left_icon: false
            size: Small 24px
            style: Fill
            select: 'No'
            state: Default
          figma_component: Chip
        content:
          labels:
          - node_id: I1732:9862;12236:833
            text: Penthouse
            box:
            - 10
            - 5
            - 57
            - 18
            style:
              font-family: '''Reddit Sans'''
              font-size: 12px
              font-weight: 600
              line-height: 18px
              color: '#222222'
            token_hint:
              typography: Header/header-annotation
              color: Text/text-primary
        token_hint:
          border-width: Stroke/stroke-action
          border-radius: Radius/radius-pill
          background: Background/Light/background-secondary
        behavior: BH-FILTER-SELECT
        data:
          source: server filter config for param apartment_type, in server order
          note: 'Order: Chung cư thương mại, Nhà ở xã hội, Duplex, Penthouse, Căn hộ dịch vụ mini, Tập thể cư xá, Officetel
            (Figma, designer-updated 2026-09-24)'
      - id: chip_5
        node_id: 1732:9867
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 3px
          justify-content: center
          align-items: center
          padding: 3px 10px 3px 10px
          width: 129.0px
          flex-shrink: 0
          height: 28.0px
          background: '#f4f4f4'
          border-radius: 999px
        ds_component:
          props:
            text: true
            change_l_icon: 875:185710
            right_icon: false
            left_icon: false
            size: Small 24px
            style: Fill
            select: 'No'
            state: Default
          figma_component: Chip
        content:
          labels:
          - node_id: I1732:9867;12236:833
            text: Căn hộ dịch vụ, mini
            box:
            - 10
            - 5
            - 109
            - 18
            style:
              font-family: '''Reddit Sans'''
              font-size: 12px
              font-weight: 600
              line-height: 18px
              color: '#222222'
            token_hint:
              typography: Header/header-annotation
              color: Text/text-primary
        token_hint:
          border-width: Stroke/stroke-action
          border-radius: Radius/radius-pill
          background: Background/Light/background-secondary
        behavior: BH-FILTER-SELECT
        data:
          source: server filter config for param apartment_type, in server order
          note: 'Order: Chung cư thương mại, Nhà ở xã hội, Duplex, Penthouse, Căn hộ dịch vụ mini, Tập thể cư xá, Officetel
            (Figma, designer-updated 2026-09-24)'
      - id: chip_6
        node_id: 1732:9876
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 3px
          justify-content: center
          align-items: center
          padding: 3px 10px 3px 10px
          width: 95.0px
          flex-shrink: 0
          height: 28.0px
          background: '#f4f4f4'
          border-radius: 999px
        ds_component:
          props:
            text: true
            change_l_icon: 875:185710
            right_icon: false
            left_icon: false
            size: Small 24px
            style: Fill
            select: 'No'
            state: Default
          figma_component: Chip
        content:
          labels:
          - node_id: I1732:9876;12236:833
            text: Tập thể, cư xá
            box:
            - 10
            - 5
            - 75
            - 18
            style:
              font-family: '''Reddit Sans'''
              font-size: 12px
              font-weight: 600
              line-height: 18px
              color: '#222222'
            token_hint:
              typography: Header/header-annotation
              color: Text/text-primary
        token_hint:
          border-width: Stroke/stroke-action
          border-radius: Radius/radius-pill
          background: Background/Light/background-secondary
        behavior: BH-FILTER-SELECT
        data:
          source: server filter config for param apartment_type, in server order
          note: 'Order: Chung cư thương mại, Nhà ở xã hội, Duplex, Penthouse, Căn hộ dịch vụ mini, Tập thể cư xá, Officetel
            (Figma, designer-updated 2026-09-24)'
      - id: chip_7
        node_id: 1732:9881
        tag: div
        figma_type: INSTANCE
        figma_name: Chip
        style:
          display: flex
          flex-direction: row
          gap: 3px
          justify-content: center
          align-items: center
          padding: 3px 10px 3px 10px
          width: 68.0px
          flex-shrink: 0
          height: 28.0px
          background: '#f4f4f4'
          border-radius: 999px
        ds_component:
          props:
            text: true
            change_l_icon: 875:185710
            right_icon: false
            left_icon: false
            size: Small 24px
            style: Fill
            select: 'No'
            state: Default
          figma_component: Chip
        content:
          labels:
          - node_id: I1732:9881;12236:833
            text: Officetel
            box:
            - 10
            - 5
            - 48
            - 18
            style:
              font-family: '''Reddit Sans'''
              font-size: 12px
              font-weight: 600
              line-height: 18px
              color: '#222222'
            token_hint:
              typography: Header/header-annotation
              color: Text/text-primary
        token_hint:
          border-width: Stroke/stroke-action
          border-radius: Radius/radius-pill
          background: Background/Light/background-secondary
        behavior: BH-FILTER-SELECT
        data:
          source: server filter config for param apartment_type, in server order
          note: 'Order: Chung cư thương mại, Nhà ở xã hội, Duplex, Penthouse, Căn hộ dịch vụ mini, Tập thể cư xá, Officetel
            (Figma, designer-updated 2026-09-24)'
    token_hint:
      padding-left: Padding/padding-large-20
      padding-top: Padding/padding-medium-16
      padding-right: Padding/padding-large-20
      padding-bottom: Padding/padding-medium-16
  - id: bottom_button
    node_id: 1732:9389
    tag: div
    figma_type: INSTANCE
    figma_name: Bottom Button
    style:
      display: flex
      flex-direction: row
      gap: 8px
      justify-content: center
      align-items: center
      padding: 16px
      width: 100%
      height: 72.0px
      background: '#ffffff'
      border-top: '1px solid #e8e8e8'
    ds_component:
      props:
        button: 2 button ↔️
      figma_component: Bottom Button
    content:
      labels:
      - node_id: I1732:9389;1263:23863;42:31667
        text: Xoá lọc
        box:
        - 72.25
        - 24
        - 55
        - 24
        style:
          font-family: '''Reddit Sans'''
          font-size: 16px
          font-weight: 700
          line-height: 24px
          color: '#222222'
        token_hint:
          typography: Label/label - page
          color: Text/text-primary
      - node_id: I1732:9389;1263:23859;42:31545
        text: Áp dụng
        box:
        - 243.25
        - 24
        - 64
        - 24
        style:
          font-family: '''Reddit Sans'''
          font-size: 16px
          font-weight: 700
          line-height: 24px
          color: '#222222'
        token_hint:
          typography: Label/label - page
          color: Text/text-primary
      surfaces:
      - node_id: I1732:9389;1263:23863
        box:
        - 16
        - 16
        - 167.5
        - 40
        style:
          background: '#ffffff'
          box-shadow: 'inset 0 0 0 1px #dddddd'
          border-radius: 8px
        token_hint:
          background: Background/Light/background-primary
          border-color: Border/border-regular
          border-radius: Radius/radius-card-small
      - node_id: I1732:9389;1263:23859
        box:
        - 191.5
        - 16
        - 167.5
        - 40
        style:
          background: '#ffd400'
          border-radius: 8px
        token_hint:
          border-radius: Radius/radius-card-small
    measured_width_px: 375.0
    token_hint:
      gap: Spacing/gap-x-small-8
      padding-left: Padding/padding-medium-16
      padding-top: Padding/padding-medium-16
      padding-right: Padding/padding-medium-16
      padding-bottom: Padding/padding-medium-16
      border-width: Stroke/stroke-divider
      background: Background/Light/background-primary
      border-color: Border/border-thin
    behavior: BH-FILTER-SELECT
  measured_width_px: 375.0
  token_hint:
    border-radius: Radius/radius-modal
    background: Background/Light/background-primary
    box-shadow: shadow-floating
```

# 3. Readiness report
## Design Readiness Report — NOXH filter on PTY Ad Listing

| | |
|---|---|
| Platform | PRD: **Android · iOS · Web**. Figma has: Desktop web · Mobile web (msite) · **1 App frame** (destination only) |
| Vertical | PTY (Nhà Tốt) |
| Figma | `vjLuAM8l0t1MyVjgDX3Y4m` · section `1732:9886` "GIT - ????" (Desktop `1732:5060` · Mobile `1732:5437`) |
| PRD | `ct-product-planning/specs/vertical/pty/noxh-add-filter-on-adlisting/product-spec.md` (draft, 2026-09-24) |
| Design Owner | Thảo Dương (inferred from session account — correct me if wrong) |
| Scope | FR-1 (UI: filter value list) · FR-2 (UI: entry tile + destination) · FR-3 is backend migration only, no UI |
| Audit source | REST `/nodes` full subtree (3,551 nodes) + `/v1/images` PNG of all 7 frames + Figma MCP `get_variable_defs` on all 8 in-scope blocks + Plugin API resolution of all 123 bound variable IDs to names (2026-09-24) |

## Tier 1 — DS Compliance

| # | Issue | Where | Note |
|---|---|---|---|
| T1-1 | NOXH tile icon is a **traced bitmap** (`image 1 [Vectorized]`, 3 vectors, raw `#000000` on raw `#ffffff`), not a DS icon and not a clean SVG | desktop `1748:5114` (layer misnamed `Image (Văn phòng, Mặt bằng kinh doanh)`) · msite `1748:5145` | The 4 existing tiles are also non-DS line illustrations, so this is a **category illustration asset** served with the tile config. **Resolved 2026-09-24:** the designer says the icon in Figma is final; dev exports it from these nodes |

Everything else in the feature nodes is bound: tile labels (`#595959` bound), applied chip (`#222222` bg / `#ffffff` text, bound), drawer chips + checkbox lists are DS instances (`Chip / Size=Small 24px, Style=Fill`, `List / Type=Checkbox, Size=Large 48px`). Minor: app applied chip `Close-outline` `1748:16290` has a raw `#ffffff` override (existing component, warning only).

## Tier 2 — Quality Score: **79 / 100 → READY WITH NOTES ⚠️**

- Annotation completeness **32/40** — Figma has 0 `[B]/[R]/[E]` annotations, but the PRD covers tap, inventory threshold (≥1), recalculation, fail-closed, back, ad-type reset. Missing: tile label wrap rule, value order in the list (conflict with Figma).
- Edge cases designed **22/30** — Entry (tile shown), destination (filter applied), filter list on desktop + msite are drawn. "0 listings → tile hidden" = the existing 4-tile row (no frame needed). Not drawn: App entry row, App filter sheet, selected state inside the list (existing behaviour).
- Interview completion **20/20** — expecting ≤ 4 intent questions, 0 fact questions.
- Figma file quality **5/10** — frames were imported from HTML: `Container` ×many, `Link`, `Frame 2085669532/…530/…534/…535`; desktop NOXH icon layer copy-named after the office tile; section named `GIT - ????`.

## ✅ Passed
- Tile is at position 5, last in the row, next to the sub-category tiles (matches PRD FR-2 placement).
- Destination shows sub-category `Căn hộ/Chung cư` + an applied `Nhà ở xã hội ×` filter chip on desktop and app.
- Both new values appear in the filter list on desktop (checkbox popover) and msite (chip drawer).

## ⚠️ Minor fixes (not blocking)
- Hard line break in the tile label: `"Nhà ở \nxã hội"` (both platforms). Other labels wrap naturally.
- msite NOXH tile skips the `Link` wrapper its 4 siblings have — structure differs from the siblings.
- PRD wording (OI-PRD): the contextual-filter entry is a **tile** in the sub-category row; the **chip** is the "Loại hình căn hộ" control in the filter bar that opens the popover (desktop) / bottom sheet (mobile). PRD also needs the param label "Loại hình căn hộ" and the Figma link.

## ❌ Blocking — needs an answer before handoff
- ~~Platform mismatch (U-7)~~ **resolved**: iOS/Android reuse the Mobile frames for the entry row and the filter sheet.
- ~~PRD ⚡ Figma conflicts~~ **resolved 2026-09-24**: label = "Loại hình căn hộ", 7 values; order = Chung cư thương mại → Nhà ở xã hội (the designer updated Figma to match the PRD; re-read verified).

## 🎨 Token audit — real bindings (added after the Figma MCP was authorized)
Correction: the first spec had `token_hint` values **inferred from hex/number** against a variable table from another file (FR-2). That breaks the skill's own rule. They have all been replaced with the variable/style **actually bound** on each node: 1,365 hints on 370/393 nodes, 0 not a real bound name. 10 nodes had an inferred hint but no binding, so that hint was removed.

| # | Finding | Count | New UI affected | Note |
|---|---|---|---|---|
| TK-1 | `Icon/*` token bound as a **background** | 19 nodes | `noxh_applied_chip_dt` (`Icon/icon-on-background`), `noxh_applied_chip_app` (`Icon/icon-primary`) | Same binding as the existing `Căn hộ/Chung cư ×` chip → comes from the HTML import |
| TK-2 | Text with **no text style**; size/weight/line-height/letter-spacing bound from different type groups | 76 text nodes | `noxh_tile_label_dt` / `_mw` | Same pattern as the 4 existing tiles |

Not blocking: the raw values are correct and the new nodes copy the existing ones. Filed as OI-TOKEN-ROLE (designer): rebind to semantic tokens / text styles when the file is cleaned up.

→ **Verdict: READY WITH NOTES ⚠️** — all blocking questions answered 2026-09-24.

## Part D — Naming defects (detection only; rename runs after you confirm the manifest)

| File | Current name | Problem |
|---|---|---|
| section | `GIT - ????` | Placeholder name |
| desktop `1748:5114` | `Image (Văn phòng, Mặt bằng kinh doanh)` | Copy of the office tile. It is the NOXH icon |
| desktop `1732:5061` · `1732:5222` | `Desktop/ Filter` ×2 | Duplicate names for 2 different screens |
| msite `1732:9159` · `1732:9404` | `mobile/ filter` ×2 | Duplicate names for 2 different screens |
| desktop `1748:8500` | `Mua Bán Chung Cư, Danh Sách 13.036…` | Named from sample data (page title) |
| `1732:5361` · `1732:9372` | `Drawer` ×2 | Desktop one is an anchored popover, not a drawer |
| many | `Container`, `Link`, `Frame 20856695xx` | Generic names from HTML import |

## Part B — Interview transcript (2026-09-24)
| # | Question (intent) | Answer |
|---|---|---|
| 1 | Tier 1: traced-bitmap NOXH icon | Handoff now. Later: the icon in Figma is final, dev exports it (OI-ICON resolved) |
| 2 | PRD vs Figma: label / values / order | Figma label + 7 values. Order later changed by the designer: **Chung cư thương mại first, then Nhà ở xã hội** (Figma updated, verified) |
| 3 | App frames missing for entry + sheet | iOS/Android reuse the Mobile frames |
| 4 | Scope Manifest | OK |
| 5 | Rename 104 layers | Yes, plus 2 more found by the gate (106 total) |
| 6 | Export mode | 1 file .md |
| 7 | (after the Figma MCP was authorized) | Tokens re-derived from real bindings; TK-1/TK-2 filed |
| 8 | Terminology | Tile = entry in the contextual filter (sub-category row). Chip = "Loại hình căn hộ" in the filter bar, opens popover (desktop) / bottom sheet (mobile) |
0 fact questions asked.

## Rename log
- Pass 1 (top level, 9) + pass 2 (inner layers of IN blocks, 95) on 2026-09-24 via `use_figma`; then 2 more (`filter_rows_app`, `breadcrumb_separator_dt`) after the gate flagged generic ids. **106 layers.** No layer inside a DS instance was touched. Old names backed up.
- REST re-read after renaming; every `figma_name` in the spec is post-rename.
