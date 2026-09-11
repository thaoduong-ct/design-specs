# [PTY][NOXH] Project Detail — Developer Handoff Spec

Feature: NOXH Project Detail page (Mobile App only).
Source PRD: `noxh-hub-detail-prd.md` (User Story 4) via Jira BSVP-154.
Source Figma: `figma.com/design/UvLAgVP7Em1fwytMbuKHuv` → section `19027:49657` "Project Detail".
Spec version 1 · 2026-09-10 · produced by `handoff-design-spec` v2.0.

## How to use

This file is the **single source of truth for what to build**, meant to be read alongside the PRD and without opening Figma.

- **Part 1 — Requirement** is prose: what to build, when, why, and the acceptance criteria. Behaviors and content rules carry ids (`BH-nn`, `CR-nn`) referenced from Part 2.
- **Part 2 — Layout** is YAML. Read in this order for any single change:
  - An AC sends you to a layout block via `verifies: [AC-n]`.
  - `screen` tells you the page.
  - `section` sends you to `Sections` to locate the node in the codebase.
  - `action` says what to do to that node.
  - `what` describes the finished frame in reading order (with the Vietnamese labels a grep can hit).
  - the `tree` **is** the finished contents — anything on-screen today that is not in the tree does not survive an `update`.

**Every number in this spec is a literal.** A `gap: 2` is `2px`, not "close to 4". Tokens are used only where the token *name* is written (`gap-small-12`). There is no rounding rule.

**Fully static Vietnamese labels are greppable** (`"Địa chỉ bất động sản"`, `"Chủ đầu tư"`, `"Các dự án NOXH khác"`, `"Xem tất cả"`, `"Xem thêm"`, `"Tư vấn hồ sơ"`, `"Quan tâm"`, `"SỐ CĂN"`, `"NĂM BÀN GIAO"`, `"DIỆN TÍCH"`, `"Đánh giá từ cư dân"`, `"Tiện ích xung quanh"`, `"Tổng quan Phường {ward_name}"`). Anything with `{placeholder}` is not directly greppable — grep only the static half (e.g. `hữu ích` for `"{n} hữu ích"`, `Tổng quan Phường` for the ward overview heading).

**If the repo localises copy**, do not grep for the Vietnamese literal — grep for the i18n key. Where you have to guess a key or a file path, write the real one back into the spec's `path_hint` in your PR.

## Tag definitions

- **Numbers are literals.** Every numeric value in this spec is the exact pixel value to implement. A token is used ONLY where the spec writes the token's name (`gap-small-12`, `padding-medium-16`). Rounding `gap: 4` up or down to a nearby token is a defect — report it, don't perform it. The only conversion allowed is one this spec explicitly names.
- **`type` on a container** is one of `row` (HORIZONTAL flex), `column` (VERTICAL flex), `stack` (absolutely-positioned overlay). Every container declares `type` — no default.
- **Leaf defaults** (a leaf declares only what deviates):
  - text color → `text-primary` unless stated
  - alignment → `center` on the cross axis unless stated
  - a leaf with no `source:` key is FIXED. `source: data` marks a leaf whose value comes from the API; every such leaf has a `data_bound` entry.
- **Axis verbs in prose** — never compose elements with `+`, `,` or `then` in `what`, `context` or an AC. Use `stacked above`, `beneath`, `in a row with`, `to the right of`, `overlapping by N`. Reading order is not an axis.
- **Reference width** is the Figma frame width (375 for App). Inner padding is a separate number.
- **`action: update`** means: keep the located section node and everything around it; replace ALL of its contents with the tree below. The tree IS the finished contents — whatever renders today and is not in the tree does not survive the replace.

## Open items

```yaml
blocking: []

before_qc:
  OI-1:
    issue: >
      Full enum values for the two status badges (construction_status, application_status) are not
      confirmed. PRD says "backend-configurable, e.g. 'Đã hoàn thành', 'Đang nhận hồ sơ'". Design
      only shows 3 combos plus one inferred sold-out state. The two-independent-enums model was
      confirmed in Q4; the value tables need PM + backend sign-off before build.
    owner: pm + backend
    until_then: >
      Build with construction_status ∈ {"Đã hoàn thành", "Đang thi công"} and
      application_status ∈ {"Đang nhận hồ sơ", "Sắp nhận hồ sơ {open_date}", "Đã đóng"}. Colour
      map per data_bound.status_badge. Wire the enums as string enums with a default "unknown"
      branch that hides the badge — do not render an empty pill.
  OI-2:
    issue: >
      Sold-out CTA state (application closed) is not drawn in the design. PRD §6.2 says the
      "Tư vấn hồ sơ" CTA renders disabled when the project is fully sold out; the "Quan tâm"
      action beside it is out of scope for this ticket per §4.2.
    owner: design
    until_then: >
      Use DS `Button-group` component variant `Property 1 = "Đóng hồ sơ"` (componentSet 16048:18933).
      Label text stays "Tư vấn hồ sơ"; DS handles the disabled visual. Confirm the variant name
      with design during QA — the exact string was read from the Figma component set.
  OI-3:
    issue: >
      Loading, error, and not-found states are described in PRD §5.1 but not drawn in Figma.
    owner: design + pm
    until_then: >
      Loading = full-page skeleton (blocks matching each section's approximate height). Error =
      retry state with "Không thể tải dự án. Thử lại" and a retry button. Not-found = "Dự án không
      tồn tại hoặc đã bị gỡ" with a "Về NOXH Hub" button. Do NOT render any section partial or
      stale under any of these three states. Confirm the exact copy with PM before ship.
  OI-4:
    issue: >
      "Điều kiện mua" / "Điều kiện vay" tabs — PRD §4.1 requires them as Phase 2 placeholders;
      design does NOT draw them. Q2 confirmed they must be built.
    owner: pm + design
    until_then: >
      Build the two placeholder tabs per the `eligibility_placeholder` section below. Tab body
      renders a single centred line "Sắp ra mắt" with the info icon (`icon-info` colour). No tab
      state persists; both tabs render the same placeholder body. Confirm final copy and whether
      the placeholder should be hidden entirely if the feature-flag disables Phase 2.
  OI-5:
    issue: >
      Data source for the horizontal unit cards inside the Characteristic component (`Đặc điểm
      bất động sản` section, below the 3 floor plan rows) — Q3 confirmed these are agent ad
      listings keyed by `project_id`, but the API and query shape are not specified anywhere.
    owner: dev + backend
    until_then: >
      Use the existing "ad listings by project_id" query (typically the same endpoint the NOXH
      Hub "Tin đăng" tab is planned to use — see PRD User Story 3 which is out of scope but shares
      the source). If no such endpoint exists yet, this becomes a blocking backend prerequisite.
  OI-6:
    issue: >
      Value-to-icon mapping for the 3 amenity category icons in the `Tổng quan Phường` row is not
      stated. The design shows 3 fixed icons (icon leaves `12042:7293`, `12042:7296`, `12042:7298`)
      but PRD implies these should reflect the actual top-3 nearby amenity categories.
    owner: pm + design
    until_then: >
      Render the 3 icons as fixed DS icons per the design (frozen sample). Flag in `8_report` if
      the API returns a category-typed set of icons at build time — if so, promote to a data-bound
      leaf and reuse the existing amenity-category-to-icon mapping in the Nearby Amenities module.

  OI-10:
    issue: >
      More-vertical icon menu (top-right of the floating header) — the design draws the
      icon but no menu is shown, and the PRD is silent on what tapping it does. Common
      choices: report the project, share via a specific channel, "Về NOXH Hub".
    owner: pm + design
    until_then: >
      Wire BH-03 to a no-op with `console.warn('more menu not defined')`. Do not hide
      the icon (removing it changes the header layout). Resolve before ship.

cosmetic:
  OI-7:
    issue: >
      Designer inconsistency: variant frames `19027:49833` and `19027:49875` use DS component set
      `project status` (16048:17486) for the right badge; variant frame `19027:49917` uses plain
      `Text Label` with `Color=Blue`. The main-frame status header (`19027:49665`) also uses plain
      `Text Label` for both badges. Q4 flattens all of this into "2 independent enums, both
      rendered as Text Label with Color driven by data" so the spec is consistent.
    owner: design
  OI-8:
    issue: >
      Card title in Related Projects section (`Card Item Grid > Product Info > Title > TEXT`)
      renders at raw `#000000`, not bound to `text-primary` (#222222). Drift. Spec maps it to
      `text-primary` per Phase 3b context rule; report in `8_report`.
    owner: design
  OI-9:
    issue: >
      The `#F2F6FC` (light-blue) Text Label background is not present in this file's Variable
      defs — DS may own it inside the Blue variant, or it may be a raw hex. Spec references it
      via the DS component's `Color=Blue` variant so dev never writes the hex directly.
    owner: design
```

## PART 1 — Requirement

```yaml
REQ-01:
  title: NOXH Project Detail page (Mobile App)
  context: >
    A buyer taps a project card from the NOXH Hub "Dự án" tab (that tab is out of scope for
    this ticket — PRD §4.2) and lands on this page. It is a project-level view sourced
    entirely from the developer, sufficient to decide whether to press "Tư vấn hồ sơ" and
    register for consultation. There is no standalone entry point until the Hub "Dự án"
    tab (a separate ticket) ships.

    The page renders top to bottom: photo gallery with a "1/N" indicator, a floating header
    (back + share + more) overlaying the photo, then a scrollable content column with the
    project status header (two badges + name + price "Từ {min} - {max}" + developer +
    3 key stats "SỐ CĂN"/"NĂM BÀN GIAO"/"DIỆN TÍCH"), the combined address-and-overview
    section headed "Địa chỉ bất động sản" and "Tổng quan Phường {ward_name}", the
    "Đặc điểm bất động sản" characteristics section (a DS component the developer uses
    as-is), an eligibility placeholder tabs section ("Điều kiện mua" / "Điều kiện vay"
    stubs for Phase 2 — see [OI-4]), a "Mô tả chi tiết" description-section (DS component
    with an "Xem thêm" expand), and a "Các dự án NOXH khác" horizontal scroller of related
    NOXH project cards. A sticky bottom "Tư vấn hồ sơ" primary CTA is visible through
    every scroll position.

  variant_a_open_completed: >
    Construction status "Đã hoàn thành" (Green) with application status "Đang nhận hồ sơ"
    (Green). Sticky CTA active. See AC-1.
  variant_b_open_in_progress: >
    Construction status "Đang thi công" (Blue) with application status "Đang nhận hồ sơ"
    (Green). Sticky CTA active. See AC-2.
  variant_c_upcoming: >
    Construction status "Đang thi công" (Blue) with application status
    "Sắp nhận hồ sơ {open_date}" (Blue), where {open_date} is a data-bound Vietnamese short
    date. Sticky CTA active. See AC-3.
  variant_d_sold_out: >
    Construction status renders per data (typically "Đã hoàn thành") with application status
    "Đã đóng" (Neutral). Sticky CTA disabled — DS Button-group variant "Đóng hồ sơ". Label
    text stays "Tư vấn hồ sơ"; DS supplies the disabled visual. See AC-4 and [OI-2].
  control: >
    While the project data is in flight → skeleton per [OI-3]. If the fetch fails → retry
    error state per [OI-3]. If the project_id no longer resolves → not-found state per
    [OI-3]. No partial or stale section renders under any of these three states.

  render_when: >
    Route: /pty/noxh/du-an/{project_id} (path is a convention; confirm at build). The page
    resolves the variant ONCE at the top of the render, from the response fields
    `construction_status` and `application_status` — never per row. Sold-out is a fourth
    variant computed as `application_status == "Đã đóng"`; all four variants use the same
    scroll layout but differ in `project_status_header` and `sticky_bottom_cta`.

  rules: [CR-01, CR-02, CR-03, CR-04, CR-05, CR-06]
  behaviors: [BH-01, BH-02, BH-03, BH-04, BH-05, BH-06, BH-07]
  acceptance_criteria:
    AC-1: >
      Rendering with `construction_status = "Đã hoàn thành"` and
      `application_status = "Đang nhận hồ sơ"` produces the `status_header_variant_a` tree
      exactly, with the left badge Green and the right badge Green, followed by the rest of
      the page in order.
    AC-2: >
      Rendering with `construction_status = "Đang thi công"` and
      `application_status = "Đang nhận hồ sơ"` produces the `status_header_variant_b` tree
      exactly, with the left badge Blue and the right badge Green.
    AC-3: >
      Rendering with `construction_status = "Đang thi công"` and
      `application_status = "Sắp nhận hồ sơ"` produces the `status_header_variant_c` tree
      exactly, with both badges Blue, and the right badge text is
      `"Sắp nhận hồ sơ {open_date}"` where `{open_date}` is the API's `open_date` value
      formatted as Vietnamese short date `d/M` (e.g. `15/7`) per CR-05.
    AC-4: >
      Rendering with `application_status = "Đã đóng"` produces the `status_header_variant_d`
      tree (right badge Neutral, text `"Đã đóng"`) and the `sticky_cta_sold_out` block
      (DS Button-group variant `Property 1 = "Đóng hồ sơ"`; label stays `"Tư vấn hồ sơ"`).
    AC-5: >
      All four status variants are mutually exclusive at render time. Rendering never
      produces a tree that contains node ids appearing only in another variant's block.
    AC-6: >
      The page renders every section listed in `Screens.project_detail.renders`, in that
      order, with no other sections and no reordering.
    AC-7: >
      The "Địa chỉ bất động sản" and "Tổng quan Phường {ward_name}" sub-sections live in
      one combined `address_and_overview` section — a single card with `background-primary`
      surface, one radius, one inner padding. There is no separate second card.
    AC-8: >
      The `characteristics` section renders as a single DS instance of
      `Characteristic` (componentId `16048:17961`) with no props. The developer does not
      rebuild the site map, floor plan rows or horizontal unit cards inside — they come
      from the DS component.
    AC-9: >
      The horizontal unit cards inside `Characteristic` are agent ad listings filtered
      by `project_id = current_project.id` (see [OI-5] for the query shape). The dev
      does not hardcode a sample list.
    AC-10: >
      The `eligibility_placeholder` section renders two tabs "Điều kiện mua" and "Điều
      kiện vay". Tapping either tab does not trigger navigation and does not perform any
      eligibility or loan check. The tab body renders `"Sắp ra mắt"` centred with the
      `icon-info` icon. See [OI-4].
    AC-11: >
      `related_projects` section renders only OTHER NOXH projects (not agent ad listings),
      via the existing platform related-projects logic. Query parameter must include the
      NOXH vertical filter. Tapping a card navigates to that project's Project Detail page
      (recursion — same route with a different project_id) per BH-06.
    AC-12: >
      The sticky bottom CTA sits at the bottom of the viewport with `background-primary`
      surface, is visible through every scroll position, and does not push page content up
      when the keyboard is not open. The Home Indicator (`19027:49822`) is iOS chrome and
      is not part of the build.
    AC-13: >
      Tapping "Tư vấn hồ sơ" (when not disabled) triggers the apply flow bound to
      `project_id`, never to a specific unit — the confirmation and any subsequent screens
      must not imply a specific unit is reserved. See BH-05 and [PRD §6.2 AC-EC-02].
    AC-14: >
      No agent-side UI affordance appears anywhere on the page — no "chỉnh sửa", no
      "báo cáo", no "liên hệ người đăng". This page is developer-sourced content only.
    AC-15: >
      Colour: every fill and text in the built page resolves to the token named in
      `Reusable.tokens` OR the raw hex listed in `Reusable.tokens` (currently: card title
      text is `text-primary` = `#222222`, not `#000000` — see [OI-8]). A raw hex that lands
      in the build without appearing in `Reusable.tokens` is a defect — report the node,
      the hex and where it was applied.
    AC-16: >
      Padding ownership: the located `<section_id>` node already carries its surface, its
      radius and its inner padding — `padding_check.surfaces = 1` for every section. The
      dev does not wrap it in a second padded container. A doubled inner padding means the
      surface was built twice.
    AC-17: >
      Vertical rhythm: the gap between two consecutive sections is exactly the value
      declared in `Screens.project_detail.slot_check.gap_between_sections` (8). It comes
      from the parent content column's `gap`, not from a margin on the section itself.
      A slot gap of 16 means someone added a section margin on top of the parent gap —
      remove the margin, not the parent gap.
    AC-18: >
      Responsive: the layout is mobile-first, fluid width — every container that carries
      content stretches to the parent width (with the section-side 8px outer padding
      preserved). No fixed pixel width on any container. Every text container includes
      `min-width: 0` so long strings truncate rather than push siblings. Design was drawn
      at 375; a viewport below 320 clips at the outer padding — do not add a horizontal
      scroll on the page body (only `related_projects.row` and the `Characteristic` unit
      cards scroll horizontally, inside their own `overflow-x: auto`).
    AC-19: >
      Truncation and wrapping — see CR-06.
    AC-20: >
      Behaviours: each interactive element listed in `behaviors` fires its original handler
      (or the newly-defined handler per that behavior). A tap on a `project status` badge
      does NOT navigate. A tap on the "Xem tất cả" button in `related_projects` navigates
      to the NOXH Hub "Dự án" tab (BH-04); this tab is out of scope for this ticket, so a
      stub route is acceptable per PRD §4.2 dependency.
    AC-21: >
      The `related_projects.row` container is `overflow-x: auto` with `scroll-snap` on each
      card (matching the platform pattern; check the current related-projects component
      first). The three visible cards in the design are a sample — the count is data-bound.

behaviors:
  BH-01:
    element: "Back button in floating header (24×24 Arrowleft-fill icon in a 32×32 white pill)"
    on_tap: "keep_as_is — navigate back one entry in the app router history"
  BH-02:
    element: "Share button in floating header (24×24 Share-outline icon in a 32×32 white pill)"
    on_tap: >
      keep_as_is — open the platform share sheet for the current project URL. If no existing
      share handler is bound on the project detail route, wire a new one that shares
      `{deep_link_to_project}` with title = current `project.name`.
  BH-03:
    element: "More button in floating header (24×24 Morevertical-outline icon in a 32×32 white pill)"
    on_tap: >
      Not stated in the PRD. Design shows the icon but not the resulting menu. Default:
      wire to a no-op with a `console.warn('more menu not defined')` and open [OI-10]
      immediately — do not hide the icon.
  BH-04:
    element: "'Xem tất cả' button beside 'Các dự án NOXH khác' heading (DS Button, Size=S, Type=Tertiary, isPill=Yes)"
    on_tap: >
      Navigate to NOXH Hub "Dự án" tab. Route TBD — that tab is out of scope for this
      ticket (PRD §4.2, User Story 2). Stub route is acceptable; leave the target URL as a
      TODO the Hub ticket resolves.
  BH-05:
    element: "'Tư vấn hồ sơ' sticky CTA (DS Button-group)"
    on_tap: >
      Active state: trigger the apply flow (see `noxh-buyer-actions-prd.md`, Phase 2 —
      out of scope here). Sold-out state: DS Button-group disabled variant, no-op on tap.
      The apply flow binds to `project_id`; never bind to a specific unit.
  BH-06:
    element: "A card inside `related_projects.row` (DS Card Item Grid at 156×274)"
    on_tap: >
      Navigate to `/pty/noxh/du-an/{tapped_card.project_id}` — the same route with a
      different `project_id`. This is intended recursion; the page unmounts and remounts.
  BH-07:
    element: "A tab in `eligibility_placeholder` ('Điều kiện mua' or 'Điều kiện vay')"
    on_tap: >
      Toggle the active tab visually. Both tabs render the same placeholder body
      (`"Sắp ra mắt"` with `icon-info`). No navigation, no data fetch, no eligibility or
      loan logic — this is a Phase 2 stub (see [OI-4]).

content_rules:
  CR-01:
    element: project_status_header.applicant_count
    rule: >
      "{applicant_count}+ người nộp" where `{applicant_count}` is an integer bucketed to
      the nearest 50 or 100 (backend decides; sample "200+"). The literal `"+"` and
      `"người nộp"` never localise. Do not render the row if the API returns 0 or null.
  CR-02:
    element: project_status_header.price_range
    rule: >
      "Từ {price_min} - {price_max}" — both values are Vietnamese-formatted currency
      strings ("890 triệu", "1,5 tỷ") produced by the existing money-formatting helper
      (do NOT format inline). The `"Từ"` and `" - "` separators are literals.
  CR-03:
    element: project_status_header.key_stats.{so_can|nam_ban_giao|dien_tich}
    rule: >
      Three columns, uppercased label + value below.
        - "SỐ CĂN" — value "{units_total} căn", integer with `.` thousands separator
        - "NĂM BÀN GIAO" — value "{handover_year}", 4-digit year, no separator
        - "DIỆN TÍCH" — value "{area_min}-{area_max} m2", integers only, m² rendered as
          "m2" per the design (not the superscript)
      If any of the 3 stats has null data, render "—" in place of the value; do not hide
      the label.
  CR-04:
    element: address_and_overview.address
    rule: >
      Two lines. Primary: "{street}, {ward}, {district}" (up to 2 visual lines,
      `line-clamp-2`). Secondary: "{ward_short}, {city}" (single line, `ellipsis-1`).
      Both come from the address payload verbatim — the spec does not compose them.
  CR-05:
    element: project_status_header.status_badges.right (application_status)
    rule: >
      When `application_status = "Sắp nhận hồ sơ"`, the badge text is
      "Sắp nhận hồ sơ {open_date}" where `{open_date}` is formatted as Vietnamese short
      date `d/M` — e.g. `15/7`, `2/12`. Do NOT zero-pad. `{open_date}` is a bare date
      string; the format helper strips the year. For other application_status values, the
      badge text is exactly the enum label (no placeholders).
  CR-06:
    element: page-wide text truncation
    rule: >
      - Project name (`project_status_header.name`): single line, `ellipsis-1`, tail
        truncation.
      - Address primary line: `line-clamp-2`, tail truncation.
      - Address secondary line: single line, `ellipsis-1`.
      - Section headings ("Địa chỉ bất động sản", "Tổng quan Phường {ward_name}",
        "Các dự án NOXH khác"): single line, `ellipsis-1`.
      - Description body inside DS `Description-section`: DS-owned (its `Expand`
        variant controls collapse). The dev sets `Expand = False` on first render; DS
        toggles on "Xem thêm" tap.
      - Related project card title: `line-clamp-2` (design: 132×40 with two lines).
      - Related project card location: `line-clamp-2` (design: 112×36 with two lines).
      - "Tư vấn hồ sơ" CTA label: never wraps.
```

## PART 2 — Layout

### Reusable

```yaml
ds_components:
  text_label:
    name: "Text Label"
    componentSet: "16048:16070"
    componentIds_used:
      green: "16048:16097"     # bg background-success-light (#ECF9F1)
      blue: "16048:16095"      # bg #F2F6FC — see [OI-9]
      neutral: "16048:16093"   # bg background-secondary (#F4F4F4)
    props:
      Icon swaps: "'7835:12497' (default icon; hidden when Icon=No)"
      Input Text: "the badge text — data-bound per data_bound.status_badge"
      Color: "Green | Blue | Neutral — data-bound per data_bound.status_badge"
      Size: "10"
      Icon: "No"
      Hightlighted: "False"
    resolved_style:
      height: 16
      padding: "4px horizontal, 0 vertical"
      radius: "radius-ad-small (4)"
      typography: "label-annotation (Reddit Sans Bold 10/16)"
      text_color:
        green: "text-success (#12A154)"
        blue: "text-info — see [OI-9]"
        neutral: "text-primary (#222222)"

  project_status:
    name: "project status"
    componentSet: "16048:17485"
    variants:
      Status: "'open for applying' | 'almost' | 'close'"
      Device: "'Mobile' | 'Desktop' (typo in file: 'esktop')"
    note: >
      Design library exposes this DS set but the actual frames use plain `Text Label`
      for both badges — see [OI-7]. This spec builds all badges as `Text Label`
      instances to avoid the inconsistency; the `project status` set is documented here
      for future migration.

  button_tertiary_s:
    name: "Button"
    componentSet: "16048:14114"
    componentIds_used:
      xem_tat_ca: "16048:14744"
    props:
      Size: "S - 24px"
      Type: "🥉  Tertiary"
      State: "Active"
      isPill: "Yes"
      Icon Left#896:98: "False (no icon)"
      Input Text: "'Xem tất cả' (fixed)"

  button_group:
    name: "Button-group"
    componentSet: "16048:18933"
    variants:
      Property 1: "'Đóng hồ sơ' | 'Chưa đóng hồ sơ' | 'Variant3'"
    resolved_style:
      height: "button-height-large (40) plus internal padding — total row 56 tall per design"
      primary_button_color: "button-primary (#FA6819)"
      surface: "background-primary (#FFFFFF)"
    note: >
      "Chưa đóng hồ sơ" is the active/open state; "Đóng hồ sơ" is the disabled/sold-out
      state (see [OI-2]). "Variant3" is unused by this feature — do not select it.

  description_section:
    name: "Description-section"
    componentSet: "1274:24347"
    variants:
      Platform: "'Mobile' | 'Web'"
      Expand: "'True' | 'False'"
    props:
      # DS-owned internals; instance only sets variants
    note: >
      Renders the "Mô tả chi tiết" heading, the body copy, and the "Xem thêm" expand
      toggle. Dev sets `Platform = "Mobile"`, `Expand = "False"` on mount. DS handles the
      toggle; the toggle is not a behaviour listed here.

  characteristic:
    name: "Characteristic"
    componentId: "16048:17961"
    componentSet: null
    props: {}   # no variant props
    note: >
      Standalone project-specific DS component that renders the whole "Đặc điểm bất động
      sản" section internally (heading, site map image, three floor plan rows, and the
      horizontal unit cards below). Dev uses as-is per Q3; do not descend into it. The
      cards inside are agent ad listings by `project_id` — see [OI-5].

  card_item_grid:
    name: "Card Item Grid"   # local component built in-line — see Layout blocks
    props: {}
    note: >
      Not a shared DS instance in this file — each card is manually laid out. The
      layout is repeated in `related_projects_layout`; do not parameterise.

  arrowleft_fill:
    name: "Arrowleft-fill"
    componentId: "12042:7436"
    size: 24
    note: "Back-arrow icon."

  share_outline:
    name: "Share-outline"
    componentId: "10219:4964"
    size: 24

  morevertical_outline:
    name: "Morevertical-outline"
    componentId: "75:7277"
    size: 24

  icon_map_thumb_tertiary:
    name: "Icon (Tertiary/M-32px)"
    componentId: "16048:16804"
    props: { Icon: "'75:7584' (map/location icon)", Size: "M - 32px", Type: "🥉 Tertiary", State: "Active", isPill: "Yes" }
    note: "The 32×32 icon chip overlaid on the map thumbnail in address_and_overview."

  star_fill:
    name: "Star-fill"
    componentId: "10219:6114"
    size: 12
    note: "The 5 star icons in the Tổng quan Phường rating."

  home_indicator:
    name: "Bars/Home Indicator/iPhone/Light - Portrait"
    componentId: "26:6970"
    note: "iOS chrome — NOT part of the build. Excluded from the tree."

  status_bar:
    name: "Bars / Status Bar / iPhone / Light"
    componentId: "104:9825"
    note: "iOS chrome — NOT part of the build. Excluded from the tree."

data_bound:
  status_badge:
    binds: [Color, Input Text]
    source: >
      construction_status and application_status from the project detail API.
      construction_status ∈ enum (values TBC per [OI-1]).
      application_status ∈ {"Đang nhận hồ sơ", "Sắp nhận hồ sơ", "Đã đóng"} — see [OI-1].
    mapping: |
      construction_status → (Color, label):
        "Đã hoàn thành" → ("Green", "Đã hoàn thành")
        "Đang thi công" → ("Blue", "Đang thi công")
        (unknown)       → hide the badge
      application_status → (Color, label):
        "Đang nhận hồ sơ"   → ("Green",   "Đang nhận hồ sơ")
        "Sắp nhận hồ sơ"    → ("Blue",    "Sắp nhận hồ sơ {open_date}")   # CR-05
        "Đã đóng"           → ("Neutral", "Đã đóng")
        (unknown)            → hide the badge
    sample_data:
      construction_status: "Đã hoàn thành"
      application_status: "Đang nhận hồ sơ"
      open_date: "15/7"

  project_name:
    binds: text
    source: "project.name"
    sample_data: "NOXH Happy Home Nhơn Trạch"

  price_range:
    binds: text
    source: "project.price_min and project.price_max (money-formatted helper — CR-02)"
    sample_data: "Từ 890 triệu - 1,5 tỷ"

  developer_name:
    binds: text
    source: "project.developer.name"
    sample_data: "Công ty Cổ phần Kim Long Nam"

  developer_verified_badge:
    binds: icon_visible
    source: "project.developer.verified (boolean)"
    mapping: "true → render the blue verified checkmark; false → omit the icon"
    sample_data: true

  applicant_count:
    binds: text
    source: "project.applicant_count (integer)"
    mapping: "CR-01: '{n}+ người nộp'; hide the row if n == 0 or null"
    sample_data: 200

  key_stats:
    binds: text (3 values)
    source: "project.units_total, project.handover_year, project.area_min + area_max"
    mapping: "CR-03"
    sample_data: { units_total: 1200, handover_year: 2026, area_min: 45, area_max: 60 }

  photo_gallery:
    binds: images
    source: "project.photos[]"
    mapping: >
      Render as a horizontal swipable gallery. Show first photo full-bleed. The "{i}/{N}"
      indicator (photo_gallery.count_pill) reflects current index and total.
    sample_data: ["<photo1>", "<photo2>", "<photo3>", "<photo4>", "<photo5>"]

  photo_count:
    binds: text
    source: "current gallery index and total (derived)"
    mapping: '"{current_index}/{total_photos}"'
    sample_data: "1/5"

  ward_name:
    binds: text
    source: "project.address.ward"
    mapping: '"Tổng quan Phường {ward_name}"'
    sample_data: "Thạnh Xuân"

  primary_address:
    binds: text
    source: "project.address.primary (composed by API: street, ward, district)"
    sample_data: "Đường Lê Quang Hòa, Phường Thạnh Xuân, Quận 12"

  secondary_address:
    binds: text
    source: "project.address.secondary (composed by API: ward_short, city)"
    sample_data: "P. Thạnh Xuân, TP Hồ Chí Minh mới"

  map_thumbnail:
    binds: image
    source: "static map image URL for project.location"
    mapping: "72×72 rounded thumbnail with a tertiary map icon overlay (icon_map_thumb_tertiary)"

  overview_rating:
    binds: [text, star_count]
    source: "project.ward_overview.rating (float, 1 decimal) and star_count (int 1..5)"
    mapping: "rating text 3.9; stars filled = round(rating) using DS Star-fill component"
    sample_data: { rating: 3.9, filled_stars: 4 }

  overview_reviews_count:
    binds: text
    source: "project.ward_overview.reviews_count (int)"
    mapping: '"{n} lượt"'
    sample_data: 21

  overview_amenities_count:
    binds: text
    source: "project.ward_overview.amenities_count (int, bucketed)"
    mapping: '"{n}+"'
    sample_data: 84

  overview_amenity_icons:
    binds: icons (3)
    source: >
      project.ward_overview.top_amenity_categories[0..2] — see [OI-6]. Currently the spec
      treats these as fixed sample icons per the design; promote to data-bound if the API
      returns them.
    mapping: "reuse the amenity-category-to-icon map in the Nearby Amenities module"
    sample_data: ["12042:7293 (school)", "12042:7296 (hospital)", "12042:7298 (restaurant)"]

  related_projects_list:
    binds: card_list
    source: >
      Existing platform "related NOXH projects" endpoint. Query MUST include
      vertical=NOXH and exclude agent ad listings; return only other NOXH projects.
    mapping: "each returned project becomes one Card Item Grid; count is dynamic"
    sample_data:
      - { name: "NOXH Bình Chánh - Lô B", photo_count: 6, price_range: "Từ 890 triệu - 1,5 tỷ", location: "Quận Bình Thạnh, P. Gia Định mới", application_status_badge: "Đang nhận hồ sơ" }
      - { name: "NOXH Bình Chánh - Lô B", photo_count: 6, price_range: "Từ 890 triệu - 1,5 tỷ", location: "Quận Bình Thạnh, P. Gia Định mới", application_status_badge: "Đang nhận hồ sơ" }
      - { name: "NOXH Bình Chánh - Lô B", photo_count: 6, price_range: "Từ 890 triệu - 1,5 tỷ", location: "Quận Bình Thạnh, P. Gia Định mới", application_status_badge: "Sắp nhận hồ sơ 15/7" }

  characteristic_project_id:
    binds: project_id (component prop)
    source: "current project.id — passed into the Characteristic DS component so its internal unit-listings query filters by this id"
    note: "Confirm the DS component actually accepts project_id as a prop — see [OI-5]."

tokens:
  # Resolved from Figma variables endpoint via get_variable_defs (Phase 4).
  # Every fill and text in the built tree resolves to a token below OR a raw hex listed here.
  background-primary:          "#FFFFFF"    # card and page background
  background-secondary:        "#F4F4F4"    # Text Label neutral bg; small icon chip bg
  background-app:              "#F7F7F7"    # divider surface in overview row
  background-success-light:    "#ECF9F1"    # Text Label green bg
  background-overlay:          "#22222280"  # photo indicator pill bg (semi-transparent)
  text-primary:                "#222222"    # main body text, headings, price digits
  text-secondary:              "#595959"    # unused in this build
  text-tertiary:               "#8C8C8C"    # "CHỦ ĐẦU TƯ", "SỐ CĂN", "NĂM BÀN GIAO", "DIỆN TÍCH", secondary address, related-project location
  text-blank:                  "#FFFFFF"    # "1/5" indicator text
  text-error:                  "#F0325E"    # price text and related-project price text
  text-success:                "#12A154"    # Text Label Green text
  icon-primary:                "#222222"    # back / share / more icons
  icon-tertiary:               "#8C8C8C"    # unused in this build
  icon-blank:                  "#FFFFFF"    # icon inside overlay pill
  icon-info:                   "#306BD9"    # verified checkmark on developer row; placeholder icon on eligibility tab
  icon-chotot:                 "#FFBA00"    # unused in this build
  icon-disabled:               "#C0C0C0"    # DS-owned disabled state; not authored here
  border-thin:                 "#E8E8E8"    # unused as an explicit stroke; DS-owned
  border-regular:              "#DDDDDD"    # unused in this build
  button-blank:                "#FFFFFF"    # DS-owned button surface
  button-primary:              "#FA6819"    # DS Button-group primary bg (via Button-group)
  button-tonal-neutral:        "#F4F4F4"    # unused in this build
  radius-pill:                 999
  radius-ad-small:             4            # Text Label badge radius
  radius-ad:                   6            # small chips
  radius-card-small:           8            # cards inside related-projects row
  radius-card:                 12           # sections' outer card radius (address_and_overview, related_projects, project_status_header)
  stroke-divider:              1            # 1px divider width
  stroke-action:               2            # DS-owned
  gap-min-2:                   2
  gap-2x-small-4:              4
  gap-2x-small-6:              6
  gap-x-small-8:               8
  gap-small-12:                12
  gap-medium-16:               16
  padding-min-2:               2
  padding-2x-small-4:          4
  padding-x-small-8:           8
  padding-small-12:            12
  padding-medium-16:           16
  button-height-large:         40
  # Raw hexes kept as-is (not resolved to a token — see 8_report):
  raw-black-000000:            "#000000"    # ONLY on related-project card title in Figma; spec maps it to text-primary per [OI-8]. Report if it appears in the build.
  raw-blue-f2f6fc:             "#F2F6FC"    # DS-owned Text Label Blue bg — never write directly; use Color=Blue on the DS component

typography:
  # Resolved from Figma. Family: Reddit Sans. weight/size/line-height are literals.
  label-annotation:      { family: "Reddit Sans", weight: 700, size: 10, line_height: 16 }   # badge text
  body-annotation:       { family: "Reddit Sans", weight: 400, size: 10, line_height: 16 }   # "200+ người nộp"; small labels
  body-caption:          { family: "Reddit Sans", weight: 400, size: 12, line_height: 18 }   # "Đánh giá từ cư dân", "Tiện ích xung quanh", related-project location
  label-caption:         { family: "Reddit Sans", weight: 700, size: 12, line_height: 18 }   # key-stat values, developer name
  header-caption:        { family: "Reddit Sans", weight: 600, size: 14, line_height: 20 }   # related-project card title
  label-section:         { family: "Reddit Sans", weight: 700, size: 14, line_height: 20 }   # "Đăng ký"-style; unused here directly
  body-section:          { family: "Reddit Sans", weight: 400, size: 14, line_height: 20 }   # address primary/secondary; "Tổng quan Phường"; body text
  header-section:        { family: "Reddit Sans", weight: 600, size: 16, line_height: 24 }   # DS heading (Description-section)
  label-page:            { family: "Reddit Sans", weight: 700, size: 16, line_height: 24 }   # "Địa chỉ bất động sản"
  display-annotation:    { family: "Reddit Sans", weight: 700, size: 18, line_height: 26 }   # project name, price, "Các dự án NOXH khác"
  display-caption:       { family: "Reddit Sans", weight: 700, size: 20, line_height: 28 }   # "3.9", "21 lượt", "84+"
  photo-pill-14:         { family: "Reddit Sans", weight: 700, size: 14, line_height: 20 }   # "1/5" — matches header-caption but weight 700

icons:
  arrowleft_fill:        { name: "Arrowleft-fill",       component_id: "12042:7436" }
  share_outline:         { name: "Share-outline",        component_id: "10219:4964" }
  morevertical_outline:  { name: "Morevertical-outline", component_id: "75:7277"    }
  location_fill:         { name: "Location-fill",        component_id: "104:8126"   }
  star_fill:             { name: "Star-fill",            component_id: "10219:6114" }
  map_icon:              { name: "map/location icon",    component_id: "75:7584"    }
  amenity_school:        { name: "amenity category (school)",     component_id: "12042:7293" }
  amenity_hospital:      { name: "amenity category (hospital)",   component_id: "12042:7296" }
  amenity_restaurant:    { name: "amenity category (restaurant)", component_id: "12042:7298" }

sample_data:
  # Consolidated for the human reader. Not what gets built.
  project:
    id: "noxh-happy-home-nhon-trach"
    name: "NOXH Happy Home Nhơn Trạch"
    photos: ["<p1>", "<p2>", "<p3>", "<p4>", "<p5>"]
    price_min: "890 triệu"
    price_max: "1,5 tỷ"
    developer: { name: "Công ty Cổ phần Kim Long Nam", verified: true }
    units_total: 1200
    handover_year: 2026
    area_min: 45
    area_max: 60
    address: { primary: "Đường Lê Quang Hòa, Phường Thạnh Xuân, Quận 12", ward: "Thạnh Xuân", secondary: "P. Thạnh Xuân, TP Hồ Chí Minh mới" }
    applicant_count: 200
    construction_status: "Đã hoàn thành"
    application_status: "Đang nhận hồ sơ"
    ward_overview:
      rating: 3.9
      reviews_count: 21
      amenities_count: 84
      top_amenity_categories: ["school", "hospital", "restaurant"]
```

### Screens

```yaml
project_detail:
  renders: [
    photo_gallery_layout,
    top_header_layout,
    status_header_variant_a,        # actual variant chosen at render — see AC-5
    address_and_overview_layout,
    characteristics_layout,
    eligibility_placeholder_layout,
    description_layout,
    related_projects_layout,
    sticky_cta_active                # actual variant chosen at render
  ]
  place: |
    app_shell (existing route container, full viewport)
      status_bar (iOS chrome, not built)
      ★ project_detail_page          ← this whole spec builds this
    (no siblings above or below)
  parent_provides: []               # full-page route, no external gaps
  untouched: []                     # no siblings in scope
  page_background: background-app   # #F7F7F7
  slot_check:
    gap_between_sections: 8         # gap in the content column between consecutive scroll sections
    gap_between_photo_and_content: 0   # content column stacks directly under the photo
    header_position: "fixed top overlay on photo, y=0, height=92"
    sticky_position: "fixed bottom, height=90 (56 button-group + 34 home indicator; home indicator is not built)"
```

### Sections

```yaml
photo_gallery:
  acted_on_by: [photo_gallery_layout]
  figma: { project_detail: "19027:49659 'Image group'" }
  path_hint: { path: unknown, provenance: unknown, fill_on_first_build: true }
  find_by: >
    The photo gallery at the top of a Project Detail page — grep for the route
    handler that mounts a "gallery" component beneath the project detail header.
    The rendered "1/{N}" pill uses the greppable string ` / ` inside a text node
    with the character "/" and a totals count.
  confirm: >
    375×375 square image container with a floating "N/M" pill overlay in the top-right,
    OVER a project photo (not a listing photo). The pill's background is
    background-overlay (#22222280). If the pill sits on the LEFT side of the photo,
    or the container is a wide (>375) hero, you are in another feature — stop and re-locate.
  current: >
    Not yet built. This is a new page. If a "Project Detail" route stub exists, place
    this gallery as the first section under the route's layout, above the header row.
    Owns the surface (full-bleed photo) and the pill's inner padding; no card wrapper.
  owns:
    this_section_provides: [full-bleed photo container 375×375, absolute-positioned pill overlay]
    therefore: "Never wrap the photo in a padded card. The pill is absolutely placed inside the photo container."
  padding_check: { inner_x: 0, inner_y: 0, surfaces: 1 }

top_header:
  acted_on_by: [top_header_layout]
  figma: { project_detail: "19027:49823 'Header'" }
  path_hint: { path: unknown, provenance: unknown, fill_on_first_build: true }
  find_by: >
    The floating header on top of the Project Detail photo — a horizontal row with a
    back arrow on the left and two icon-buttons on the right (share, more), each in
    a 32×32 white pill. It shares no unique text string; anchor on structure.
  confirm: >
    Three white 32×32 pill-shaped icon buttons floating over the photo, back-arrow at
    left, share and more at right in that order. If the more-vertical icon is missing
    or the header sits on a solid background (not floating over an image), you are in
    another page's header — stop.
  current: >
    Not yet built. Place ABOVE the photo gallery in DOM order but visually z-indexed
    on top of it (position: fixed, top: 0, safe-area padding handled by the app shell).
  owns:
    this_section_provides: [absolute-positioned header row, white pill button surfaces]
    therefore: "No page background surface here. The photo is what shows behind."
  padding_check: { inner_x: 16, inner_y: 0, surfaces: 1 }

project_status_header:
  acted_on_by:
    - status_header_variant_a
    - status_header_variant_b
    - status_header_variant_c
    - status_header_variant_d_sold_out
  figma:
    project_detail:
      main: "19027:49665 'Frame 151089792' (rendering variant A in the master frame)"
      variant_a: "19027:49833"
      variant_b: "19027:49875"
      variant_c: "19027:49917"
      variant_d: "(not drawn — inferred per [OI-1] + [OI-2])"
  path_hint: { path: unknown, provenance: unknown, fill_on_first_build: true }
  find_by: >
    A white card directly under the photo gallery, containing (top-to-bottom): a row
    of two small pill badges followed by "200+ người nộp"; the project name (bold 18);
    the price line starting with "Từ" in red; a "CHỦ ĐẦU TƯ" label with the developer
    name and a verified checkmark; and a three-column stats row with headers
    "SỐ CĂN", "NĂM BÀN GIAO", "DIỆN TÍCH".
  confirm: >
    The "CHỦ ĐẦU TƯ" label appears in this section (not in the address section below).
    The three stat headers are ALL UPPERCASE and appear side by side in one row.
    If you see the stats stacked vertically or the developer row inside a card that
    also has a map thumbnail, you are in the wrong section — stop.
  current: >
    Not yet built. This section is the second scroll section below the photo. Owns
    the card surface, radius, inner padding.
  owns:
    this_section_provides: [white card surface, radius-card (12), inner padding 16/12, all child content]
    therefore: "Never wrap in a second padded card. The dev builds the card ONCE."
  padding_check: { inner_x: 16, inner_y: 12, surfaces: 1 }

address_and_overview:
  acted_on_by: [address_and_overview_layout]
  figma: { project_detail: "19027:49708 'Frame 2085668299' (wraps 19027:49709 'Location')" }
  path_hint: { path: unknown, provenance: unknown, fill_on_first_build: true }
  find_by: >
    A white card containing two headings: "Địa chỉ bất động sản" (bold 16) with an
    address block and a 72×72 map thumbnail to its right, followed BELOW by
    "Tổng quan Phường {ward_name}" (regular 14) with a three-column row showing a
    rating (e.g. "3.9"), a reviews count ("21 lượt", "Đánh giá từ cư dân"), and an
    amenity count ("84+" with three small overlapping icons, "Tiện ích xung quanh").
  confirm: >
    Both "Địa chỉ bất động sản" and "Tổng quan Phường {ward_name}" are inside ONE
    card, one below the other, sharing one radius, one inner padding, ONE background.
    If they render as two separate cards (a gap of background-app between them),
    that is the wrong shape — re-check `owns.this_section_provides`.
  current: >
    Not yet built. Per user Q2, keep both blocks combined into one card. This differs
    from the PRD list which named "Localities" and "Amenities" as two sections.
  owns:
    this_section_provides: [white card surface, radius-card (12), inner padding 16/12, dividers between overview stats]
    therefore: "The two headings share the card. Do not build two cards. Do not add a divider between the two blocks."
  padding_check: { inner_x: 16, inner_y: 12, surfaces: 1 }

characteristics:
  acted_on_by: [characteristics_layout]
  figma: { project_detail: "19027:49747 'Characteristic' (INSTANCE, componentId 16048:17961)" }
  path_hint: { path: unknown, provenance: unknown, fill_on_first_build: true }
  find_by: >
    A section headed "Đặc điểm bất động sản" (bold 18) containing a large site-map
    image, three floor-plan rows ("3 Phòng ngủ", "2 Phòng ngủ", "1 Phòng ngủ" with
    bed/bath/area rows), and a horizontal scroller of unit ad cards below.
  confirm: >
    The section renders "Đặc điểm bất động sản" as a heading. If you see "Địa chỉ" or
    "Mô tả chi tiết" as the heading, you are in the wrong section. The heading is the
    DS component's own — do not re-add it in the parent.
  current: >
    Not yet built. Use the DS `Characteristic` component as-is. Its interior is DS-owned;
    do NOT rebuild it. The horizontal unit cards inside come from an agent-ads-by-project
    query the DS component runs internally — pass `project_id` as a prop (see [OI-5]).
  owns:
    this_section_provides: [entire section rendering, including its own surface, padding, radius, and internals]
    therefore: "The developer instantiates the DS component and passes props only. No wrapper card, no manual heading."
  padding_check: { inner_x: 8, inner_y: 0, surfaces: 1 }   # outer wrapper adds 8px side padding only; the DS component owns the rest

eligibility_placeholder:
  acted_on_by: [eligibility_placeholder_layout]
  figma: { project_detail: "(not drawn — see [OI-4])" }
  path_hint: { path: unknown, provenance: unknown, fill_on_first_build: true }
  find_by: >
    NEW section — not yet in the codebase. Place it between the `characteristics`
    section and the `description` section. When built, anchor on the tab labels
    "Điều kiện mua" and "Điều kiện vay" (visible as tab headers).
  confirm: >
    Two side-by-side tabs "Điều kiện mua" and "Điều kiện vay" with an active-tab
    underline. Body renders "Sắp ra mắt" centred, with the icon-info icon above the
    text. Tapping a tab switches the underline but the body stays the same.
  current: >
    Does not exist. Introduced by this ticket per PRD §4.1 (Phase 2 placeholder — no
    eligibility or loan logic). Confirm the placement (between characteristics and
    description) if the design ships a mock later.
  owns:
    this_section_provides: [white card surface, radius-card (12), inner padding 16/16, tabs + body]
    therefore: "One card, one radius, one padding. Placeholder body is NOT a real component — it is a static line the dev writes."
  padding_check: { inner_x: 16, inner_y: 16, surfaces: 1 }

description:
  acted_on_by: [description_layout]
  figma: { project_detail: "19027:49748 'Description-section' (INSTANCE, componentId 1274:24348)" }
  path_hint: { path: unknown, provenance: unknown, fill_on_first_build: true }
  find_by: >
    A section headed "Mô tả chi tiết" (bold 16) with a paragraph of body text and a
    "Xem thêm" toggle at the bottom-left.
  confirm: >
    Heading is exactly "Mô tả chi tiết"; the body wraps with the DS Expand=False
    variant, so only ~3 lines show initially. Tapping "Xem thêm" reveals the rest.
    If the "Xem thêm" button is at bottom-right or the heading reads "Chi tiết mô
    tả", you are in another feature — stop.
  current: >
    Not yet built. Use DS `Description-section` instance with `Platform=Mobile`,
    `Expand=False`. Pass the body copy as a prop.
  owns:
    this_section_provides: [entire section rendering, DS component owns surface, padding, heading, body, expand toggle]
    therefore: "The developer places the DS instance and passes props. No wrapper."
  padding_check: { inner_x: 8, inner_y: 0, surfaces: 1 }

related_projects:
  acted_on_by: [related_projects_layout]
  figma: { project_detail: "19027:49749 'Frame 2085665968'" }
  path_hint: { path: unknown, provenance: unknown, fill_on_first_build: true }
  find_by: >
    A white card headed "Các dự án NOXH khác" (bold 18) with a "Xem tất cả" tertiary
    pill button on the right, followed by a horizontally-scrollable row of cards
    (156 wide, 274 tall each) that each show a project photo, a status badge, some
    stats, a project name, a price starting with "Từ", and a location line with a
    pin icon.
  confirm: >
    Heading is "Các dự án NOXH khác" (not "Tin đăng khác", not "Dự án tương tự").
    Each card is 156 wide — smaller than the section width — so at least 2 cards
    are visible side by side. Cards scroll horizontally within the section; the
    page body itself does not scroll horizontally.
  current: >
    Not yet built. Use the existing platform "related projects" logic, but filter
    to NOXH-only projects (never include agent-posted ad listings). Card layout is
    built inline per the tree — do not import an "Ad Card" component from elsewhere.
  owns:
    this_section_provides: [white card surface, radius-card (12), inner padding 16/12/16/16, heading row + horizontal scroller]
    therefore: "One outer card, one radius, one padding. Cards inside have their own radius (radius-card-small = 8) but do not add margin outside them."
  padding_check: { inner_x: 16, inner_y: 12, surfaces: 1 }

sticky_bottom_cta:
  acted_on_by: [sticky_cta_active, sticky_cta_sold_out]
  figma: { project_detail: "19027:49820 'bottom-button'" }
  path_hint: { path: unknown, provenance: unknown, fill_on_first_build: true }
  find_by: >
    The Button-group DS instance at the bottom of the viewport, containing (or being)
    a primary CTA "Tư vấn hồ sơ" that stays visible while the page scrolls. Anchor
    on the Vietnamese string "Tư vấn hồ sơ".
  confirm: >
    Sits at the bottom of the viewport, background-primary, includes a 40px-tall
    primary orange button whose label is exactly "Tư vấn hồ sơ". If the label reads
    "Đăng ký tư vấn" or "Liên hệ" you are in another feature — stop.
  current: >
    Not yet built. Use DS `Button-group` with `Property 1 = "Chưa đóng hồ sơ"` for
    the active state and `Property 1 = "Đóng hồ sơ"` for the sold-out state. The
    home indicator (19027:49822) below the button is iOS chrome, NOT part of the
    build (the app shell handles safe-area).
  owns:
    this_section_provides: [background-primary surface at the very bottom of the viewport, the DS Button-group instance]
    therefore: "The dev does not build a second surface behind the DS button. The DS Button-group already owns its internals."
  padding_check: { inner_x: 0, inner_y: 0, surfaces: 1 }
```

### Layout blocks

Read order: an AC → `screen` → `section` → `Sections` (locate) → `action` → `what` → `tree`. The tree is the finished contents. Any node not in the tree does not survive an `action: update`.

```yaml
# ============================================================================
# BLOCK — photo_gallery_layout
# ============================================================================
screen: project_detail
section: photo_gallery
requirement: REQ-01
verifies: [AC-6]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its contents
  with the tree below. The tree IS the finished contents — whatever renders today
  and is not in the tree does not survive the replace.
what: >
  A full-bleed 375×375 photo container. In the top-right corner, a dark pill
  overlay ("1/5") sits inset 16 from the right edge and 12 from the top of the
  photo. The pill has a dark semi-transparent background (`background-overlay`)
  with white bold text (`text-blank`). The pill is anchored (absolute) inside
  the photo container, not part of the page's normal flow.
figma: "19027:49659"
reference_width: 375
maths: "root height = 375 (photo is a square at the reference width)"
computed:
  photo_gallery_root: 375x375
  photo_image: 375x375
  count_pill_wrapper: 77x52
  count_pill: 45x28
tree:
  - id: photo_gallery_root
    type: stack
    figma: "19027:49659"
    children:
      - id: photo_image
        type: image
        source: data
        name: "{photo_gallery[current_index]}"
        size: [375, 375]
      - id: count_pill_wrapper
        type: stack
        align: [end, start]              # anchor to top-right
        offset: [inset-right: 16, inset-top: 12]
        children:
          - id: count_pill
            type: row
            gap: 10
            pad: [12, 4, 12, 4]
            bg: background-overlay
            radius: radius-pill
            children:
              - id: photo_count_text
                type: text
                source: data
                name: "{photo_count}"     # CR: "{current_index}/{total_photos}"
                typography: photo-pill-14
                color: text-blank

# ============================================================================
# BLOCK — top_header_layout
# ============================================================================
screen: project_detail
section: top_header
requirement: REQ-01
verifies: [AC-6, AC-20]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its contents
  with the tree below. The tree IS the finished contents — whatever renders today
  and is not in the tree does not survive the replace.
what: >
  A row that floats above the photo. The row is padded 16 on each side and has
  the back button (32×32 white pill with a dark back-arrow icon) on the left,
  and to the right a group of two 32×32 white pills — the share icon then the
  more-vertical icon, gap 8 between them.
figma: "19027:49823"
reference_width: 375
maths: "root height = 48 (nav row; excludes status bar 44 which is iOS chrome)"
computed:
  header_root: 375x48
  back_pill: 32x32
  right_group: 72x32
  share_pill: 32x32
  more_pill: 32x32
tree:
  - id: header_root
    type: row
    figma: "19027:49825"
    pad: [16, 0, 16, 0]
    justify: space-between            # gap 227 in Figma = distribute
    align: center
    children:
      - id: back_pill
        type: row
        pad: [4, 4, 4, 4]
        bg: background-primary
        radius: radius-pill
        behavior: BH-01
        children:
          - id: back_icon
            type: icon
            name: Arrowleft-fill
            size: 24
            color: icon-primary
      - id: right_group
        type: row
        gap: gap-x-small-8
        children:
          - id: share_pill
            type: row
            pad: [4, 4, 4, 4]
            bg: background-primary
            radius: radius-pill
            behavior: BH-02
            children:
              - id: share_icon
                type: icon
                name: Share-outline
                size: 24
                color: icon-primary
          - id: more_pill
            type: row
            pad: [4, 4, 4, 4]
            bg: background-primary
            radius: radius-pill
            behavior: BH-03
            children:
              - id: more_icon
                type: icon
                name: Morevertical-outline
                size: 24
                color: icon-primary

# ============================================================================
# BLOCK — status_header_variant_a  ("Đã hoàn thành" + "Đang nhận hồ sơ")
# ============================================================================
screen: project_detail
section: project_status_header
requirement: REQ-01
verifies: [AC-1, AC-5, AC-6, AC-15, AC-16, AC-17]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its contents
  with the tree below. The tree IS the finished contents — whatever renders today
  and is not in the tree does not survive the replace.
what: >
  A white card. Top row: two small pill badges (both Green) reading "Đã hoàn thành"
  and "Đang nhận hồ sơ" in a row with gap 4, followed to the right by a small avatar
  group and the caption "200+ người nộp". Below the badges row: the project name
  "NOXH Happy Home Nhơn Trạch" bold 18, then the price "Từ 890 triệu - 1,5 tỷ" bold
  18 in red. Below that: an uppercase label "CHỦ ĐẦU TƯ" in tertiary grey, and
  beneath it a row with a small verified checkmark icon and the developer name
  "Công ty Cổ phần Kim Long Nam". At the bottom: a three-column stats row —
  "SỐ CĂN" over "1.200 căn", "NĂM BÀN GIAO" over "2026", "DIỆN TÍCH" over "45-60 m2",
  each column 100 wide with gap 12 between them.
figma: "19027:49665 (main frame) · 19027:49833 (variant frame)"
reference_width: 375
maths: "12 + (16 + 4 + 94 + 8 + 34) + 12 = 184  |  badges row + price group (94: name 26 + price 26 + seller 34 + gaps 8) + stats 34, wrapped by 12/12 vertical padding"
computed:
  status_header_root: 375x184
  status_header_inner: 343x160
  badges_and_count_row: 343x16
  badges_row: 233x16
  badge_left_construction: 76x16
  badge_right_application: 85x16
  count_group: 106x16
  count_avatar_group: 32x16
  count_text: 72x16
  name_and_seller_column: 343x94
  project_name: 343x26
  price_text: 343x26
  seller_info_row: 343x34
  seller_label_ci: 343x16
  seller_row: 343x18
  key_stats_row: 343x34
  stat_column: 100x34
  stat_label: "~40x16"
  stat_value: "~55x18"
tree:
  - id: status_header_root
    type: column
    figma: "19027:49665"
    bg: background-primary
    radius: radius-card
    pad: [16, 12, 16, 12]
    gap: gap-small-12
    children:
      - id: status_header_inner
        type: column
        gap: gap-small-12
        children:
          - id: name_stats_group
            type: column
            gap: gap-2x-small-4
            children:
              - id: badges_and_count_row
                type: row
                gap: gap-2x-small-4
                children:
                  - id: badges_row
                    type: row
                    gap: gap-2x-small-4
                    children:
                      - id: badge_left_construction
                        type: instance
                        use: text_label
                        source: data
                        props:
                          Color: "Green"                # data_bound.status_badge → construction_status == "Đã hoàn thành"
                          Input Text: "Đã hoàn thành"
                          Size: "10"
                          Icon: "No"
                          Hightlighted: "False"
                      - id: badge_right_application
                        type: instance
                        use: text_label
                        source: data
                        props:
                          Color: "Green"                # data_bound.status_badge → application_status == "Đang nhận hồ sơ"
                          Input Text: "Đang nhận hồ sơ"
                          Size: "10"
                          Icon: "No"
                          Hightlighted: "False"
                  - id: count_group
                    type: row
                    gap: gap-min-2
                    children:
                      - id: count_avatar_group
                        type: stack                    # 3 overlapping avatars, ~32 total width, 16 tall
                        source: data
                        name: "{applicant_avatars_group}"
                        size: [32, 16]
                        note: "Small stack of 3 overlapping avatar rounds. Design has this as a Group; the dev may render 3 <img class='avatar' /> absolutely positioned with -8px overlap."
                      - id: count_text
                        type: text
                        source: data
                        name: "{applicant_count}+ người nộp"    # CR-01
                        typography: body-annotation
                        color: text-primary
              - id: name_and_seller_column
                type: column
                gap: gap-2x-small-4
                children:
                  - id: project_name
                    type: text
                    source: data
                    name: "{project_name}"
                    typography: display-annotation
                    color: text-primary
                    truncate: ellipsis-1
                  - id: price_text
                    type: text
                    source: data
                    name: "{price_range}"              # CR-02: "Từ {price_min} - {price_max}"
                    typography: display-annotation
                    color: text-error
                  - id: seller_info_row
                    type: column
                    children:
                      - id: seller_label_ci
                        type: text
                        name: "CHỦ ĐẦU TƯ"
                        typography: body-annotation
                        color: text-tertiary
                      - id: seller_row
                        type: row
                        gap: gap-2x-small-4
                        children:
                          - id: seller_verified_icon
                            type: icon
                            source: data
                            name: "{developer_verified_badge}"  # icon-info verified checkmark; hidden when false
                            size: 12
                            color: icon-info
                          - id: seller_name
                            type: text
                            source: data
                            name: "{developer_name}"
                            typography: label-caption
                            color: text-primary
          - id: key_stats_row
            type: row
            gap: gap-small-12
            children:
              - id: stat_column_units
                type: column
                gap: gap-min-2
                size: [100, 34]
                children:
                  - { id: stat_units_label, type: text, name: "SỐ CĂN", typography: body-annotation, color: text-tertiary }
                  - id: stat_units_value
                    type: text
                    source: data
                    name: "{units_total} căn"        # CR-03: "1.200 căn"
                    typography: label-caption
                    color: text-primary
              - id: stat_column_year
                type: column
                gap: gap-min-2
                size: [100, 34]
                children:
                  - { id: stat_year_label, type: text, name: "NĂM BÀN GIAO", typography: body-annotation, color: text-tertiary }
                  - id: stat_year_value
                    type: text
                    source: data
                    name: "{handover_year}"          # CR-03: "2026"
                    typography: label-caption
                    color: text-primary
              - id: stat_column_area
                type: column
                gap: gap-min-2
                size: [100, 34]
                children:
                  - { id: stat_area_label, type: text, name: "DIỆN TÍCH", typography: body-annotation, color: text-tertiary }
                  - id: stat_area_value
                    type: text
                    source: data
                    name: "{area_min}-{area_max} m2"  # CR-03: "45-60 m2"
                    typography: label-caption
                    color: text-primary

# ============================================================================
# BLOCK — status_header_variant_b  ("Đang thi công" + "Đang nhận hồ sơ")
# ============================================================================
screen: project_detail
section: project_status_header
requirement: REQ-01
verifies: [AC-2, AC-5]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its contents
  with the tree below. The tree IS the finished contents — whatever renders today
  and is not in the tree does not survive the replace.
what: >
  Identical to variant_a in every respect except the LEFT badge reads "Đang thi công"
  and its colour is Blue (`Text Label Color=Blue`), while the RIGHT badge stays Green
  with text "Đang nhận hồ sơ".
figma: "19027:49875"
reference_width: 375
maths: "identical to variant_a"
computed:
  status_header_root: 375x184
  status_header_inner: 343x160
  badge_left_construction: 72x16
  badge_right_application: 85x16
  (all other ids identical to variant_a)
tree:
  # Same tree as variant_a with two prop changes only:
  extends: status_header_variant_a
  overrides:
    - id: badge_left_construction
      props:
        Color: "Blue"
        Input Text: "Đang thi công"
    # badge_right_application stays as in variant_a
# NOTE: `extends` here is a shorthand for "reuse the parent tree, override only these
# nodes' props" — the dev implements the same tree, and the render-time enum decides
# which literal `props` land. AC-5 checks mutual exclusion. No shared parts library.

# ============================================================================
# BLOCK — status_header_variant_c  ("Đang thi công" + "Sắp nhận hồ sơ {open_date}")
# ============================================================================
screen: project_detail
section: project_status_header
requirement: REQ-01
verifies: [AC-3, AC-5]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its contents
  with the tree below. The tree IS the finished contents — whatever renders today
  and is not in the tree does not survive the replace.
what: >
  Identical to variant_a in structure. LEFT badge Blue text "Đang thi công".
  RIGHT badge Blue text "Sắp nhận hồ sơ 15/7" (where 15/7 is the data-bound
  `{open_date}` per CR-05).
figma: "19027:49917"
reference_width: 375
maths: "identical to variant_a"
computed:
  status_header_root: 375x184
  badge_left_construction: 72x16
  badge_right_application: 101x16
tree:
  extends: status_header_variant_a
  overrides:
    - id: badge_left_construction
      props:
        Color: "Blue"
        Input Text: "Đang thi công"
    - id: badge_right_application
      props:
        Color: "Blue"
        Input Text: "Sắp nhận hồ sơ {open_date}"     # CR-05

# ============================================================================
# BLOCK — status_header_variant_d_sold_out  (inferred; see [OI-1]/[OI-2])
# ============================================================================
screen: project_detail
section: project_status_header
requirement: REQ-01
verifies: [AC-4, AC-5]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its contents
  with the tree below. The tree IS the finished contents — whatever renders today
  and is not in the tree does not survive the replace.
what: >
  Identical to variant_a in structure. LEFT badge inherits from data
  (typically Green "Đã hoàn thành" when a project sold out after handover, but
  Blue "Đang thi công" is also valid). RIGHT badge is Neutral (Text Label
  Color=Neutral) with text "Đã đóng".
figma: "(not drawn — inferred)"
reference_width: 375
maths: "identical to variant_a"
computed:
  status_header_root: 375x184
tree:
  extends: status_header_variant_a
  overrides:
    - id: badge_left_construction
      props:
        Color: data                                   # inherits from construction_status
        Input Text: data                              # "{construction_status_label}"
    - id: badge_right_application
      props:
        Color: "Neutral"
        Input Text: "Đã đóng"

# ============================================================================
# BLOCK — address_and_overview_layout
# ============================================================================
screen: project_detail
section: address_and_overview
requirement: REQ-01
verifies: [AC-6, AC-7, AC-15, AC-16, AC-17, AC-19]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its contents
  with the tree below. The tree IS the finished contents — whatever renders today
  and is not in the tree does not survive the replace.
what: >
  A white card containing two blocks separated by a 16 gap. Block one has the
  heading "Địa chỉ bất động sản" (bold 16) above a row containing the address
  text (primary line up to 2 lines then a smaller secondary line, 4 gap between
  them) on the left, and a 72×72 map thumbnail with a rounded rectangle image
  and a small tertiary map icon centred on it, on the right. Block two has the
  heading "Tổng quan Phường Thạnh Xuân" (regular 14) above a row divided into
  three groups by 1-wide dividers: rating "3.9" bold 20 above five 12×12 stars;
  reviews "21 lượt" bold 20 above "Đánh giá từ cư dân" regular 12; amenities
  "84+" bold 20 in a row with three small icon chips (F4F4F4 bg) overlapping
  by 5 above "Tiện ích xung quanh" regular 12.
figma: "19027:49708"
reference_width: 375
maths: "12 + 104 + 16 + 76 + 12 = 220 (measured 216 with 4 rounding). Outer wrapper adds 8 side padding, no vertical. Inner card 359 wide, 216 tall."
computed:
  address_and_overview_wrapper: 375x216
  address_and_overview_card: 359x216
  address_title_block: 327x104
  address_heading: 327x24
  address_row: 327x72
  address_container: 247x64
  address_primary: 247x40
  address_secondary: 247x20
  map_thumb: 72x72
  map_icon_chip: 32x32
  overview_block: 327x76
  overview_inner: 327x76
  overview_heading: 195x20
  overview_row: 327x50
  rating_group: 64x50
  rating_text: 28x28
  stars: 64x12
  divider: 1x24
  reviews_group: 101x50
  reviews_count: 62x28
  reviews_label: 101x18
  amenities_group: 107x50
  amenities_count_row: 102x28
  amenities_count_text: 36x28
  amenities_icons_row: 62x24
  amenity_icon_chip: 24x24
  amenity_icon: 16x16
  amenities_label: 107x18
tree:
  - id: address_and_overview_wrapper
    type: column
    figma: "19027:49708"
    pad: [8, 0, 8, 0]                # outer 8px side padding to reach 375 from 359 card
    gap: gap-x-small-8               # ignored — single child
    children:
      - id: address_and_overview_card
        type: column
        figma: "19027:49709"
        bg: background-primary
        radius: radius-card
        pad: [16, 12, 16, 12]
        gap: gap-small-12
        children:
          - id: address_title_block
            type: column
            gap: gap-x-small-8
            children:
              - id: address_heading
                type: text
                name: "Địa chỉ bất động sản"
                typography: label-page
                color: text-primary
                truncate: ellipsis-1
              - id: address_row
                type: row
                gap: gap-x-small-8
                children:
                  - id: address_container
                    type: column
                    gap: gap-2x-small-4
                    children:
                      - id: address_primary
                        type: text
                        source: data
                        name: "{primary_address}"     # CR-04
                        typography: body-section
                        color: text-primary
                        truncate: line-clamp-2
                      - id: address_secondary
                        type: text
                        source: data
                        name: "{secondary_address}"   # CR-04
                        typography: body-section
                        color: text-tertiary
                        truncate: ellipsis-1
                  - id: map_thumb
                    type: stack
                    size: [72, 72]
                    children:
                      - id: map_thumb_image
                        type: image
                        source: data
                        name: "{map_thumbnail}"
                        size: [72, 72]
                        radius: radius-card-small
                      - id: map_icon_chip
                        type: instance
                        use: icon_map_thumb_tertiary
                        # DS instance; owns its 32×32 pill and the map icon inside
          - id: overview_block
            type: column
            gap: gap-medium-16
            children:
              - id: overview_inner
                type: column
                gap: gap-2x-small-6
                children:
                  - id: overview_heading
                    type: text
                    source: data
                    name: "Tổng quan Phường {ward_name}"
                    typography: body-section
                    color: text-primary
                  - id: overview_row
                    type: row
                    gap: gap-small-12
                    align: center
                    children:
                      - id: rating_group
                        type: column
                        gap: 7                          # off-DS-scale — see 8_report
                        align: start
                        children:
                          - id: rating_text
                            type: text
                            source: data
                            name: "{overview_rating}"   # sample "3.9"
                            typography: display-caption
                            color: text-primary
                          - id: stars
                            type: row
                            gap: 1                       # off-DS-scale — see 8_report
                            children:
                              - id: star_1
                                type: instance
                                use: star_fill
                              - id: star_2
                                type: instance
                                use: star_fill
                              - id: star_3
                                type: instance
                                use: star_fill
                              - id: star_4
                                type: instance
                                use: star_fill
                              - id: star_5
                                type: instance
                                use: star_fill
                                # Filled-star count is data_bound.overview_rating; use DS Star-fill for filled and its outline sibling for unfilled — DS-owned.
                      - id: divider_1
                        type: rect
                        size: [1, 24]
                        bg: background-app
                      - id: reviews_group
                        type: column
                        gap: gap-2x-small-4
                        children:
                          - id: reviews_count
                            type: text
                            source: data
                            name: "{overview_reviews_count} lượt"
                            typography: display-caption
                            color: text-primary
                          - id: reviews_label
                            type: text
                            name: "Đánh giá từ cư dân"
                            typography: body-caption
                            color: text-primary
                      - id: divider_2
                        type: rect
                        size: [1, 24]
                        bg: background-app
                      - id: amenities_group
                        type: column
                        gap: gap-2x-small-4
                        children:
                          - id: amenities_count_row
                            type: row
                            gap: gap-2x-small-4
                            align: center
                            children:
                              - id: amenities_count_text
                                type: text
                                source: data
                                name: "{overview_amenities_count}+"
                                typography: display-caption
                                color: text-primary
                              - id: amenities_icons_row
                                type: row
                                gap: -5                     # negative overlap — see 8_report
                                children:
                                  - id: amenity_icon_chip_1
                                    type: row
                                    pad: [4, 4, 4, 4]
                                    bg: background-secondary
                                    radius: radius-pill
                                    children:
                                      - id: amenity_icon_1
                                        type: icon
                                        source: data
                                        name: "{overview_amenity_icons[0]}"   # see [OI-6]
                                        size: 16
                                  - id: amenity_icon_chip_2
                                    type: row
                                    pad: [4, 4, 4, 4]
                                    bg: background-secondary
                                    radius: radius-pill
                                    children:
                                      - id: amenity_icon_2
                                        type: icon
                                        source: data
                                        name: "{overview_amenity_icons[1]}"
                                        size: 16
                                  - id: amenity_icon_chip_3
                                    type: row
                                    pad: [4, 4, 4, 4]
                                    bg: background-secondary
                                    radius: radius-pill
                                    children:
                                      - id: amenity_icon_3
                                        type: icon
                                        source: data
                                        name: "{overview_amenity_icons[2]}"
                                        size: 16
                          - id: amenities_label
                            type: text
                            name: "Tiện ích xung quanh"
                            typography: body-caption
                            color: text-primary

# ============================================================================
# BLOCK — characteristics_layout
# ============================================================================
screen: project_detail
section: characteristics
requirement: REQ-01
verifies: [AC-6, AC-8, AC-9]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its contents
  with the tree below. The tree IS the finished contents — whatever renders today
  and is not in the tree does not survive the replace.
what: >
  A wrapper that adds 8 side padding, containing one DS `Characteristic` instance.
  The instance renders the whole "Đặc điểm bất động sản" section internally: the
  heading, a large site-map image, three floor-plan rows (3PN / 2PN / 1PN), and a
  horizontal scroller of agent-posted unit ad cards filtered by this `project_id`.
  The developer does not build any of those internals here.
figma: "19027:49747"
reference_width: 375
maths: "root height = 889 (measured from Figma; DS-owned)"
computed:
  characteristics_wrapper: 375x889
  characteristic_instance: 375x889
tree:
  - id: characteristics_wrapper
    type: column
    figma: "19027:49747"
    pad: [8, 0, 8, 0]
    children:
      - id: characteristic_instance
        type: instance
        use: characteristic
        source: data
        props:
          project_id: "{project.id}"          # see [OI-5] — confirm the DS prop name

# ============================================================================
# BLOCK — eligibility_placeholder_layout   (NEW section, per PRD §4.1)
# ============================================================================
screen: project_detail
section: eligibility_placeholder
requirement: REQ-01
verifies: [AC-6, AC-10]
action: >
  NEW. This section does not exist today. Insert a new card between the
  `characteristics` section above and the `description` section below.
what: >
  A white card with a tabs row at the top ("Điều kiện mua" and "Điều kiện vay"
  side by side, active-tab underline in the primary token) and a body block below
  showing a centred info icon (icon-info) above the text "Sắp ra mắt" (bold 14).
figma: "(none — new)"
reference_width: 375
maths: "16 (top pad) + 40 (tabs) + 24 (gap) + 40 (icon) + 8 (gap) + 20 (line) + 16 (bot pad) = 164"
computed:
  eligibility_placeholder_root: 375x164
  eligibility_tabs_row: 343x40
  eligibility_tab: "~171x40"
  eligibility_body: 343x68
  eligibility_body_icon: 40x40
  eligibility_body_text: 343x20
tree:
  - id: eligibility_placeholder_root
    type: column
    bg: background-primary
    radius: radius-card
    pad: [16, 16, 16, 16]
    gap: gap-medium-16
    figma: "(none)"
    children:
      - id: eligibility_tabs_row
        type: row
        align: stretch
        children:
          - id: eligibility_tab_buy
            type: column
            grow: 1
            align: center
            pad: [0, 8, 0, 8]
            behavior: BH-07
            children:
              - id: eligibility_tab_buy_label
                type: text
                name: "Điều kiện mua"
                typography: header-caption
                color: text-primary
              - id: eligibility_tab_buy_underline
                type: rect
                size: [null, 2]                    # full-width underline; visible when active
                bg: button-primary
          - id: eligibility_tab_loan
            type: column
            grow: 1
            align: center
            pad: [0, 8, 0, 8]
            behavior: BH-07
            children:
              - id: eligibility_tab_loan_label
                type: text
                name: "Điều kiện vay"
                typography: header-caption
                color: text-tertiary                # inactive
              - id: eligibility_tab_loan_underline
                type: rect
                size: [null, 2]
                bg: transparent                     # inactive: no underline
      - id: eligibility_body
        type: column
        align: center
        gap: gap-x-small-8
        children:
          - id: eligibility_body_icon
            type: icon
            name: info-fill                        # component name TBC — DS should have an info-circle icon; use icon-info colour
            size: 40
            color: icon-info
          - id: eligibility_body_text
            type: text
            name: "Sắp ra mắt"
            typography: header-caption
            color: text-tertiary

# ============================================================================
# BLOCK — description_layout
# ============================================================================
screen: project_detail
section: description
requirement: REQ-01
verifies: [AC-6, AC-19]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its contents
  with the tree below. The tree IS the finished contents — whatever renders today
  and is not in the tree does not survive the replace.
what: >
  A wrapper that adds 8 side padding, containing one DS `Description-section`
  instance with `Platform=Mobile, Expand=False`. The instance renders the
  "Mô tả chi tiết" heading, the body text, and the "Xem thêm" expand toggle.
figma: "19027:49748"
reference_width: 375
maths: "root height = 287 (DS-owned, at Expand=False)"
computed:
  description_wrapper: 375x287
  description_instance: 375x287
tree:
  - id: description_wrapper
    type: column
    figma: "19027:49748"
    pad: [8, 0, 8, 0]
    children:
      - id: description_instance
        type: instance
        use: description_section
        source: data
        props:
          Platform: "Mobile"
          Expand: "False"
          # body copy prop name TBC — pass project.description or equivalent

# ============================================================================
# BLOCK — related_projects_layout
# ============================================================================
screen: project_detail
section: related_projects
requirement: REQ-01
verifies: [AC-6, AC-11, AC-15, AC-18, AC-19, AC-20, AC-21]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its contents
  with the tree below. The tree IS the finished contents — whatever renders today
  and is not in the tree does not survive the replace.
what: >
  A white card with 16/12/16/16 inner padding. Top row: the section heading
  "Các dự án NOXH khác" (bold 18) on the left and a small "Xem tất cả" tertiary
  pill button on the right. Below it, a horizontally-scrolling row (gap 12
  between cards) of at least 3 project cards, each 156×274: a square photo
  156×156 with a status pill badge in the top-left corner and a small photo-count
  stat in the bottom-left; below the photo, a Product Info block (156×118,
  padding 12/8/12/8) with the project name (up to 2 lines, bold 14), the price
  (bold 14 red) directly below, and a location row (location-fill icon 16 +
  a two-line location text 12 grey).
figma: "19027:49749"
reference_width: 375
maths: "12 (top pad) + 26 (heading row) + 12 (gap) + 274 (cards row) + 16 (bot pad) = 340"
computed:
  related_projects_root: 375x340
  related_header_row: 343x26
  related_heading: 260x26
  xem_tat_ca_button: 75x24
  related_scroller_row: 343x274                # visible frame; content width may exceed
  card_grid: 156x274
  card_photo_stack: 156x156
  card_photo: 156x156
  card_status_badge: "83–107 x 16 (data-dependent width)"
  card_stats_container: 156x24
  card_product_info: 156x118
  card_product_content: 132x62
  card_title: 132x40
  card_price: 132x20
  card_location_row: 132x36
  card_location_icon: 16x16
  card_location_text: 112x36
tree:
  - id: related_projects_root
    type: column
    figma: "19027:49749"
    bg: background-primary
    radius: radius-card
    pad: [16, 12, 16, 16]
    gap: gap-small-12
    children:
      - id: related_header_row
        type: row
        gap: gap-x-small-8
        justify: space-between
        align: center
        children:
          - id: related_heading
            type: text
            name: "Các dự án NOXH khác"
            typography: display-annotation
            color: text-primary
            truncate: ellipsis-1
          - id: xem_tat_ca_button
            type: instance
            use: button_tertiary_s
            behavior: BH-04
            props:
              Input Text: "Xem tất cả"
              Icon Left#896:98: false
      - id: related_scroller_row
        type: row
        gap: gap-small-12
        overflow: [x: auto]
        scroll_snap: card
        children:
          - id: card_grid_repeatable                    # rendered per related_projects_list item
            source: data
            name: "{related_projects_list}"
            layout: card_grid_template
            note: >
              The template below is applied once per item in related_projects_list. The tree
              lists two concrete card instances (from the Figma sample), but at runtime the
              count is dynamic — do not hardcode 3.
          - id: card_grid_1
            type: column
            behavior: BH-06
            children:
              - id: card_photo_stack_1
                type: stack
                bg: background-secondary                # image fallback (background-app also valid)
                radius: radius-card-small
                children:
                  - id: card_photo_1
                    type: image
                    source: data
                    name: "{related_projects_list[0].photo}"
                    size: [156, 156]
                  - id: card_status_badge_1
                    type: instance
                    use: text_label
                    source: data
                    offset: [inset-left: 8, inset-top: 8]
                    props:
                      Color: data                     # per data_bound.status_badge — sample "Green"
                      Input Text: "{related_projects_list[0].application_status_badge}"
                  - id: card_stats_container_1
                    type: row
                    offset: [inset-left: 0, inset-bottom: 0]
                    pad: [8, 0, 8, 8]
                    children:
                      - id: card_photo_count_group_1
                        type: row
                        gap: gap-2x-small-4
                        align: center
                        children:
                          - id: card_photo_count_text_1
                            type: text
                            source: data
                            name: "{related_projects_list[0].photo_count}"
                            typography: label-annotation
                            color: text-blank
                          - id: card_photo_count_icon_1
                            type: icon
                            name: photo-count-icon             # DS componentId 10784:1546
                            size: 12
                            color: icon-blank
                      - id: card_video_icon_group_1
                        type: row
                        gap: gap-min-2
                        children:
                          - id: card_video_icon_1
                            type: icon
                            name: video-icon                    # DS componentId 10784:1548
                            size: 12
                            color: icon-blank
              - id: card_product_info_1
                type: column
                pad: [12, 8, 12, 8]
                gap: gap-2x-small-4
                children:
                  - id: card_product_content_1
                    type: column
                    gap: gap-min-2
                    children:
                      - id: card_title_1
                        type: text
                        source: data
                        name: "{related_projects_list[0].name}"
                        typography: header-caption
                        color: text-primary                   # NOTE: mapped from raw #000000 per [OI-8]
                        truncate: line-clamp-2
                      - id: card_price_1
                        type: text
                        source: data
                        name: "{related_projects_list[0].price_range}"
                        typography: header-caption
                        color: text-error
                  - id: card_location_row_1
                    type: row
                    gap: gap-2x-small-4
                    align: start
                    children:
                      - id: card_location_icon_1
                        type: icon
                        name: Location-fill                    # componentId 104:8126
                        size: 16
                        color: icon-tertiary
                      - id: card_location_text_1
                        type: text
                        source: data
                        name: "{related_projects_list[0].location}"
                        typography: body-caption
                        color: text-tertiary
                        truncate: line-clamp-2
          # ... second concrete card sample repeats identically with [1] in place of [0].
          - id: card_grid_2
            type: column
            behavior: BH-06
            note: "Same structure as card_grid_1 with related_projects_list[1] bound."

# ============================================================================
# BLOCK — sticky_cta_active
# ============================================================================
screen: project_detail
section: sticky_bottom_cta
requirement: REQ-01
verifies: [AC-6, AC-12, AC-13, AC-20]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its contents
  with the tree below. The tree IS the finished contents — whatever renders today
  and is not in the tree does not survive the replace.
what: >
  A white bar fixed to the bottom of the viewport, containing a single DS
  Button-group instance with variant `Property 1 = "Chưa đóng hồ sơ"` (the
  active state — application is still open). The DS instance renders the CTA
  buttons inside; the primary CTA label is "Tư vấn hồ sơ".
figma: "19027:49820"
reference_width: 375
maths: "root height = 56 (Button-group). Home indicator (34) is iOS chrome and not built."
computed:
  sticky_cta_root: 375x56
  button_group_instance: 375x56
tree:
  - id: sticky_cta_root
    type: column
    figma: "19027:49820"
    bg: background-primary
    children:
      - id: button_group_instance
        type: instance
        use: button_group
        props:
          Property 1: "Chưa đóng hồ sơ"
        note: "Primary button label 'Tư vấn hồ sơ' is DS-owned; wire the on_tap handler per BH-05."

# ============================================================================
# BLOCK — sticky_cta_sold_out
# ============================================================================
screen: project_detail
section: sticky_bottom_cta
requirement: REQ-01
verifies: [AC-4, AC-12, AC-13, AC-20]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its contents
  with the tree below. The tree IS the finished contents — whatever renders today
  and is not in the tree does not survive the replace.
what: >
  Identical to sticky_cta_active in every respect except the DS Button-group
  variant is `Property 1 = "Đóng hồ sơ"` (application closed). The label stays
  "Tư vấn hồ sơ"; the DS component renders the disabled visual.
figma: "(not drawn — inferred per [OI-2])"
reference_width: 375
maths: "root height = 56"
computed:
  sticky_cta_root: 375x56
tree:
  - id: sticky_cta_root
    type: column
    bg: background-primary
    children:
      - id: button_group_instance
        type: instance
        use: button_group
        props:
          Property 1: "Đóng hồ sơ"
        note: "Tap is a no-op — see BH-05."
```

### Verify checklist

Numbered groups in run order. Each names the ACs it verifies and how to read a failure.

```yaml
0_right_section:                    # AC-6, AC-7, AC-8, AC-11, AC-14
  - "Before editing any file, confirm the located node for each section matches its
     `find_by` and `confirm`. If two nodes look alike (e.g. `Địa chỉ` vs `Đặc điểm`),
     re-read the confirm text — a partial-match rebuild of the wrong card is the
     single most common failure on this page."
  - "The near-miss sibling for each section is unchanged. Diff the neighbouring
     scroll section BEFORE and AFTER. A change there means you edited the wrong node."

1_padding_ownership:                # AC-7, AC-16
  - "Every section's `padding_check.surfaces == 1`. If two surfaces (two `bg` fills
     with a radius) are stacked, the section was built twice — remove the outer wrapper."
  - "Doubled inner padding (e.g. 32 instead of 16) means the same story — the located
     node ALREADY had padding, and the dev added a second card inside."

2_variant_isolation:                # AC-1..AC-5
  - "Rendering any single (construction_status, application_status) combo produces
     exactly one status_header block, one sticky_cta block, and no leaves whose ids
     appear only in another variant's tree."

3_computed:                         # AC-7, AC-17
  - "For each block, the measured root W and H match `computed.<root_id>`. If W and H
     are swapped, the block was built on the wrong axis — re-read `type`."
  - "Section-to-section gap in the content column is exactly 8 (the parent gap). A gap
     of 16 means someone added a margin on top — remove the margin, not the parent gap."

4_responsive:                       # AC-18, AC-19, AC-21
  - "Test at viewport widths 320, 360, 375, 414, 430. The page body must not scroll
     horizontally at any width — only the related_projects.row scroller does. Every
     text container has `min-width: 0` so a long value truncates rather than pushes."

5_tokens_and_literals:              # AC-15
  - "Every fill and text in the built page resolves to a token named in `Reusable.tokens`
     OR the two raw hex entries (raw-black-000000 or raw-blue-f2f6fc). Any other hex is
     a defect — report the node, the hex applied, and the token or DS component that
     should have supplied the colour."
  - "Every bare number in the tree was built as written. A `gap: 2` that landed as 4
     because someone rounded to the nearest token is a defect. Report the node, the
     spec number, and the number built. Do NOT rewrite the spec."

6_components_and_data:              # AC-8, AC-9, AC-11
  - "Every leaf with `source: data` resolves at runtime, through the source named in
     `data_bound`. A DS name hardcoded where the tree said `source: data` is a frozen
     sample — the classic failure mode being a status badge Color/text left as the
     Figma sample when the API result should drive both. Confirm by rendering a fixture
     where each enum takes a different value than the design shows."
  - "The Characteristic DS instance renders internal cards via its own project_id-keyed
     query — you should NOT see a related-ads fetch made from the parent page for those
     cards. If you do, [OI-5] resolved wrongly."

7_content_and_behavior:             # AC-10, AC-13, AC-14, AC-20
  - "No agent-editable affordance on the page (no 'chỉnh sửa', no 'báo cáo', no
     'liên hệ người đăng'). If a similar-looking page's shell includes those, they
     must be hidden for this route."
  - "Tapping a status badge does nothing. Tapping the 'Xem tất cả' button navigates
     to a NOXH Hub Dự án tab route (or a stub if the tab is unbuilt — BH-04). Tapping
     a related project card navigates to the same route with a different project_id
     — the page unmounts and remounts, no history stack pollution."

8_report:                           # to hand back after the build
  # (see Phase 3 appendices below for the sync tables the designer should apply to Figma)
  - "measured padding + slot gaps for every section"
  - "measured root W × H for every block, side-by-side with `computed`"
  - "every value off the DS token scale that was built as a literal — `gap: 2`,
     `gap: 1` (Stars container), `gap: 6` (rating column), `gap: 7` (unused; drift),
     `gap: -5` (amenity icon overlap) — flagged for design to confirm intentional"
  - "every leaf with `source: data` and how it resolved at build time"
  - "every raw hex that made it into the build, whether resolved to a documented raw
     token or unmapped"
  - "any Part 1 statement that Part 2 contradicted (e.g. a CR whose expected string
     format could not be enforced with the actual API payload)"
  - "the breakpoint used, and any layout drift observed at 320 and 430"
  - "the actual path the section landed at, written back into each Sections.path_hint"
```


---

## Appendix A — Phase 3a: Figma layer renames (to be applied by designer)

Phase 3 of `handoff-design-spec` normally renames `Frame 123456` / duplicate-name layers in Figma via `use_figma`, so the design file and its spec read as one. On this run the Figma write failed permissions in the harness — the personal-access token is read-only for this file, or the auto-mode classifier blocked write-through. The spec is unaffected (every reference in Part 2 keys off node ids, not layer names).

Below are the 60 renames the designer should apply, one batch, before this design ships to production. **Never rename** anything inside a component instance (`Characteristic`, `Description-section`, `Button-group`, `Button`, `Text Label`, `Icon`, `Star-fill`, iOS chrome bars), inside an icon's `vector`/`path` children, or in imported page chrome. **Only the frames listed below.**

```text
node id         old name                                 → new name
--------------- ----------------------------------------   ------------------------------
19027:49661     Frame 2085664603                         → photo_indicator_wrapper
19027:49662     Frame 2085664602                         → photo_indicator_pill
19027:49664     Frame 151090326                          → content_column
19027:49665     Frame 151089792                          → project_status_header
19027:49666     Frame 151088114                          → project_status_header_inner
19027:49667     Frame 151090498                          → project_status_header_content
19027:49668     Frame 2085665820                         → project_info_group
19027:49669     Frame 2085668319                         → badges_and_count_row
19027:49670     Frame 2085668374                         → badges_row
19027:49672     Frame 2085668319                         → badge_right_wrapper
19027:49674     Frame 2085668373                         → applicant_count_group
19027:49675     Group 1000005552                         → applicant_avatar_group
19027:49680     Frame 151090537                          → name_and_seller_column
19027:49681     Frame 2085663186                         → name_and_seller_row
19027:49682     Frame 2085664598                         → name_price_seller_column
19027:49688     Frame 2085668333                         → seller_row_outer
19027:49689     Seller info text container               → seller_row_inner
19027:49690     Frame 2085668295                         → seller_row
19027:49694     Frame 2085668311                         → key_stats_container
19027:49695     Frame 2085668277                         → key_stats_group
19027:49696     Frame 2085668276                         → key_stats_row
19027:49697     Location text container                  → stat_column_units
19027:49701     Location text container                  → stat_column_year
19027:49705     Location text container                  → stat_column_area
19027:49708     Frame 2085668299                         → address_and_overview_wrapper
19027:49709     Location                                 → address_and_overview_card
19027:49710     Title                                    → address_block
19027:49711     Listing Title                            → address_heading
19027:49712     Frame 2085668262                         → address_row
19027:49713     Listing Address Container                → address_container
19027:49714     Primary Address                          → address_primary
19027:49715     Secondary Address                        → address_secondary
19027:49716     Map                                      → map_thumb
19027:49717     Listing Thumbnail                        → map_thumb_image
19027:49719     Frame 2085668250                         → overview_block
19027:49720     Frame 2085668246                         → overview_inner
19027:49721     Secondary Address                        → overview_heading
19027:49722     Frame 2085668245                         → overview_row
19027:49723     Rating Container                         → rating_group
19027:49724     Rating                                   → rating_value
19027:49725     Stars Container                          → stars_row
19027:49732     Reviews Container                        → reviews_group
19027:49733     Reviews Count                            → reviews_count_value
19027:49736     Reviews Container                        → amenities_group   # duplicate name — this one is the amenities sub-row
19027:49737     Frame 2085668263                         → amenities_count_row
19027:49738     Reviews Count                            → amenities_count_value
19027:49739     Frame 2085668260                         → amenities_icons_row
19027:49740     Frame 2085668256                         → amenity_icon_chip_1
19027:49742     Frame 2085668257                         → amenity_icon_chip_2
19027:49744     Frame 2085668258                         → amenity_icon_chip_3
19027:49746     Reviews Label                            → amenities_label
19027:49749     Frame 2085665968                         → related_projects_section
19027:49750     Frame 2085665988                         → related_projects_header
19027:49753     row                                      → related_projects_scroller
19027:49820     bottom-button                            → sticky_cta
19027:49825     Header/ App & M-site                     → top_header_nav_row
19027:49826     Frame 151088061                          → back_pill
19027:49828     Frame 2085664615                         → right_group
19027:49829     Frame 151088061                          → share_pill   # duplicate id-generated name — right side, share button
19027:49831     Frame 2085663188                         → more_pill
```

## Appendix B — Phase 3b: Token / hex resolution decisions

Every fill and text in Part 2 that carries a token name below was read from Figma's `boundVariables` (via `get_variable_defs` on node `19027:49658`). Nothing was inferred by hex-nearest-token.

| Case | Where | Decision |
|---|---|---|
| **Bound to variable** — the node already declared `boundVariables.color` | ~95% of colours in the tree | Use the token name from `Reusable.tokens`. Never override. |
| **Raw `#000000` on card title** (`19027:49769`, `19027:49791`, `19027:49813`) | `Related Projects` cards | Mapped to `text-primary` (`#222222`) per Phase 3b context rule. Reported as [OI-8]. |
| **Raw `#F2F6FC`** on Text Label Blue variant background | Status badges Blue variant (main frame + variant frames) | Kept raw. Owned by DS via `Color=Blue` prop — dev never writes the hex directly. Reported as [OI-9]. |
| **`#F7F7F7` on the overview dividers** (`19027:49731`, `19027:49735`) | Between rating / reviews / amenities columns | Bound to variable — Figma reports `background-app`. That's the divider surface as designed; keep. |
| **`#222222` at 50% alpha (`#22222280`) on the photo pill** | Photo `1/5` pill background | Bound to `background-overlay`. |
| Surface-to-brand map (would render a grey card gold) | none in this tree | Hard-stop rule — never triggered here. |

## Appendix C — Phase 4 read provenance

For anyone auditing what this spec was built from:

| Call | Endpoint / MCP | Nodes | Purpose |
|---|---|---|---|
| Phase 0 | `GET /v1/me` | — | Token validity |
| Phase 1 | `GET /v1/files/{k}/nodes?ids=19027:49657,12382:7190&depth=2` | 2 | Frame list + variant discovery |
| Phase 1 | `GET /v1/images/{k}?ids=…&format=png&scale=1` | 5 | Cheap screenshots (main frame + 3 variants + Characteristic reference) |
| Phase 4a | `GET /v1/files/{k}/nodes?ids=19027:49708&depth=8` | 1 | Location section deep read (address vs overview subsections) |
| Phase 4a | `GET /v1/files/{k}/nodes?ids=19027:49659,19027:49664,19027:49820,19027:49823,19027:49665,19027:49749,19027:49833,19027:49875,19027:49917&depth=12` | 9 | Main frame + related_projects + 3 status variants, full depth |
| Phase 4a | `GET /v1/files/{k}/nodes?ids=16048:17485,16048:16070,16048:18933,16048:14114,1274:24347&depth=2` | 5 | DS component set variant enumeration (`project status`, `Text Label`, `Button-group`, `Button`, `Description-section`) |
| Phase 4b | `mcp__…__get_variable_defs(nodeId=19027:49658)` | 1 | Token name resolution — the only source for the token names in `Reusable.tokens` |
| Phase 4c | `get_design_context` | — | Not needed — REST + `get_variable_defs` sufficed |
| Phase 3a | `use_figma` (rename batch, 60 nodes) | — | Refused by the harness classifier; rename delivered as Appendix A to-do instead |

**Sources not consulted** (would need Enterprise scope or additional cost): `/v1/files/{k}/variables/local` returned 403 (token lacks `file_variables:read`) — replaced by MCP `get_variable_defs`. No web/desktop frames were fetched — Q1 confirmed Mobile App only.

## Appendix D — Self-check results

Ran on file at 100.7KB / 2138+ lines:

| Check | Result |
|---|---|
| YAML parse | pass |
| Every AC referenced in `verifies:` is defined | pass |
| Every BH/CR/OI referenced somewhere is defined | pass (OI-10 was added after first pass; see below) |
| No `+` composition in `what`/`context`/AC | pass |
| No `# waived` markers or `Waived values` section | pass |
| Every block has `action`, `reference_width`, `computed`, `tree`, `what`, `figma` | pass |
| Every section has `padding_check` | pass |
| Screen has `slot_check` | pass |
| `reference_width == 375` on every block | pass |
| `action: update` blocks contain the three replace-all sentences verbatim | pass |
| Question round ran once (Q1–Q4 in Phase 2) | pass |
| No matrix / `in_variant` / per-variant override keys in a block | pass — the two `extends:` cases in status_header variants B/C/D name a single parent block explicitly (not a matrix) and their `overrides` list changes prop values only, not tree structure. |
| Ids unique inside each block | pass |
| Colours: every colour is a token or a documented raw hex | pass |

**Only defect found on first pass** and fixed: `BH-03` referenced `[OI-10]` which was not initially defined. `OI-10` is now added under `before_qc`.
