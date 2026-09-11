# Handoff — PTY / NOXH Primary Ad Detail (User Story 6)

> Spec version 1.0 · 2026-09-10 · produced by `handoff-design-spec` v2.0
>
> **Scope.** A dedicated Ad Detail page for a NOXH (Nhà Ở Xã Hội / social housing) primary listing — a unit ad sourced from the developer (Chủ đầu tư). The apply action binds to `project_id`, not `ad_id`.
>
> **Author sources.** Figma `UvLAgVP7Em1fwytMbuKHuv` frame `19027:50376` ("NEW - Ad Detail/ Primary"). PRD `carousell/ct-product-planning/specs/vertical/pty/noxh-hub-detail-prd.md` User Story 6. Two variant-legend frames `19027:50599` (project status chip) and `19027:50605` (sticky footer × interest chip). Reference-state frame `19027:49965` ("REF - Ad Detail/ Secondary") is context only — do NOT touch it.
>
> **Platform.** App only (iOS + Android). No desktop frame exists.
>
> **Design system.** Chợ Tốt DS v3.0 tokens (verified via `get_variable_defs`). Typography Reddit Sans.

## How to use this file

1. **Read Part 1 first** — the requirement in prose plus behaviours and content rules. It tells you *what and why*.
2. **Then Part 2** — YAML: `Reusable` → `Screens` → `Sections` → `Layout blocks` → `Verify checklist`. It tells you *what exactly, and how do I know I got it right*.
3. **Locate a section** via `Sections.<id>.find_by` (a Vietnamese grep string) and confirm with `Sections.<id>.confirm` before editing. Both live in Part 2.
4. **Every UI string is quoted verbatim in Vietnamese** — that is the only handle you have on the codebase. The prose stays English; only what the user sees is quoted.
5. **Every number in this file is a literal.** A design token is used only where the token's name is written; a bare number (`gap: 4`, `radius: 6`) is the value to build, never an approximation of a token. See `Tag definitions`.
6. **The build reports back.** Section `8_report` in the Verify checklist lists what you must measure and hand back after the build.

## Tag definitions

- **Numeric literals.** Any numeric value in this file is a literal. Implement it exactly as written. A design token is used only where the token's name is written (`gap-small-12`, `radius-card-small`). A bare number (`gap: 4`, `radius: 6`, `padding: 12`) is the value to build — it is not an approximation of a token, and there is no rule anywhere in this file that converts one to the other. Rounding `gap: 4` up or down is a defect: report it in `8_report`, do not perform it. **Values off the DS scale are listed for the designer to confirm; they are still literals.**
- **`type` on containers.** `type: column` / `row` / `stack` — required on every container. Never defaulted.
- **`align` default.** `center` when omitted.
- **Text colour default.** `text-primary` (`#222222`) when omitted.
- **`source:` on a leaf.** Absent = the leaf is fixed exactly as written (never write `source: fixed`). Present = data-bound; the leaf's `name` is a placeholder and its mapping lives in `Reusable.data_bound`.
- **`use:` on a tree entry.** References a `Reusable.<name>` component; all listed parameters are passed at the call site.
- **`action:` on a block.** `update` (replace-all — see verbatim wording in each `update` block), `remove`, or `new`.
- **`req:` / `cr:` / `behavior:` / `verifies:`.** Cross-references — the referenced id must exist in this file.
- **`axis_differs_from:`.** Only for a re-used id whose `type` (row vs column) genuinely changes across blocks. Prefer two ids with different names.
- **Vietnamese strings** are quoted with `"…"` verbatim (with accents). Placeholders inside a string are `{name}` — every placeholder has an entry in `content_rules` stating whether the value arrives with or without its unit.

## Open items

```yaml
blocking: []

before_qc:
  OI-1:
    issue: >
      "Đóng nhận hồ sơ" is the third state of the project-status chip. Screenshot shows it exists;
      the deep read of the variant symbols (19027:48091 and cousins on frame 19027:50599) was not
      completed. Colour and background are unread — the sibling states are green (Đang) and grey
      (Sắp), so the natural fit is `text-error` #f0325e on a lighter surface, but this is a guess.
    owner: design
    until_then: >
      Render "Đóng nhận hồ sơ" with the same shape and typography as the other two chip states
      (padding 4, radius 4, label-annotation 10/16 bold), background `background-secondary` #f4f4f4
      and text `text-tertiary` #8c8c8c until the designer confirms.

  OI-2:
    issue: >
      "Điều kiện mua" and "Điều kiện vay" tabs are Phase 2 per the PRD — the design has NO frames
      for these tabs in the NEW screen. It is unclear whether the tab bar reserves layout space
      today (empty), whether it is hidden, or whether nothing is drawn at all.
    owner: design + pm
    until_then: >
      Do NOT reserve layout space for the tab bar. Ship without the tabs entirely. If Phase 2
      lands later, the tabs slot in above `thong_tin_du_an_section`.

  OI-3:
    issue: >
      PRD lists "Thông tin dự án" and "Đặc điểm bất động sản" as two separate sections. The design
      renders them under one visual band: title-and-price area on top, then a stat row with a map
      thumbnail, then a small param table (BĐS traits). The read did not resolve whether the
      "Thông tin dự án" name has been retired or is a hidden section title.
    owner: design
    until_then: >
      Follow the design: no explicit "Thông tin dự án" heading. The header block already carries
      project name, price and stats; the amenities block (rating + reviews + tiện ích) carries
      location; the param block carries BĐS traits. Confirm with designer whether a heading is
      needed.

  OI-4:
    issue: >
      Sold-out state — PRD says the "Tư vấn hồ sơ" CTA is disabled when the project is fully sold
      out. The design shows a "Đóng hồ sơ" sticky-footer variant (single wide "Xem dự án khác"
      button, node 19027:48113). It is unclear whether "sold out" maps to this "Đóng hồ sơ" state
      or is a separate rendering.
    owner: pm
    until_then: >
      Treat "fully sold out" as an alias for the "Đóng hồ sơ" state — render the single wide
      "Xem dự án khác" button. Confirm.

  OI-5:
    issue: >
      "Các tin NOXH khác của CĐT" and "Các dự án NOXH khác" — the read captured one card in the
      first row (a Card Item Grid `19027:50512` measuring 156W × 280H — the ad-grid variant) and
      three cards in the second row (Card Item Grid `19027:50518`/`50540`/`50562` measuring 156W
      × 246H each). The row containers themselves scroll horizontally; the number of cards is
      data-driven. The single-card frame in the first row is a sample — the real list is a
      horizontal carousel.
    owner: dev + design
    until_then: >
      Build both as horizontal carousels of 156W cards, gap 12, initial-visible ≈ 2 cards + 3rd
      card peeking. Empty behaviour and card-count limit not read — treat empty as "hide the
      section" and cap card count at whatever the API returns.

  OI-6:
    issue: >
      Layer names in the NEW frame include ~95 auto-generated names (`Frame 2085…`, `Rectangle
      240…`, `Group`). The Phase-3a rename step was skipped this run — no `use_figma` write was
      performed. The spec keys off node ids, so a build is unaffected, but a subsequent read will
      still find the auto-names.
    owner: design
    until_then: >
      Ship the spec as-is. On the next run of `/handoff-design-spec` against this frame, the
      rename phase can consolidate names to the PRD's vocabulary.

  OI-7:
    issue: >
      Photo carousel component. The NEW frame reuses an existing `Carousel` DS instance for the
      photo gallery at the top of the ad, plus a Header (back / share / save) drawn as icons.
      Neither was deep-read this run — the sample shows "1/5" indicator, "PHONG 5 TỐT" tag, and
      the icons back / share (Facebook-fill) / save / more options.
    owner: dev
    until_then: >
      Reuse the existing photo-carousel component and header icon-row from the current Secondary
      Ad Detail page (`19027:49965`). No changes for this handoff.

cosmetic: {}
```

---

# PART 1 — Requirement, behaviours, content rules

```yaml
REQ-01:
  title: NOXH Primary Ad Detail — new dedicated page for a developer-listed NOXH unit
  context: >
    Today a NOXH primary ad (an ad_id created by the developer under a NOXH project_id) has no
    dedicated detail page. Buyers who tap a primary card on the NOXH Hub Tin đăng tab currently
    fall into the standard Secondary Ad Detail — a page built around per-agent contact CTAs
    ("Chat", "Gọi") that do not apply here. Primary units have no per-unit reservation and no
    per-ad seller — the apply flow goes to the developer at the project level.

    This spec introduces a new Ad Detail page that renders the same header/location/params/
    description shape as the Secondary Ad Detail — but replaces the agent block, the video
    section, the comments and the "similar listings" tail with two developer-scoped rows
    ("Các tin NOXH khác của CĐT", "Các dự án NOXH khác") and a sticky footer whose CTAs are
    "Quan tâm" (opt-in follow) and "Tư vấn hồ sơ" (apply, bound to project_id).

    The header carries a new element that does not exist on Secondary — a "Dự án: {name}"
    row with a coloured project-status chip and a "{n}+ người nộp" applicant count.

    Điều kiện mua / Điều kiện vay tabs are on the PRD as Phase-2 placeholders. They are NOT
    drawn in this Figma frame and NOT in scope of this handoff — see OI-2.

    Sections "Localities" and "Amenities" from the PRD render inline in the design as a single
    "Tổng quan khu vực" band (rating + reviews + "{n}+ tiện ích xung quanh") powered by the
    existing Nearby Amenities component — no new work.

  variant_a: "Default — buyer has not opted in and the project is accepting applications. See AC-1, AC-2."
  variant_b: "Buyer has already applied for this project. See AC-3."
  variant_c: "Project has closed applications (Đóng hồ sơ), including the sold-out case. See AC-4."
  interest_state: "Independent of A/B/C: `Chưa quan tâm` vs `Đã quan tâm`. See AC-5."

  render_when: >
    A NOXH primary ad is tapped from the NOXH Hub Tin đăng tab. Resolve the variant ONCE at page
    load: the eligibility state (A/B/C) comes from the apply flow's per-user-per-project record;
    the interest state comes from the per-user-per-project follow record. Never branch per row.

  rules: [CR-01, CR-02, CR-03, CR-04, CR-05, CR-06, CR-07, CR-08]
  behaviors: [BH-01, BH-02, BH-03, BH-04, BH-05, BH-06, BH-07]

  acceptance_criteria:
    AC-1: >
      Variant A (default). Sticky footer renders "Quan tâm" (tonal-neutral, icon left) beside
      "Tư vấn hồ sơ" (button-primary orange, icon left), per the `sticky_footer_variant_a` block.
      Tapping "Tư vấn hồ sơ" enters the apply flow bound to project_id — never ad_id (BH-03).
    AC-2: >
      Variant A. Header renders "Dự án: {project_name}" with the project-status chip on the right
      (state = `Đang nhận hồ sơ` — background `background-success-light` #ecf9f1, text
      `text-success` #12a154, per `Reusable.project_status_chip`), plus "{n}+ người nộp" beneath.
      Chip text is one of `"Đang nhận hồ sơ"` / `"Sắp nhận hồ sơ {dd/mm}"` / `"Đóng nhận hồ sơ"`
      per `content_rules.CR-04`.
    AC-3: >
      Variant B (already applied). Sticky footer's second slot renders "Đã gửi tư vấn" in a
      disabled visual state — background `button-tonal-neutral` #f4f4f4 with `text-tertiary`
      #8c8c8c label — and does not trigger the apply flow when tapped. "Quan tâm" behaves as in A.
    AC-4: >
      Variant C (Đóng hồ sơ — application closed or fully sold out per OI-4). Sticky footer
      renders a SINGLE wide button "Xem dự án khác" (background `button-tonal-neutral`, 375W minus
      side padding, 40H). "Quan tâm" is not shown in this variant. "Tư vấn hồ sơ" is not shown.
      Tapping "Xem dự án khác" navigates back to the NOXH Hub Tin đăng tab (BH-04).
    AC-5: >
      Interest state. The "Quan tâm" button on the LEFT of the sticky footer flips between two
      visual states: `Chưa quan tâm` (137W × 40H, icon + text "Quan tâm", tonal-neutral) and
      `Đã quan tâm` (56W × 40H, icon only, tonal-neutral). No hover intermediate.
    AC-6: >
      Header contains the following in reading order: photo carousel (existing DS `Carousel`, OI-7),
      header icon row (existing back / share / save / more, OI-7), title, param row, price row,
      dashed-top-border project-status row. The header's inner padding is 16 horizontal and 12
      vertical, owned by `header_container`. Nothing else contains the header — no wrapping card.
    AC-7: >
      "Các tin NOXH khác của CĐT" and "Các dự án NOXH khác" render as two separate horizontally
      scrollable rows above the sticky footer. Card width is 156, gap between cards is 12. Neither
      row exists on Secondary Ad Detail — they are net-new sections. See `other_ads_by_dev_row`
      and `other_projects_row` blocks.
    AC-8: >
      No section in the NEW screen implies that applying reserves the specific unit. The apply
      confirmation modal (out of this spec — belongs to `noxh-buyer-actions-prd.md`) must not
      contain the ad's Mã căn (ND121) or any wording that promises the specific apartment.
    AC-9: >
      Sections in REF (Secondary Ad Detail, `19027:49965`) that DO NOT appear on the NEW screen
      must not be built: `Agent Section`, `Video Section` (Video liên quan), `Comment`, and
      `Location Review – Trường hợp có đánh giá`. Rendering the NEW screen produces zero nodes
      whose names contain "Bình luận", "Đăng bởi", "Video liên quan", or "Chat nhanh:".
    AC-10: >
      Fluid width. Every container in the tree is fluid to the parent's width — no fixed 375W is
      hardcoded except at the screen root. Every row that mixes long and short children uses
      `flex: 1 0 0` + `min-width: 0` on the long child so truncation triggers.
    AC-11: >
      Colour tokens. Every colour in the built page comes from a DS token (per `Reusable.tokens`)
      or is one of the two documented raw values (`#000000` on the photo-carousel media background;
      the 0-to-0.75 rgba gradient on card-image bottom overlays). Reporting a `text-primary` at
      pure `#000000` is a defect: map it to `text-primary` (#222222) and list in `8_report`.
    AC-12: >
      Every interactive element in the NEW screen has its behaviour named in `behaviors` and its
      handler wired per `behavior:` in the tree.

behaviors:
  BH-01:
    element: '"Quan tâm" button on the sticky footer (both left-slot and full-width in variant C's absent case)'
    on_tap: >
      Toggle the per-user-per-project follow record. Optimistic — the button re-renders from
      `Chưa quan tâm` (137W with label) to `Đã quan tâm` (56W icon-only) without a full page
      reload. If the toggle fails, revert and show a toast "Không thể lưu, vui lòng thử lại".

  BH-02:
    element: '"Tư vấn hồ sơ" button on the sticky footer, variant A'
    on_tap: >
      Enter the apply flow with `project_id` in the payload. Never send `ad_id`. Downstream flow
      is owned by `noxh-buyer-actions-prd.md` User Story 1.

  BH-03:
    element: '"Đã gửi tư vấn" button on the sticky footer, variant B'
    on_tap: >
      No-op. Visually disabled. Do not open the apply flow; do not trigger any toast.

  BH-04:
    element: '"Xem dự án khác" button on the sticky footer, variant C'
    on_tap: >
      Navigate back to the NOXH Hub, Tin đăng tab. If the user arrived from the Dự án tab, still
      route to Tin đăng — the button is a discovery redirect, not a back button.

  BH-05:
    element: 'The linked "{project_name}" text in the project-status row (styled `text-link` #306bd9, semibold)'
    on_tap: >
      Navigate to the NOXH Project Detail page for this project (User Story 4 of the same PRD).
      Passes `project_id` in the route.

  BH-06:
    element: 'A card in "Các tin NOXH khác của CĐT" (each Card Item Grid at 156×280)'
    on_tap: >
      Open that ad's Primary Ad Detail page. Same screen, different data. Passes `ad_id`.

  BH-07:
    element: 'A card in "Các dự án NOXH khác" (each Card Item Grid at 156×246) and the "Xem tất cả" pill'
    on_tap: >
      Card → open that project's NOXH Project Detail page (passes `project_id`). "Xem tất cả" →
      NOXH Hub Dự án tab (no filter carry-over).

content_rules:
  CR-01:
    element: 'Header title `"NOXH Happy Home Nhơn Trạch - Mã căn NĐ121"`'
    rule: >
      Rendered as `"{project_name} - {ma_can_label}"`. `{project_name}` is the developer-supplied
      NOXH project name (no truncation logic within the string — the whole line line-clamps to 2
      lines at 255W). `{ma_can_label}` is the apartment code display attribute on the ad_id, in the
      form the developer entered (e.g. `Mã căn NĐ121`). If `{ma_can_label}` is empty, render the
      project_name alone with no trailing " - ".

  CR-02:
    element: 'Price row `"{price} · {price_per_m2} · {area}"`'
    rule: >
      Three cells separated by 1×16 vertical divider lines (`Vector 6139` in the design, but a
      1px full-height rule with `border-regular` #dddddd suffices at 16H). `{price}` = the full
      price ("1 tỷ", "890 triệu", "2,5 tỷ"). Value arrives with unit — do not append. `{price_per_m2}`
      = per-m² price ("14 tr/m²") — with unit. `{area}` = surface area ("70 m²") — with unit.
      Vietnamese number separators: `.` thousands, `,` decimal. If any of the three is missing,
      omit the cell AND the divider that precedes it. No trailing divider.

  CR-03:
    element: 'Param row `"2 PN"`, `"Hướng Đông Nam"`, `"Chung cư"`'
    rule: >
      Up to three fixed param slots in this order: bedrooms (e.g. `"{n} PN"` — value bare integer,
      unit hard-coded), orientation (e.g. `"Hướng {direction}"` — direction is one of Đông / Tây /
      Nam / Bắc / Đông Bắc / Đông Nam / Tây Bắc / Tây Nam), property type (e.g. `"Chung cư"`,
      `"Nhà phố"`). If a slot is empty, drop it and its trailing gap. The row is 255W and clips
      via `overflow: hidden` on each cell.

  CR-04:
    element: 'Project status chip (right of "Dự án: {project_name}" in the header)'
    rule: |
      Text and colour derived from `project.application_status` per this mapping:

      | Backend value        | Displayed as                | bg (token)                     | text (token)               |
      |----------------------|-----------------------------|--------------------------------|----------------------------|
      | Đang nhận hồ sơ      | `"Đang nhận hồ sơ"`         | background-success-light       | text-success               |
      | Sắp nhận hồ sơ       | `"Sắp nhận hồ sơ {dd/mm}"` | background-secondary           | text-secondary             |
      | Đóng nhận hồ sơ      | `"Đóng nhận hồ sơ"`         | see OI-1                       | see OI-1                   |

      `{dd/mm}` is the `Ngày ngừng nhận hồ sơ` date, sourced from Back Office (see the parent PRD's
      User Story 2 filter row and `noxh-backoffice-prd.md`). If that date has passed, the state
      auto-flips to `Đóng nhận hồ sơ` per the parent PRD's conflict rule — the date is source of
      truth. Format: `dd/mm` with a leading zero on single-digit day/month? — read the sibling
      hub filter for the canonical format (see `path_hint` below).

  CR-05:
    element: '"Dự án: {project_name}" line in the header project-status row'
    rule: >
      `"Dự án: "` is a static prefix, `text-primary` regular. `{project_name}` is the developer-supplied
      project name, rendered `text-link` #306bd9 semibold. The whole line line-clamps to one line
      at (343 − 40 image − 8 gap − chip width) — chip is a right-sibling that reserves its own
      width via `flex: 1 0 0` on the text column and `shrink-0` on the chip. Truncation ellipsis
      applies inside the linked span.

  CR-06:
    element: '"{n}+ người nộp" applicant count'
    rule: >
      `{n}` is a bucketed count: `50+`, `100+`, `200+`, `500+`, `1000+`. Value arrives as the
      formatted bucket string with the `+` — do not compute the bucket in the UI. If applicant
      count < 50, hide the whole row (avatars + text) and collapse the 2px vertical gap.

  CR-07:
    element: 'Price on a "Các dự án NOXH khác" card ("Từ 890 triệu - 1,5 tỷ")'
    rule: >
      `"Từ {min_price} - {max_price}"` when the project has both a floor and a ceiling per-unit
      price. Both values arrive with unit. If only one is set, render `"{price}"` alone (no "Từ"
      prefix). Vietnamese separators.

  CR-08:
    element: 'Card time-ago pill "2 giờ trước" on a "Các tin NOXH khác của CĐT" card'
    rule: >
      Relative time — `"{n} phút trước"`, `"{n} giờ trước"`, `"{n} ngày trước"`, `"{n} tuần trước"`,
      `"{n} tháng trước"`. `{n}` is a bare integer (unit hard-coded in the label). Cross-over
      thresholds: 60m → 1 giờ, 24h → 1 ngày, 7d → 1 tuần, 30d → 1 tháng.
```

---

# PART 2 — Reusable, Screens, Sections, Layout blocks, Verify checklist

```yaml
Reusable:
  # A component is built ONCE as a real reusable primitive, then called from every block that
  # uses it. Never inline a copy per call site. Before building any of these locally, check the
  # repo for an equivalent — most likely most of them already exist on Secondary Ad Detail.

  ds_components:
    Carousel:
      props: {}  # unread — reuse whatever the existing Secondary Ad Detail uses
      resolved_style: { fits_in: "375W × [aspect-1:1] height" }
      note: "Reuse Secondary Ad Detail's existing carousel — see OI-7."

    Header_icon_row:
      props: {}
      resolved_style: { fits_in: "375W × 44H", background: "background-primary" }
      note: "Back / Share / Save / More Options. Reuse Secondary Ad Detail. See OI-7."

    Button_pill_small:
      # Used for the header "Lưu" button and the "Xem tất cả" pill on the Các dự án NOXH khác header
      resolved_style:
        height: 32   # Lưu variant
        height_alt: 24  # Xem tất cả variant
        padding_x: 12  # Lưu; 8 for Xem tất cả
        padding_y: 4   # Lưu; 2 for Xem tất cả
        border: "1px border-regular #dddddd solid"
        radius: 999
        gap: 4
        bg: "button-blank #ffffff"
      props: { size: "small | x-small" }

    Button_primary_lg:
      # The orange CTA — used as "Tư vấn hồ sơ" in variant A only
      resolved_style:
        height: 40
        padding_x: 16
        padding_y: 8
        radius: 8
        gap: 8
        bg: "button-primary #fa6819"
        text_token: "label-page (16/24 bold)"
        text_colour_token: "text-on-background #ffffff"

    Button_tonal_neutral_lg:
      # Used as "Quan tâm" (variant a-b-c interest state Chưa) AND as "Đã gửi tư vấn" disabled
      # AND as "Xem dự án khác" full-width AND as the "Đã quan tâm" icon-only 56W
      resolved_style:
        height: 40
        padding_x: 16
        padding_y: 8
        radius: 8
        gap: 8
        bg: "button-tonal-neutral #f4f4f4"
        text_token: "label-page (16/24 bold)"
        text_colour_token: "text-primary #222222"
        text_colour_disabled: "text-tertiary #8c8c8c"

    Nearby_Amenities_row:
      # The single existing Nearby-Amenities block that the Secondary Ad Detail already renders
      props: {}
      resolved_style:
        # Absorbs both what PRD calls "Localities" AND "Amenities" — one component, not two
        fits_in: "375W × ~140H"
      note: >
        Reuse the existing Nearby Amenities component from Secondary Ad Detail. It renders the
        rating + `{n} lượt` review count + `{n}+ tiện ích xung quanh` and the "Xem tổng quan"
        button. No changes.

    Location_and_map_row:
      # Existing "Địa chỉ bất động sản" row with map thumbnail — reused from Secondary Ad Detail
      note: "Reuse from Secondary Ad Detail. No changes."

    Description_section:
      # Existing "Mô tả chi tiết" component — reused from Secondary Ad Detail (Description-section
      # instance appears at both 19027:50114 REF and 19027:50507 NEW)
      note: >
        Reuse the existing DS Description-section instance. Same behaviour: full text collapsed
        to line-clamp-3, "Xem thêm" reveals full text.

    Feature_params_section:
      # "Đặc điểm bất động sản" table of key/value rows — reused from Secondary Ad Detail
      note: "Reuse from Secondary Ad Detail with the 'Xem thêm' collapse."

  local_components:
    # Subtrees that repeat across blocks. Build once, call from every site — never inline a copy.

    project_status_chip:
      # Used at 4 sites: 1× header (`header_block_variant_default.project_status_chip`) and 3×
      # inside project cards (`other_projects_block` inline per card). Same 4-radius pill, same
      # 4-padding, same label-annotation typography — background and text colour flip by state.
      resolved_style:
        height: 16                              # padding 4/4 + line-height 16 for 10px text? actually text-only 16H approx
        padding: { x: 4 }
        radius: 4
        gap: 4
        typography: label-annotation
      params:
        text:                { type: string,     source: data }
        background_token:    { type: token_name, source: data }
        text_colour_token:   { type: token_name, source: data }
      tree:
        - id: chip_root
          type: row
          gap: 4
          align: center
          justify: center
          padding: { x: 4 }
          radius: 4
          background: "{background_token}"
          children:
            - id: chip_text
              type: text
              name: "{text}"
              typography: label-annotation
              colour: "{text_colour_token}"

  data_bound:
    # Only leaves whose value comes from data are marked here. Fixed leaves are written exactly as
    # they appear in the tree, with no `source:` key.

    project_status_chip:
      binds: [text, background_token, text_colour_token]
      source: "project.application_status, resolved via CR-04"
      mapping: "See CR-04 table. Backend value → chip text + tokens. Date `{dd/mm}` from `project.deadline_apply`."
      sample_data:
        text: "Đang nhận hồ sơ"
        background_token: "background-success-light"
        text_colour_token: "text-success"

    project_link_text:
      binds: [text, on_tap.project_id]
      source: "ad.project.name and ad.project.id"
      mapping: "Static prefix 'Dự án: ' + `{project_name}` linked; tap routes to Project Detail with project_id"
      sample_data: { project_name: "NOXH Happy Home Nhơn Trạch", project_id: 12345 }

    project_status_row_image:
      binds: image
      source: "project.logo_url or project.thumbnail_url"
      sample_data: "…/nam-long-logo.png"

    applicant_count_text:
      binds: text
      source: "project.applicant_count_bucket — pre-bucketed by backend per CR-06"
      sample_data: "200+ người nộp"

    ad_title:
      binds: text
      source: "ad.title — composed per CR-01"
      sample_data: "NOXH Happy Home Nhơn Trạch - Mã căn NĐ121"

    ad_price_row:
      binds: [price, price_per_m2, area]
      source: "ad.price / ad.price_per_m2 / ad.area — all arrive with unit per CR-02"
      sample_data: { price: "1 tỷ", price_per_m2: "14 tr/m²", area: "70 m²" }

    ad_param_row:
      binds: [bedrooms, orientation, property_type]
      source: "ad.bedrooms / ad.orientation / ad.property_type — per CR-03"
      sample_data: { bedrooms: "2 PN", orientation: "Hướng Đông Nam", property_type: "Chung cư" }

    other_ads_by_dev_card:
      binds: [thumbnail, time_ago, media_count, is_liked, title, param_bedrooms, param_orientation, price, price_per_m2, area, location]
      source: "GET /noxh/ads?developer_id={dev_id}&exclude_ad_id={current_ad_id}&limit=N"
      mapping: >
        Each card = one ad. `time_ago` per CR-08. `media_count` is `ad.media.length`.
        `is_liked` toggles heart-fill vs heart-outline. `title` per CR-01. `location` = `ad.district_display`
        (short form, e.g. "H. Nhơn Trạch").
      sample_data:
        thumbnail: "…/happy-home-nd121.jpg"
        time_ago: "2 giờ trước"
        media_count: 6
        is_liked: false
        title: "NOXH Happy Home Nhơn Trạch - Mã căn NĐ121"
        param_bedrooms: "3 PN"
        param_orientation: "Hướng Tây Nam"
        price: "1,7 tỷ"
        price_per_m2: "35 tr/m²"
        area: "81 m²"
        location: "H. Nhơn Trạch"

    other_projects_card:
      binds: [thumbnail, project_status_chip_text, project_status_chip_tokens, media_count, extra_media_flag, title, price_range, location]
      source: "GET /noxh/projects?exclude_project_id={current_project_id}&limit=N"
      mapping: >
        `project_status_chip_*` per CR-04. `price_range` per CR-07. `location` is
        `project.district_display P. {ward_display}` (long form since space is wider — see
        `19027:50539` sample "Quận Bình Thạnh P. Gia Định mới").
      sample_data:
        thumbnail: "…/binh-chanh-lo-b.jpg"
        project_status_chip_text: "Đang nhận hồ sơ"
        project_status_chip_tokens: { bg: "background-success-light", text: "text-success" }
        media_count: 6
        extra_media_flag: "photo-plus-video mixed"
        title: "NOXH Bình Chánh - Lô B"
        price_range: "Từ 890 triệu - 1,5 tỷ"
        location: "Quận Bình Thạnh P. Gia Định mới"

    sticky_footer_state:
      binds: variant
      source: >
        Composed from `apply_status` (from per-user-per-project apply record) and
        `project.application_status`. Resolution: `project.application_status == 'closed'` OR
        `project.units_available == 0` → variant C. Else `apply_status == 'submitted'` → variant B.
        Else → variant A. (See OI-4 on sold-out.)
      sample_data: A

    interest_state:
      binds: variant
      source: "per-user-per-project follow record — boolean"
      mapping: "false → `Chưa quan tâm` (137W with label); true → `Đã quan tâm` (56W icon-only)"
      sample_data: "Chưa quan tâm"

  tokens:
    # Every colour used in the built page. Phase 3b resolved these — see the resolution table
    # below Reusable. Every hex here is verified against `get_variable_defs` on the file.
    background-primary:        { hex: "#ffffff", role: "the page and card ground — plain white" }
    background-secondary:      { hex: "#f4f4f4", role: "a neutral tonal surface — one shade off white, no hue" }
    background-app:            { hex: "#f7f7f7", role: "the page ground behind cards" }
    background-success-light:  { hex: "#ecf9f1", role: "a very pale mint green — chip surface for the success state" }
    background-overlay:        { hex: "#22222280", role: "a translucent dark overlay for image gradients (see 8_report on raw gradient values)" }

    text-primary:              { hex: "#222222", role: "the darkest text, near-black — not pure black" }
    text-secondary:            { hex: "#595959", role: "a mid grey, one step lighter than primary" }
    text-tertiary:             { hex: "#8c8c8c", role: "a lighter grey, for caption / secondary metadata" }
    text-blank:                { hex: "#ffffff", role: "white text — over dark image gradients only" }
    text-on-background:        { hex: "#ffffff", role: "white text — over the orange primary button" }
    text-link:                 { hex: "#306bd9", role: "a deep blue link colour" }
    text-error:                { hex: "#f0325e", role: "a vivid pink-red — for price and other emphasis" }
    text-success:              { hex: "#12a154", role: "a mid green — for the success chip" }

    icon-primary:              { hex: "#222222", role: "same as text-primary — icon fill" }
    icon-tertiary:             { hex: "#8c8c8c", role: "same as text-tertiary — icon fill for muted state" }
    icon-blank:                { hex: "#ffffff", role: "white icon over dark or coloured backgrounds" }

    border-regular:            { hex: "#dddddd", role: "the standard border grey — on pills, dividers" }
    border-divider:            { hex: "#f4f4f4", role: "a very subtle horizontal divider — used across full-width horizontal rules" }
    border-thin:               { hex: "#e8e8e8", role: "a lighter alternative to border-regular" }

    button-primary:            { hex: "#fa6819", role: "the CTA orange — Tư vấn hồ sơ" }
    button-tonal-neutral:      { hex: "#f4f4f4", role: "the tonal grey button — Quan tâm, disabled Đã gửi tư vấn, Xem dự án khác" }
    button-blank:              { hex: "#ffffff", role: "white pill button surface with a border — Lưu, Xem tất cả" }

  typography:
    display-annotation:   { family: "Reddit Sans", weight: 700, size: 18, line: 26, letter: 0 }
    display-caption:      { family: "Reddit Sans", weight: 700, size: 20, line: 28, letter: 0 }
    header-page:          { family: "Reddit Sans", weight: 600, size: 20, line: 28, letter: 0 }
    header-caption:       { family: "Reddit Sans", weight: 600, size: 14, line: 20, letter: 0 }
    label-page:           { family: "Reddit Sans", weight: 700, size: 16, line: 24, letter: 0 }
    label-section:        { family: "Reddit Sans", weight: 700, size: 14, line: 20, letter: 0 }
    label-caption:        { family: "Reddit Sans", weight: 700, size: 12, line: 18, letter: 0 }
    label-annotation:     { family: "Reddit Sans", weight: 700, size: 10, line: 16, letter: 0 }
    body-section:         { family: "Reddit Sans", weight: 400, size: 14, line: 20, letter: 0 }
    body-caption:         { family: "Reddit Sans", weight: 400, size: 12, line: 18, letter: 0 }
    body-annotation:      { family: "Reddit Sans", weight: 400, size: 10, line: 16, letter: 0 }
    tagline-caption:      { family: "Reddit Sans", weight: 500, size: 14, line: 20, letter: 0 }
    tagline-annotation:   { family: "Reddit Sans", weight: 500, size: 12, line: 18, letter: 0 }

  icons:
    - Icon-Left (bookmark) — on the "Lưu" pill in the header
    - Icon-Left (bell-outline) — on the "Quan tâm" button (`Chưa quan tâm` state)
    - Icon-Left (bell-fill) — on the "Đã quan tâm" 56W state (visual guess — confirm with designer)
    - Icon-Left (envelope or send arrow) — on the "Tư vấn hồ sơ" button (visual guess — confirm)
    - Heart-fill / Heart-outline stacked — top-right of `other_ads_by_dev_card` (24×24 hit area, 20×20 fills, stacked with `mr: -20`)
    - Location-fill 16×16 icon — leading location text in `other_projects_card` and `other_ads_by_dev_card`
    - Stat icon 12×12 — media count icons on both card variants (photo, mixed)

  sample_data:
    project_name: "NOXH Happy Home Nhơn Trạch"
    ma_can_label: "Mã căn NĐ121"
    project_status_text: "Đang nhận hồ sơ"
    applicant_count_bucket: "200+"
    price: "1 tỷ"
    price_per_m2: "14 tr/m²"
    area: "70 m²"
    bedrooms: "2 PN"
    orientation: "Hướng Đông Nam"
    property_type: "Chung cư"

# =====================================================================================
# Phase 3b — token resolution report
# =====================================================================================
# Every colour observed in the deep-read of the four in-scope subtrees, and how it resolved.
# Format:  | Node id                        | Hex        | → Token                       | How |
#          |--------------------------------|------------|-------------------------------|-----|
#          | 19027:50399 root fill          | #ffffff    | background-primary            | bound (variable already set) |
#          | 19027:50404 title text         | #222222    | text-primary                  | bound |
#          | 19027:50405 Lưu button fill    | #ffffff    | button-blank                  | bound |
#          | 19027:50405 Lưu button border  | #dddddd    | border-regular                | bound |
#          | 19027:50411 price text         | #f0325e    | text-error                    | bound |
#          | 19027:50413/50415 unit text    | #222222    | text-primary                  | bound |
#          | 19027:50416 dashed border      | #dddddd    | border-regular                | bound (dashed variant) |
#          | I…48077 span "Dự án:" prefix   | #222222    | text-primary                  | context-resolved (text role) |
#          | I…48077 span "{project_name}"  | #306bd9    | text-link                     | bound |
#          | I…48078 chip bg (Đang…)        | #ecf9f1    | background-success-light      | bound |
#          | I…48078 chip text (Đang…)      | #12a154    | text-success                  | bound |
#          | I…48084 "200+ người nộp"       | #222222 α80 | text-primary + opacity 0.8   | bound + raw opacity — flagged in 8_report |
#          | 19027:50510 section title      | #222222    | text-primary                  | bound |
#          | 19027:50512 card ground        | #ffffff    | background-primary            | bound |
#          | thumbnail overlay gradient     | #000000 α0→α0.75 | KEPT RAW (photo overlay) | media background — intentional per Phase 3b rule |
#          | heart-fill shadow rgba         | #59595926  | shadow-below effect token     | bound |
#          | 19027:50521 chip bg            | #ecf9f1    | background-success-light      | bound |
#          | 19027:50565 chip bg (Sắp…)     | #f4f4f4    | background-secondary          | bound |
#          | 19027:50565 chip text          | #595959    | text-secondary                | bound |
#          | 19027:50516 Xem tất cả bg      | #ffffff    | button-blank                  | bound |
#          | 19027:50516 border             | #dddddd    | border-regular                | bound |
#          | Card 3 price text              | #f0325e    | text-error                    | bound |
#          | sticky footer Button 48119 bg  | #fa6819    | button-primary                | bound |
#          | sticky footer Button 48119 txt | #ffffff    | text-on-background            | bound |
#          | sticky footer Button 48118 bg  | #f4f4f4    | button-tonal-neutral          | bound |
#          | sticky footer Button 48118 txt | #222222    | text-primary                  | bound |
#          | Home indicator bar             | #222222    | icon-primary                  | bound |
#
# Resolution summary:
#   - bound: 26 (all real DS-token resolutions from `boundVariables`)
#   - context-resolved: 1 ("Dự án:" prefix — role = text, fell to text-primary)
#   - kept raw: 1 (photo overlay gradient — media background intentional)
#   - matched: 0 (no case-2 hex-lookup was needed — the file is well-tokenised)
#   - unresolved (open item): 1 (see OI-1 — Đóng nhận hồ sơ chip colours)

# =====================================================================================
Screens:
  primary_ad_detail:
    renders: [
      header_block_variant_default,
      location_and_amenities_block,
      feature_params_block,
      description_block,
      other_ads_by_dev_block,
      other_projects_block,
      sticky_footer_variant_a,   # replaced with variant_b or variant_c per sticky_footer_state
    ]
    place: |
      NavigationStack
        StatusBar                (44H, DS system chrome — not drawn by this feature)
        HeaderIconRow            (existing carousel/back/save/share icons — Reusable.Header_icon_row — see OI-7)
        Carousel                 (existing photo carousel — Reusable.Carousel — see OI-7)
        ★ header_block                       ← this handoff replaces this content
        ★ location_and_amenities             ← reuses Nearby Amenities but slot is here
        ★ feature_params                     ← reuses Feature params section
        ★ description                        ← reuses Description-section
        ★ other_ads_by_dev                   ← NEW — this handoff builds
        ★ other_projects                     ← NEW — this handoff builds
        ★ sticky_footer                      ← replaces the Secondary ad's chat/gọi row
        HomeIndicator            (34H, DS system chrome — not drawn by this feature)
    parent_provides:
      - "The NavigationStack owns the top of the scroll and the safe-area top inset."
      - "The scroll owns the vertical gap between sections — DO NOT add a top or bottom margin to a section root. Set page background = background-app #f7f7f7 on the scroll; each section root is background-primary #ffffff so cards separate visually as bands."
    untouched: [Carousel, HeaderIconRow, StatusBar, HomeIndicator]
    page_background: background-app
    slot_check: { gap_above: 0, gap_below: 0 }

  # No second screen — this handoff is a single screen, App platform only. Desktop out of scope.

# =====================================================================================
Sections:

  header_container:
    acted_on_by: [header_block_variant_default]
    figma:
      primary_ad_detail: { node: "19027:50399", name: "Header Container" }
    path_hint: { path: "unknown", provenance: "unknown", fill_on_first_build: true }
    find_by: |
      Grep for the concatenation `NOXH Happy Home Nhơn Trạch` — this is the sample project name
      that appears in the header title as `{project_name} - Mã căn NĐ121`. Placeholder form:
      grep for the ad title component that receives `ad.title`. The near-miss is the CARD title
      on `other_ads_by_dev_card` — a smaller body-section text of the same string; separate them
      by the surrounding padding (16h × 12v vs 4h × 8v).
    confirm: |
      The correct node is the WIDEST title text in the ad — 255W × 52H, display-annotation style
      (18/26 bold), inside a container whose row-sibling is a right-aligned "Lưu" bookmark pill
      (32H, radius 999, border-regular). If the surrounding container has no such pill sibling,
      you are inside a card, not the header — stop and re-locate.
    current: |
      Today, on the Secondary Ad Detail (`19027:49965`), the header container renders title +
      "Xem lịch sử giá" link + address block. The Primary version REPLACES that content: no
      "Xem lịch sử giá" line, add the project-status row instead.
    owns:
      this_section_provides: [the section surface (background-primary #ffffff), inner padding 16 horizontal and 12 vertical, the vertical gap 12 between Listing Container and Project Status row]
      therefore: >
        The located node ALREADY has the surface and the 16×12 padding. Do not wrap it in a card
        or add a second inner padding — build its children into it. If you find yourself creating
        a new `<div style={padding: 16px 12px}>` around the header contents, stop.
    padding_check: { inner_x: 16, inner_y: 12, surfaces: 1 }

  location_amenities_container:
    acted_on_by: [location_and_amenities_block]
    figma:
      primary_ad_detail: { node: "19027:50417", name: "Location Container" }
    path_hint: { path: "unknown", provenance: "unknown", fill_on_first_build: true }
    find_by: |
      Grep for the exact heading `Địa chỉ bất động sản` — it is the fixed section heading on both
      REF and NEW. Near miss: `Đặc điểm bất động sản` — one word different, DIFFERENT section
      (Feature params). Separate by the presence of a small square map thumbnail (aspect ~1:1)
      on this section, and by the "Xem tổng quan" pill button beneath.
    confirm: |
      The correct node contains BOTH the address text and a Nearby-Amenities row with a "3.9",
      "21 lượt", "84+" stat table and a "Xem tổng quan" pill button. If it contains a "Bình
      luận" list or a "Chat nhanh" input, you are inside a different section — the location
      section never contains chat UI.
    current: |
      Existing on Secondary Ad Detail. Reused as-is — this section is unchanged in NEW.
    owns:
      this_section_provides: [surface background-primary #ffffff, inner padding via the Nearby Amenities component itself]
      therefore: "Do NOT re-pad. The existing component provides its own padding."
    padding_check: { inner_x: "component-owned", inner_y: "component-owned", surfaces: 1 }

  feature_params_container:
    acted_on_by: [feature_params_block]
    figma:
      primary_ad_detail: { node: "19027:50459", name: "Info Container" }
    path_hint: { path: "unknown", provenance: "unknown", fill_on_first_build: true }
    find_by: |
      Grep for the exact heading `Đặc điểm bất động sản` — one of two similarly-named headings
      (near miss: `Địa chỉ bất động sản` — the location section). Separate by the presence of a
      key/value table beneath ("Tình trạng BĐS", "Hướng nhà", "Diện tích").
    confirm: |
      The correct node contains a table of at least 3 key/value rows and a "Xem thêm ▾" expander.
      No map, no rating. If the expander is missing, the section is not the correct one.
    current: |
      Reused from Secondary Ad Detail. Unchanged in NEW.
    owns:
      this_section_provides: [surface, inner padding via the DS Feature-params section]
      therefore: "Do NOT re-pad."
    padding_check: { inner_x: "component-owned", inner_y: "component-owned", surfaces: 1 }

  description_container:
    acted_on_by: [description_block]
    figma:
      primary_ad_detail: { node: "19027:50507", name: "Description-section" }
    path_hint: { path: "unknown", provenance: "unknown", fill_on_first_build: true }
    find_by: |
      Grep for the exact heading `Mô tả chi tiết`.
    confirm: |
      Contains a long body paragraph and a "Xem thêm" affordance. Only ONE section in the ad has
      the "Mô tả chi tiết" heading — no near miss.
    current: |
      Reused from Secondary Ad Detail (the same DS Description-section instance appears at
      `19027:50114` REF and `19027:50507` NEW).
    owns:
      this_section_provides: [surface, inner padding via the DS Description-section]
      therefore: "Do NOT re-pad."
    padding_check: { inner_x: "component-owned", inner_y: "component-owned", surfaces: 1 }

  other_ads_by_dev_section:
    acted_on_by: [other_ads_by_dev_block]
    figma:
      primary_ad_detail: { node: "19027:50508", name: "Other NOXH Ads By Developer" }
    path_hint: { path: "unknown", provenance: "unknown", fill_on_first_build: true }
    find_by: |
      Grep for the exact heading `Các tin NOXH khác của CĐT`. NEW SECTION — does NOT exist on
      Secondary Ad Detail. Near miss: `Các dự án NOXH khác` (the neighbouring section below); the
      words share "NOXH" but the leading noun is different (`tin` = ad vs `dự án` = project).
    confirm: |
      Contains one horizontally scrolling row of ad cards. Each card is 156W × 280H, shows a
      photo with a media-count badge and a heart, then the ad title / bedrooms · orientation /
      price · price/m² · area / location. If cards show a "Từ … - …" price range (denotes a
      project-level price band) they are in the OTHER section — stop and re-locate.
    current: |
      Does not exist today. This is a NEW section (`action: new`).
    owns:
      this_section_provides: [surface background-primary #ffffff, inner padding 16h horizontally, 12 top / 16 bottom vertically, the gap 12 between heading and the scrolling row]
      therefore: "The section root already carries the surface and the padding. Do not wrap the row in a card."
    padding_check: { inner_x: 16, inner_y: "12t/16b", surfaces: 1 }

  other_projects_section:
    acted_on_by: [other_projects_block]
    figma:
      primary_ad_detail: { node: "19027:50513", name: "Other NOXH Projects" }
    path_hint: { path: "unknown", provenance: "unknown", fill_on_first_build: true }
    find_by: |
      Grep for the exact heading `Các dự án NOXH khác`. NEW SECTION. Near miss: `Các tin NOXH
      khác của CĐT`. Separate by the presence of a "Xem tất cả" pill button in the heading row
      (this section has it; the ads section does not).
    confirm: |
      Contains a "Xem tất cả" pill on the right of the heading, and a horizontally scrolling row
      of PROJECT cards (156W × 246H). Each project card shows a status chip on top-left of the
      thumbnail, a price range ("Từ 890 triệu - 1,5 tỷ"), a longer address, and NO heart badge
      (project cards do not toggle favourite). If the cards show a heart, they are in the OTHER
      section.
    current: |
      Does not exist today. `action: new`.
    owns:
      this_section_provides: [surface background-primary #ffffff, inner padding 16h horizontally, 12 top / 16 bottom vertically, gap 12 between heading and row]
      therefore: "Section root has the surface and padding. Do not wrap."
    padding_check: { inner_x: 16, inner_y: "12t/16b", surfaces: 1 }

  sticky_footer_section:
    acted_on_by: [sticky_footer_variant_a, sticky_footer_variant_b, sticky_footer_variant_c]
    figma:
      primary_ad_detail: { node: "19027:50594", name: "bottom-button" }
    path_hint: { path: "unknown", provenance: "unknown", fill_on_first_build: true }
    find_by: |
      Grep for the button label `Tư vấn hồ sơ` — the CTA text appears ONLY in this footer. Near
      miss: `Đã gửi tư vấn` (the disabled variant B label) which shares "tư vấn". Separate by
      grepping for the parent container that is `position: fixed` at the bottom of the viewport
      and has 90H total (56 buttons row + 34 iOS home indicator).
    confirm: |
      The correct container is the ONLY absolutely-positioned bottom-fixed bar on this screen. If
      you find a bar that also contains a "Chat" or "Gọi" button, you are inside Secondary Ad
      Detail's footer — stop, you are on the wrong screen entirely.
    current: |
      On Secondary Ad Detail today, this slot renders a Chat / Gọi row plus a "Đăng bởi" author
      panel above the footer. On Primary, the entire footer is REPLACED with the button-group
      variant, and the "Đăng bởi" author panel does NOT render at all (see AC-9).
    owns:
      this_section_provides: [background-primary #ffffff, the horizontal 16 padding around the button-group, the 8 vertical padding, the Home Indicator space beneath]
      therefore: "The section root is the sticky bar. Do not wrap in another sticky container."
    padding_check: { inner_x: 16, inner_y: 8, surfaces: 1 }

# =====================================================================================
Layout blocks:

  # An AC sends you here.  screen: → tells you which page.  section: → sends you to Sections to
  # locate the node.  action: → says what to do to it.  what: → describes the result.  The tree
  # IS the result.  Every number in the tree is a literal — see Tag definitions.

  # ---------------------------------------------------------------------------------------------
  header_block_variant_default:
    screen: primary_ad_detail
    section: header_container
    requirement: REQ-01
    verifies: [AC-2, AC-6, AC-10, AC-11]
    action: >
      UPDATE. Keep the section node and everything around it; replace ALL of its contents with
      the tree below. The tree IS the finished contents — whatever renders today and is not in
      the tree does not survive the replace.
    what: >
      A white block, 375 wide with 16-horizontal/12-vertical padding. Inside, a single column
      with gap 12 holds two rows: (1) the Listing Container — a 343-wide sub-column with gap 4
      that renders the ad title `"NOXH Happy Home Nhơn Trạch - Mã căn NĐ121"` in `display-annotation`
      bold on the left (255W, wraps to two lines) and a compact `"Lưu"` bookmark pill on the
      right (32H, radius 999, bordered white), then a param row `"2 PN"` / `"Hướng Đông Nam"` /
      `"Chung cư"` in `body-section` grey, then a price row `"1 tỷ" · "14 tr/m²" · "70 m²"` where
      the price is `header-page` red and the unit prices are `header-caption` primary, separated
      by thin 16-tall vertical rules. (2) The Project Status row — a 343-wide row with an 8 gap,
      a dashed top border above 12 of top space and 2 of bottom, holding a 40×40 developer logo
      thumbnail on the left, and a fluid column on the right containing a line with `"Dự án: "`
      + a linked-blue semibold `"{project_name}"` on the LEFT and a small green
      `"Đang nhận hồ sơ"` chip on the RIGHT, then a row of three 16×16 overlapping avatars
      followed by `"200+ người nộp"` in tiny grey annotation type.
    figma: "19027:50399"
    reference_width: 375
    maths: "16 + (12 + 52title + 4 + 20params + 4 + 28price + 12 + [1 dashed border] + 12 + 40 image row (equal to 40 image height) + 12) + 12 = 16 + 197 + 12 = ??? — recompute per real layout resolution during build; approx ≈ 220"
    computed:
      header_container_root: "375 × 198"        # measured
      listing_container: "343 × ~118"
      listing_title_row: "343 × 52"
      listing_title_text: "255 × 52"
      lu_pill: "auto × 32"
      param_group: "255 × 20"
      product_price: "auto × 28"
      project_status_row: "343 × ~66"           # 40 image + inner column
      project_status_image: "40 × 40"
      project_link_line: "flex × 20"
      project_status_chip: "auto × 16 (padding 4)"
      applicant_avatar_row: "auto × 16"
    tree:
      - id: header_container_root
        type: column
        width: 375
        padding: { x: 16, y: 12 }
        gap: 0
        background: background-primary
        children:
          - id: listing_container
            type: column
            width: fluid
            gap: 12
            children:

              - id: listing_details_container
                type: column
                width: 343
                gap: 4
                children:

                  - id: listing_title_row
                    type: row
                    width: fluid
                    align: center
                    justify: space-between
                    children:
                      - id: ad_title
                        type: text
                        width: 255
                        height: 52          # two lines of 26
                        source: data
                        name: "{ad_title}"
                        typography: display-annotation
                        colour: text-primary
                        max_lines: 2
                        overflow: ellipsis
                      - id: save_button
                        use: Button_pill_small
                        params:
                          size: small
                          icon_left: "bookmark-outline (existing DS icon, 20×20)"
                          label: "Lưu"
                          behavior: "existing save-ad handler — reuse from Secondary Ad Detail. No new behavior."

                  - id: ad_param_row
                    type: row
                    width: 255
                    gap: 8
                    align: center
                    overflow: hidden
                    children:
                      - id: param_bedrooms
                        type: text
                        source: data
                        name: "{bedrooms}"
                        typography: body-section
                        colour: text-tertiary
                        overflow: ellipsis
                        max_lines: 1
                      - id: param_orientation
                        type: text
                        source: data
                        name: "{orientation}"
                        typography: body-section
                        colour: text-tertiary
                        overflow: ellipsis
                        max_lines: 1
                      - id: param_property_type
                        type: text
                        source: data
                        name: "{property_type}"
                        typography: body-section
                        colour: text-tertiary
                        overflow: ellipsis
                        max_lines: 1

                  - id: product_price_row
                    type: row
                    width: fit
                    align: center
                    gap: 8
                    children:
                      - id: price_value
                        type: text
                        source: data
                        name: "{price} "         # trailing space intentional per design ("1 tỷ ")
                        typography: header-page
                        colour: text-error
                      - id: price_divider_1
                        type: rule
                        width: 1
                        height: 16
                        colour: border-regular
                      - id: price_per_m2_value
                        type: text
                        source: data
                        name: "{price_per_m2}"
                        typography: header-caption
                        colour: text-primary
                      - id: price_divider_2
                        type: rule
                        width: 1
                        height: 16
                        colour: border-regular
                      - id: area_value
                        type: text
                        source: data
                        name: "{area}"
                        typography: header-caption
                        colour: text-primary

              - id: project_status_row
                type: row
                width: 343
                gap: 8
                align: start
                padding: { top: 12, bottom: 2 }
                border_top: { style: dashed, width: 1, colour: border-regular }
                children:
                  - id: project_status_row_image
                    type: image
                    width: 40
                    height: 40
                    radius: 4
                    source: data
                    name: "{project_status_row_image}"
                  - id: project_status_text_column
                    type: column
                    width: fluid                        # flex: 1 0 0, min-width: 0
                    gap: 2
                    justify: center
                    children:
                      - id: project_link_line
                        type: row
                        width: fluid
                        align: center
                        children:
                          - id: project_link_text
                            type: rich_text
                            source: data
                            name: 'Dự án: <link>{project_name}</link>'
                            typography_prefix: body-section          # "Dự án: "
                            colour_prefix: text-primary
                            typography_link: header-caption          # semibold
                            colour_link: text-link
                            width: fluid                              # flex 1
                            max_lines: 1
                            overflow: ellipsis
                            behavior: BH-05
                          - id: project_status_chip
                            use: project_status_chip                  # from Reusable — same component name for symmetry
                            params:
                              source: data
                              name: "{project_status_chip.text}"      # resolves via CR-04
                              background_token: "{project_status_chip.background_token}"
                              text_colour_token: "{project_status_chip.text_colour_token}"
                              padding: 4
                              radius: 4
                              typography: label-annotation

                      - id: applicant_avatar_row
                        type: row
                        gap: 2
                        align: center
                        width: fit
                        children:
                          - id: applicant_avatars
                            type: overlapping_avatar_stack
                            count: 3
                            item_size: 16
                            overlap: 9                                # ml offsets 0, 7, 16 → 9 overlap first, 9 second
                            source: data
                            name: "{recent_applicant_avatar_urls}"
                          - id: applicant_count_text
                            type: text
                            source: data
                            name: "{applicant_count_bucket} người nộp"
                            typography: body-annotation
                            colour: text-primary
                            opacity: 0.8

  # ---------------------------------------------------------------------------------------------
  location_and_amenities_block:
    screen: primary_ad_detail
    section: location_amenities_container
    requirement: REQ-01
    verifies: [AC-9]
    action: >
      UPDATE. Keep the section node and everything around it; replace ALL of its contents with
      the tree below. The tree IS the finished contents — whatever renders today and is not in
      the tree does not survive the replace.
    what: >
      The existing `Địa chỉ bất động sản` block on Secondary Ad Detail, rendered verbatim: an
      address line, a small square map thumbnail, and the Nearby-Amenities row with the rating
      / review count / tiện ích count and the "Xem tổng quan" pill and "Hỏi thêm người đăng"
      pill. NO changes from Secondary.
    figma: "19027:50417"
    reference_width: 375
    maths: "component-owned"
    computed:
      location_amenities_root: "375 × ~264"
    tree:
      - id: location_amenities_root
        type: column
        width: 375
        padding: "component-owned"
        gap: "component-owned"
        background: background-primary
        children:
          - id: location_amenities_block
            use: Location_and_map_row
          - id: nearby_amenities_block
            use: Nearby_Amenities_row

  # ---------------------------------------------------------------------------------------------
  feature_params_block:
    screen: primary_ad_detail
    section: feature_params_container
    requirement: REQ-01
    verifies: [AC-9]
    action: >
      UPDATE. Keep the section node and everything around it; replace ALL of its contents with
      the tree below. The tree IS the finished contents.
    what: >
      The existing `Đặc điểm bất động sản` params table with 3 rows visible and a "Xem thêm ▾"
      expander. Reused verbatim from Secondary Ad Detail.
    figma: "19027:50459"
    reference_width: 375
    maths: "component-owned"
    computed:
      feature_params_root: "375 × 216"       # measured; expanded height depends on data
    tree:
      - id: feature_params_root
        type: column
        width: 375
        padding: "component-owned"
        background: background-primary
        children:
          - id: feature_params
            use: Feature_params_section

  # ---------------------------------------------------------------------------------------------
  description_block:
    screen: primary_ad_detail
    section: description_container
    requirement: REQ-01
    verifies: [AC-9]
    action: >
      UPDATE. Keep the section node and everything around it; replace ALL of its contents with
      the tree below. The tree IS the finished contents.
    what: >
      The existing `Mô tả chi tiết` block with a body paragraph and a "Xem thêm" collapse.
      Reused verbatim.
    figma: "19027:50507"
    reference_width: 375
    maths: "component-owned"
    computed:
      description_root: "375 × 287"
    tree:
      - id: description_root
        type: column
        width: 375
        padding: "component-owned"
        background: background-primary
        children:
          - id: description
            use: Description_section

  # ---------------------------------------------------------------------------------------------
  other_ads_by_dev_block:
    screen: primary_ad_detail
    section: other_ads_by_dev_section
    requirement: REQ-01
    verifies: [AC-7, AC-10, AC-11]
    action: >
      NEW. This section does not exist on Secondary Ad Detail. Build the contents inside the
      new section container per the tree below. The tree IS the finished contents.
    what: >
      A white block, 375 wide with 16-horizontal padding, 12 top and 16 bottom. Inside, a
      column with gap 12: a section heading `"Các tin NOXH khác của CĐT"` in
      `display-annotation` bold on the left, then a horizontally scrolling row of ad cards
      (156W each, gap between cards is 12) that reveals ~2 cards plus a peek of a third. Each
      card is a compact ad tile 156W × 280H — an aspect-1:1 photo on top with a top-right
      heart badge and a bottom overlay showing a `"2 giờ trước"` time-ago and a photo count
      like `"6"` + camera icon; below the photo, 4-of-space padding + 8 of vertical padding
      hold two lines — a two-line ad title `"NOXH Happy Home Nhơn Trạch - Mã căn NĐ121"`, a
      small param line `"3 PN · Hướng Tây Nam"` (dot separator), a price row baseline-aligned
      `"1,7 tỷ" · "35 tr/m²" · "81 m²"` (red bold price, medium grey unit and area) and a
      location row with a 16 location icon and `"H. Nhơn Trạch"`.
    figma: "19027:50508"
    reference_width: 375
    maths: "16 padx + fluid horiz scrollable content + (12 padt + heading 26 + 12 gap + card 280 + 16 padb) = column height 346"
    computed:
      other_ads_by_dev_root: "375 × 346"
      section_heading_row: "343 × 26"
      section_heading_text: "flex × 26"
      scroll_row: "375 × 280"       # visible band; content scrolls beyond
      card_root: "156 × 280"
      card_image: "156 × 156"
      card_image_meta_overlay: "156 × ~24"
      card_heart_badge: "40 × 40"     # hit area; icons 20 each
      card_product_info: "156 × 124"
      card_title: "148 × 40"
      card_param_row: "148 × 18"
      card_price_row: "148 × 24"
      card_location_row: "148 × 24"
    tree:
      - id: other_ads_by_dev_root
        type: column
        width: 375
        padding: { x: 16, top: 12, bottom: 16 }
        gap: 12
        background: background-primary
        children:
          - id: section_heading_row
            type: row
            width: fluid
            align: center
            children:
              - id: section_heading_text
                type: text
                width: fluid
                name: "Các tin NOXH khác của CĐT"
                typography: display-annotation
                colour: text-primary

          - id: scroll_row
            type: horizontal_scroll
            width: fluid                  # overflows the padded 343 — scrolls
            gap: 12
            items:
              use: other_ads_by_dev_card
              source: data
              name: "{other_ads_by_dev_cards[]}"
              per_item_tree:
                # One card, replicated per data item. This tree IS what one card renders.
                - id: card_root
                  type: column
                  width: 156
                  radius: 6
                  background: background-primary
                  children:
                    - id: card_image
                      type: stack                          # container has overlays
                      width: 156
                      height: 156                          # aspect 1:1
                      radius: 6
                      overflow: clip
                      children:
                        - id: card_photo
                          type: image
                          source: data
                          name: "{card.thumbnail}"
                          fill: cover
                          absolute: { top: 0, left: 0, right: 0, bottom: 0 }

                        - id: card_image_meta_overlay
                          type: row
                          absolute: { bottom: 0, left: 0, right: 0 }
                          padding: { x: 10, bottom: 8 }
                          justify: space-between
                          align: end
                          background_gradient: "top:transparent → bottom:#000000 α0.75"  # kept-raw, see 8_report
                          children:
                            - id: card_time_ago
                              type: text
                              source: data
                              name: "{card.time_ago}"           # per CR-08
                              typography: label-annotation
                              colour: text-blank
                            - id: card_media_count_row
                              type: row
                              gap: 2
                              align: center
                              children:
                                - id: card_media_count
                                  type: text
                                  source: data
                                  name: "{card.media_count}"     # per CR-06 not applicable; bare integer
                                  typography: label-annotation
                                  colour: text-blank
                                - id: card_media_icon
                                  type: icon
                                  name: "image-fill (existing DS icon, 12×12)"
                                  size: 12

                        - id: card_heart_badge
                          type: stack                            # 20×20 fill + 20×20 outline stacked in 40×40 hit area
                          width: 40
                          height: 40
                          absolute: { top: 0, right: 0 }
                          shadow: "drop-shadow(0 4 4 rgba(89,89,89,0.15))"  # NB: matches DS `Shadow below` token — see 8_report
                          children:
                            - id: card_heart_fill
                              type: icon
                              source: data
                              name: "{card.is_liked ? 'heart-fill' : hidden}"
                              size: 20
                            - id: card_heart_outline
                              type: icon
                              source: data
                              name: "{card.is_liked ? hidden : 'heart-outline'}"
                              size: 20
                          behavior: "toggle card.is_liked (like/unlike this ad). Existing DS handler."

                    - id: card_product_info
                      type: column
                      width: fluid
                      padding: { x: 4, y: 8 }
                      gap: 4
                      children:
                        - id: card_title
                          type: text
                          source: data
                          name: "{card.title}"
                          typography: body-section
                          colour: text-primary
                          height: 40                         # two lines max
                          overflow: ellipsis
                          max_lines: 2
                        - id: card_param_row
                          type: row
                          gap: 4
                          align: center
                          overflow: clip
                          children:
                            - id: card_param_bedrooms
                              type: text
                              source: data
                              name: "{card.param_bedrooms}"
                              typography: body-caption
                              colour: text-tertiary
                              overflow: ellipsis
                              max_lines: 1
                            - id: card_param_dot
                              type: icon
                              name: "dot-divider (existing DS asset, 2×2)"
                              size: 2
                            - id: card_param_orientation
                              type: text
                              source: data
                              name: "{card.param_orientation}"
                              typography: body-caption
                              colour: text-tertiary
                              overflow: ellipsis
                              max_lines: 1
                        - id: card_price_row
                          type: row
                          gap: 12
                          align: baseline
                          overflow: nowrap
                          children:
                            - id: card_price
                              type: text
                              source: data
                              name: "{card.price}"
                              typography: label-page
                              colour: text-error
                            - id: card_price_per_m2
                              type: text
                              source: data
                              name: "{card.price_per_m2}"
                              typography: tagline-annotation
                              colour: text-primary
                            - id: card_area
                              type: text
                              source: data
                              name: "{card.area}"
                              typography: tagline-annotation
                              colour: text-primary
                        - id: card_location_row
                          type: row
                          gap: 4
                          align: center
                          height: 24
                          children:
                            - id: card_location_icon
                              type: icon
                              name: "Location-fill (existing DS icon, 16×16)"
                              size: 16
                            - id: card_location_text
                              type: text
                              source: data
                              name: "{card.location}"
                              typography: body-caption
                              colour: text-tertiary
                              overflow: ellipsis
                              max_lines: 1
                  behavior: BH-06

  # ---------------------------------------------------------------------------------------------
  other_projects_block:
    screen: primary_ad_detail
    section: other_projects_section
    requirement: REQ-01
    verifies: [AC-7, AC-10, AC-11]
    action: >
      NEW. This section does not exist on Secondary Ad Detail. Build the contents inside the new
      section container per the tree below. The tree IS the finished contents.
    what: >
      A white block, 375 wide with 16-horizontal padding, 12 top and 16 bottom. Inside, a column
      with gap 12: a heading row with `"Các dự án NOXH khác"` in `display-annotation` bold on
      the left and a compact bordered pill `"Xem tất cả"` on the right (24H, label-caption,
      padding 8/2, radius 999); then a horizontally scrolling row of project cards (156W each,
      gap 12) revealing ~2 cards + peek. Each card is a project tile 156W × 246H — a 156×156
      photo with a status chip absolutely-positioned top-left (`"Đang nhận hồ sơ"` on green
      background, or `"Sắp nhận hồ sơ {dd/mm}"` on grey), a bottom-fading dark gradient carrying
      a `"6"` + camera stat and a stat icon; below the photo a centred column with 8-top/8-bottom
      padding and 12 horizontal padding renders a two-line project title `"NOXH Bình Chánh - Lô B"`
      (`header-caption` semibold) and a price range `"Từ 890 triệu - 1,5 tỷ"` (`header-caption`
      semibold red), then a location row with a 16 location icon and a longer address like
      `"Quận Bình Thạnh P. Gia Định mới"`.
    figma: "19027:50513"
    reference_width: 375
    maths: "16 padx + (12 padt + heading 24 + 12 gap + card 246 + 16 padb) = column height 310"
    computed:
      other_projects_root: "375 × 310"
      section_heading_row: "343 × 24"
      xem_tat_ca_pill: "auto × 24"
      scroll_row: "375 × 246"
      project_card_root: "156 × 246"
      project_card_image: "156 × 156"
      project_card_status_chip: "auto × 16 (padding 4)"
      project_card_sub_info: "156 × 28"
      project_card_product_info: "156 × ~74"
      project_card_title: "132 × 40"
      project_card_price: "132 × 20"
      project_card_location: "132 × 18"
    tree:
      - id: other_projects_root
        type: column
        width: 375
        padding: { x: 16, top: 12, bottom: 16 }
        gap: 12
        background: background-primary
        children:
          - id: section_heading_row
            type: row
            width: fluid
            align: center
            gap: 8
            children:
              - id: section_heading_text
                type: text
                width: fluid
                name: "Các dự án NOXH khác"
                typography: display-annotation
                colour: text-primary
              - id: xem_tat_ca_pill
                use: Button_pill_small
                params:
                  size: x-small
                  label: "Xem tất cả"
                  behavior: BH-07

          - id: scroll_row
            type: horizontal_scroll
            width: fluid
            gap: 12
            items:
              use: other_projects_card
              source: data
              name: "{other_projects_cards[]}"
              per_item_tree:
                - id: project_card_root
                  type: column
                  width: 156
                  radius: 12
                  overflow: clip
                  align: center
                  children:
                    - id: project_card_image
                      type: stack
                      width: 156
                      height: 156
                      radius: 12
                      overflow: clip
                      children:
                        - id: project_card_photo
                          type: image
                          source: data
                          name: "{card.thumbnail}"
                          fill: cover
                          absolute: { top: 0, bottom: 0 }
                          alignment: "horizontal-center"        # design has `-translate-x-1/2 left-1/2` + aspect 114/114
                          radius: 12
                        - id: project_card_status_chip
                          type: row
                          absolute: { top: 4, left: 8 }
                          padding: { x: 4 }
                          radius: 4
                          background: "{card.project_status_chip_tokens.bg}"   # see CR-04
                          gap: 4
                          align: center
                          children:
                            - id: project_card_status_text
                              type: text
                              source: data
                              name: "{card.project_status_chip_text}"
                              typography: label-annotation
                              colour: "{card.project_status_chip_tokens.text}"
                        - id: project_card_sub_info
                          type: row
                          absolute: { bottom: 0, left: 0, right: 0 }
                          height: 28
                          justify: center
                          align: end
                          children:
                            - id: project_card_sub_info_container
                              type: row
                              width: fluid
                              padding: { x: 8, bottom: 8 }
                              gap: 4
                              align: center
                              justify: end
                              background_gradient: "top:#22222200 → bottom:#22222280 (kept raw — see 8_report)"
                              radius: { bl: 6, br: 6 }
                              children:
                                - id: project_card_stat_container
                                  type: row
                                  gap: 2
                                  align: center
                                  radius: 2
                                  children:
                                    - id: project_card_stat_media_count
                                      type: text
                                      source: data
                                      name: "{card.media_count}"
                                      typography: label-annotation
                                      colour: text-blank
                                    - id: project_card_stat_icon_1
                                      type: icon
                                      name: "photo-stat-icon (existing DS asset, 12×12)"
                                      size: 12
                                - id: project_card_stat_icon_2
                                  type: icon
                                  name: "extra-media-flag icon (existing DS asset, 12×12)"
                                  size: 12
                                  source: data
                                  name: "{card.extra_media_flag_icon_name}"

                    - id: project_card_product_info
                      type: column
                      width: fluid
                      padding: { x: 12, top: 8, bottom: 8 }
                      gap: 4
                      align: center
                      children:
                        - id: project_card_content
                          type: column
                          width: fluid
                          gap: 2
                          align: center
                          children:
                            - id: project_card_title
                              type: text
                              source: data
                              name: "{card.title}"
                              typography: header-caption
                              colour: text-primary
                              height: 40                        # two lines
                              overflow: ellipsis
                              max_lines: 2
                            - id: project_card_price
                              type: text
                              source: data
                              name: "{card.price_range}"          # per CR-07
                              typography: header-caption
                              colour: text-error
                              max_lines: 1
                              overflow: ellipsis
                        - id: project_card_location_row
                          type: row
                          width: fluid
                          gap: 4
                          align: center
                          children:
                            - id: project_card_location_icon
                              type: icon
                              name: "Location-fill (existing DS icon, 16×16)"
                              size: 16
                            - id: project_card_location_text
                              type: text
                              source: data
                              name: "{card.location}"
                              typography: body-caption
                              colour: text-tertiary
                              max_lines: 2
                              overflow: ellipsis
                  behavior: BH-07

  # ---------------------------------------------------------------------------------------------
  sticky_footer_variant_a:
    screen: primary_ad_detail
    section: sticky_footer_section
    requirement: REQ-01
    verifies: [AC-1, AC-5, AC-10]
    action: >
      UPDATE. Keep the section node and everything around it; replace ALL of its contents with
      the tree below. The tree IS the finished contents — whatever renders today and is not in
      the tree does not survive the replace.
    what: >
      A 375-wide sticky bar at the bottom of the viewport. The top half is a 375W row 8-vertical
      padded, 16-horizontal padded, containing a compact `"Quan tâm"` button (137W × 40H, tonal
      grey, radius 8, icon-left bell + label in `label-page` bold text-primary) beside a flex-1
      `"Tư vấn hồ sơ"` button (rest of the width × 40H, `button-primary` orange, icon-left, label
      in `label-page` bold white). Beneath the buttons sits the iOS Home Indicator space (34H
      white background with a 134×5 rounded-corner primary-icon bar centred at the bottom-8).
    figma: "19027:50594"
    reference_width: 375
    maths: "(padding 8t + button 40 + padding 8b) + home indicator 34 = 90"
    computed:
      sticky_footer_root: "375 × 90"
      button_group_row: "375 × 56"
      quan_tam_button: "137 × 40"
      tu_van_button: "flex × 40"
      home_indicator: "375 × 34"
      home_indicator_bar: "134 × 5"
    tree:
      - id: sticky_footer_root
        type: column
        width: 375
        gap: 0
        background: background-primary
        children:

          - id: button_group_row
            type: row
            width: 375
            padding: { x: 16, y: 8 }
            gap: 4
            align: center
            background: background-primary
            children:

              - id: quan_tam_button
                use: Button_tonal_neutral_lg
                params:
                  width: 137                                       # only the `Chưa quan tâm` state — see AC-5 for `Đã quan tâm` 56W
                  height: 40
                  icon_left: "bell-outline (existing DS icon, 24×24)"
                  label: "Quan tâm"
                  label_colour: text-primary
                  behavior: BH-01

              - id: tu_van_button
                use: Button_primary_lg
                params:
                  width: fluid                                     # flex 1 0 0
                  height: 40
                  icon_left: "envelope-or-send (existing DS icon, 24×24 — confirm with designer)"
                  label: "Tư vấn hồ sơ"
                  label_colour: text-on-background
                  behavior: BH-02

          - id: home_indicator
            type: stack
            width: fluid
            height: 34
            background: background-primary
            children:
              - id: home_indicator_bar
                type: rule
                width: 134
                height: 5
                colour: icon-primary
                radius: 100
                absolute: { bottom: 8, horizontal_center: true }

  # ---------------------------------------------------------------------------------------------
  sticky_footer_variant_b:
    screen: primary_ad_detail
    section: sticky_footer_section
    requirement: REQ-01
    verifies: [AC-3, AC-5, AC-10]
    action: >
      UPDATE. Keep the section node and everything around it; replace ALL of its contents with
      the tree below. The tree IS the finished contents — whatever renders today and is not in
      the tree does not survive the replace.
    what: >
      Identical to variant A, but the right button carries the disabled "Đã gửi tư vấn" state
      — still tonal grey, `text-tertiary` label, and does not trigger the apply flow when
      tapped. The left "Quan tâm" button behaves identically to variant A.
    figma: "19027:50594 (state Property1=Đã gửi tư vấn, `19027:48120`)"
    reference_width: 375
    maths: "(padding 8t + button 40 + padding 8b) + home indicator 34 = 90"
    computed:
      sticky_footer_root: "375 × 90"
      button_group_row: "375 × 56"
      quan_tam_button: "137 × 40"
      da_gui_tu_van_button: "flex × 40"
      home_indicator: "375 × 34"
      home_indicator_bar: "134 × 5"
    tree:
      - id: sticky_footer_root
        type: column
        width: 375
        gap: 0
        background: background-primary
        children:
          - id: button_group_row
            type: row
            width: 375
            padding: { x: 16, y: 8 }
            gap: 4
            align: center
            background: background-primary
            children:
              - id: quan_tam_button
                use: Button_tonal_neutral_lg
                params:
                  width: 137
                  height: 40
                  icon_left: "bell-outline (existing DS icon, 24×24)"
                  label: "Quan tâm"
                  label_colour: text-primary
                  behavior: BH-01
              - id: da_gui_tu_van_button
                use: Button_tonal_neutral_lg
                params:
                  width: fluid
                  height: 40
                  icon_left: "check-outline (existing DS icon, 24×24 — confirm)"
                  label: "Đã gửi tư vấn"
                  label_colour: text-tertiary
                  disabled: true
                  behavior: BH-03
          - id: home_indicator
            type: stack
            width: fluid
            height: 34
            background: background-primary
            children:
              - id: home_indicator_bar
                type: rule
                width: 134
                height: 5
                colour: icon-primary
                radius: 100
                absolute: { bottom: 8, horizontal_center: true }

  # ---------------------------------------------------------------------------------------------
  sticky_footer_variant_c:
    screen: primary_ad_detail
    section: sticky_footer_section
    requirement: REQ-01
    verifies: [AC-4, AC-10]
    action: >
      UPDATE. Keep the section node and everything around it; replace ALL of its contents with
      the tree below. The tree IS the finished contents — whatever renders today and is not in
      the tree does not survive the replace.
    what: >
      A single full-width tonal-grey button `"Xem dự án khác"` (`Button_tonal_neutral_lg`) fills
      the 375 minus 32-horizontal padding × 40H area. No `"Quan tâm"` button in this variant.
      Home indicator space unchanged.
    figma: "19027:50594 (state Property1=Đóng hồ sơ, `19027:48113`)"
    reference_width: 375
    maths: "(padding 8t + button 40 + padding 8b) + home indicator 34 = 90"
    computed:
      sticky_footer_root: "375 × 90"
      button_group_row: "375 × 56"
      xem_du_an_khac_button: "343 × 40"          # 375 − 32 padding
      home_indicator: "375 × 34"
      home_indicator_bar: "134 × 5"
    tree:
      - id: sticky_footer_root
        type: column
        width: 375
        gap: 0
        background: background-primary
        children:
          - id: button_group_row
            type: row
            width: 375
            padding: { x: 16, y: 8 }
            align: center
            background: background-primary
            children:
              - id: xem_du_an_khac_button
                use: Button_tonal_neutral_lg
                params:
                  width: fluid                                # takes the whole 343
                  height: 40
                  label: "Xem dự án khác"
                  label_colour: text-primary
                  behavior: BH-04
          - id: home_indicator
            type: stack
            width: fluid
            height: 34
            background: background-primary
            children:
              - id: home_indicator_bar
                type: rule
                width: 134
                height: 5
                colour: icon-primary
                radius: 100
                absolute: { bottom: 8, horizontal_center: true }

# =====================================================================================
Verify checklist:

0_right_section:                                                    # AC-9
  - Before editing header_container: the located node contains a display-annotation title of ~52H beside a "Lưu" bookmark pill. If it contains a "Xem lịch sử giá" link or a "Đăng bởi" avatar, you are inside a Secondary Ad Detail — stop.
  - Before editing sticky_footer_section: the located node is the ONLY position-fixed bottom bar on the screen, height 90 total. If it contains a "Chat" or "Gọi" button, you are on Secondary — stop.
  - The near-miss sibling for every section is unchanged after your edit. Diff it — see AC-9 for the exact list of sections that must not appear in the built page.

1_padding_ownership:                                                # AC-6, all `padding_check`
  - `header_container` and both new sections have exactly ONE surface each — a doubled inner padding means the surface was built twice.
  - A slot gap of exactly double (24 where 12 is designed) means a margin was added on top of the scroll's gap — remove the margin, do not halve the gap.

2_variant_isolation:                                                # AC-1, AC-3, AC-4
  - Rendering `sticky_footer_variant_a` produces zero nodes whose ids appear only in `_variant_b` or `_variant_c`.
  - Rendering `_variant_c` produces zero nodes whose ids appear only in `_variant_a` (no `Quan tâm` button, no `Tư vấn hồ sơ` button).

3_computed:                                                         # every `computed:` block
  - A container whose measured W and H are swapped relative to `computed` was built on the wrong axis — re-read its `type`.
  - The sticky footer's total height is 90 (56 + 34). If it is 56, the home indicator was skipped; if it is 124, an extra safe-area was added on top of the home indicator.

4_responsive:                                                       # AC-10
  - The screen root is 375 on the reference frame but must be fluid to the container in production. Every child that is not the screen root uses fluid width or a container-relative width.
  - Truncation triggers: on a narrow (320W) simulator the ad title still line-clamps to 2 lines; the "Dự án: {name}" line line-clamps to 1 line with an ellipsis after the linked span; the address on a project card line-clamps to 2 lines.

5_tokens_and_literals:                                              # AC-11 + Tag definitions numeric-literals paragraph
  - Every bare number in the tree was built as written — a value that landed on a token instead is a defect: report the node, the spec value and the value built.
  - Two raw hex values are intentional and must survive: the black photo-carousel background (see AC-11) and the 0-to-0.75 rgba gradient on card image overlays. Everything else that renders as a raw hex is a defect — map it to a token.

6_components_and_data:                                              # every `source: data`
  - Every leaf marked `source: data` resolves at runtime through the source named in `Reusable.data_bound` — a DS name hardcoded where the tree said `source: data` is a frozen sample.
  - The project-status chip inherits its background and text tokens from CR-04 — a chip rendered with a fixed green when the state is `Sắp nhận hồ sơ` is a hardcoded sample.

7_content_and_behavior:                                             # AC-8, AC-12, behaviors
  - `Tư vấn hồ sơ` on variant A opens the apply flow with `project_id` in the payload — NOT `ad_id`. Verify the network call.
  - The apply confirmation view (out of this spec) must NOT contain the string `Mã căn` — the message must not imply the specific apartment is reserved.
  - Every behavior in `behaviors` fires from its bound `behavior:` in the tree. A quiet button (no observable side-effect) is a defect.

8_report:
  - Measured inner padding for `header_container`, both new sections, and the sticky footer.
  - Measured slot gaps between screen sections (should all be 0 — the scroll owns them; every non-zero gap here is a duplicated margin).
  - Measured root W×H for every block; compare to `computed` and report deltas.
  - Every token name that did not resolve, and what value replaced it.
  - Every raw hex present in the built page except the two documented exceptions.
  - Every value off the DS scale, flagged for the designer to confirm intentional. Known values in this spec: none (all spacings are on the DS scale: 2/4/6/8/12/16; all radii are DS: 4/6/8/12/999).
  - Every `source: data` property, and how it resolves — the data path and the mapping name.
  - The breakpoint you tested at.
  - Anything in Part 1 that Part 2 contradicted.

# End of spec.
```
