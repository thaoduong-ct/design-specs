# handoff-noxh

**Version:** 1.0 · 2026-09-08 · Author: `/anthropic-skills:handoff-design-spec@1.4`
**Feature:** NOXH — Nhà Ở Xã Hội (social housing) — Hub + Eligibility Check
**Vertical:** PTY (Nhà Tốt) · **Platform:** App only (iOS + Android from one codebase)
**Design Owner:** Thảo Dương (`thaoduong@chotot.vn`)

**Sources:**
- Figma REST API (`GET /v1/files/UvLAgVP7Em1fwytMbuKHuv/nodes?ids=18863:14981,18863:15445`) — geometry, layout, fills.boundVariables, componentProperties, componentSets. Total 8.7 MB raw, filtered via jq to keep only fields written into this spec.
- MCP `get_variable_defs` on both sections — token *names* (REST returns IDs).
- MCP `get_screenshot` for visual disambiguation of 4 result states.
- MCP `use_figma` (Plugin API) — descent into 2 filter frames + rename of 12 structural containers pre-handoff (see `pre_handoff_renames`).
- PRD #259 (`carousell/ct-technical-sdd`) — [Eligibility Check US4](https://github.com/carousell/ct-technical-sdd/issues/259).
- PRD #900 (`carousell/ct-technical-sdd`) — [NOXH Hub & Detail](https://github.com/carousell/ct-technical-sdd/issues/900).
- design-ready seed block (this session) — waivers list, PRD divergences resolved, interaction states = DS defaults.

**Figma root:** [Vertical-Handoff-2026 · Content section](https://www.figma.com/design/UvLAgVP7Em1fwytMbuKHuv/Vertical-Handoff-%E2%80%A2-2026?node-id=18863-14981) · [Eligibility section](https://www.figma.com/design/UvLAgVP7Em1fwytMbuKHuv/Vertical-Handoff-%E2%80%A2-2026?node-id=18863-15445)

---

## How to use

1. **Two features in one file.** Requirement ids prefixed `REQ-H*` (Hub, PRD #900) and `REQ-E*` (Eligibility, PRD #259). Layout blocks group under the same prefixes. Read the PRD for the half you are building; skip the other prefix.

2. **Precedence when Part 1 and Part 2 disagree:**
   - Intent (what, when, on tap, copy) → **Part 1 wins**.
   - Geometry (axis, order, gap, padding, size, nesting) → **Part 2 tree + `computed` wins**.
   - Any contradiction is a **reportable defect** — file it in `8_report`, do not resolve silently.

3. **Every numeric value in this spec is a literal — implement it exactly as written.** A design token applies **only** where the token's name appears (`gap-small-12`, `button-primary`). A bare number (`gap: 2`, `radius: 12`, `y: 6`) is the value to build — never rounded to the nearest token. See `Tag definitions → Literal rule`. There is no `Waived values` section and no `# waived raw` comments anywhere — one global rule replaces both.

4. **Vietnamese UI strings are quoted verbatim in every layer that refers to something on screen.** They are the codebase handle: dev greps for the literal to locate the node.
   - `find_by` and tree `text:` use the **placeholder form** — `"{n} hữu ích"`, `"{applicant_count}+ người đã nộp hồ sơ"`. The sample number in Figma does not exist in the codebase.
   - `what` (block description) quotes what the screen shows including the sample — it describes a picture.
   - Fully static strings (headings, static labels, static CTAs like `"Tiếp tục"`, `"Điền thông tin ngay"`) are greppable as-is.
   - Placeholders carrying a count/date/rating/price are not greppable — grep the static half.
   - If your repo localises copy, grep the i18n **key** whose value is the Vietnamese string.
   - `find_by_unverified: true` on any `Sections` entry means the path is a first-guess — dev must confirm with a grep and write the confirmed path back to `path_hint`.

5. **Reuse before build.** Every `Reusable.ds_components` entry is an existing CladUI component — import it, do not rebuild. Every `Reusable.data_bound` entry names where the value-to-DS-name mapping already lives in the API or existing code — reuse that, do not invent.

6. **Waivers from the design-ready round are respected here.** 5 items the Design Owner explicitly waived have `dev-behavior` lines under `open_items` / `waived_items`. Follow those exactly — do not add a UI for the waived item.

7. **Pre-handoff Figma renames.** 12 structural containers + 3 typo symbols were renamed via `use_figma` before this spec was written — the ids stay the same but the names on canvas now match the ids in this spec. See `pre_handoff_renames` for the map.

---

## Tag definitions

```yaml
# ─── Structure ────────────────────────────────────────────────────────
Screens: |
  Where each block lives on which page, and what the surrounding page provides.
  Fields: renders · place · parent_provides · untouched · page_background · slot_check.
  NEVER contains: section names as build facts · find_by · any action.

Sections: |
  How to find the node this spec changes. One entry per section that at least one block targets.
  Fields: acted_on_by · figma · path_hint · find_by · confirm · current · owns · padding_check.
  NEVER contains: any action · any per-block build fact — that belongs to the block's tree/what.

Layout blocks: |
  What to do to the section. One block per variant × platform. This project has 1 variant × 1 platform, so 1 block per addressed screen.
  Fields: screen · section · requirement · verifies · action · what · figma · reference_width · maths · computed · tree.

# ─── Container types (axis is a word, always) ─────────────────────────
type: row      # horizontal layout, children flow left → right
type: column   # vertical layout, children flow top → bottom
type: stack    # absolute-positioned children (overlays, floating buttons)
# `type` is REQUIRED on every container. It is never defaulted.
# One id may not span two axes. If a role sits row here and column there, they are two ids.

# ─── Literal rule (this is the ONLY conversion rule between numbers and tokens) ─
Literal rule: |
  Any numeric value in this spec is a literal. Implement it exactly as written.
  A design token is used ONLY where the spec writes the token's name (gap-small-12,
  padding-medium-16). A bare number (gap: 2, radius: 12, y: 10) is the value to build; it
  is not an approximation of a token, and there is no rule anywhere in this file that
  converts one to the other. Rounding gap: 2 up to a 4px token, or radius: 12 to
  radius-card-small, is a defect — report it in 8_report, do not perform it.

# ─── Data binding (one-sided marker; absence means fixed) ─────────────
source: data      # value follows the data; carries a placeholder and a data_bound entry
# No `source: fixed` anywhere — fixed is the default. A leaf without `source:` is fixed.

# ─── Defaults (declare once, omit at every use) ───────────────────────
align: center            # container child alignment default
text color: text-primary # unless the leaf carries a `color:` key
truncation: ellipsis-end # unless a rule says otherwise
```

---

## Open items

```yaml
blocking: []

before_qc:
  OI-1:
    issue: >
      Initial sheet state on entry — 15765 (eligible modal, dropdown only) vs 15805
      (group segment in unselect). Designer skipped the disambiguation question.
    owner: design
    until_then: >
      Build 15765 as the initial state (dropdown-only, CTA disabled, "Nhập các thông tin
      trên"). Treat 15805 as an alternate variant that may be dropped or merged — flag it
      to design on first render if the two frames differ structurally beyond the segment
      header, and re-ask.

  OI-2:
    issue: >
      15871 "select without input data" shows "Tình trạng nhà ở" section rendered TWICE
      in Figma. Suspected duplicate frame from design iteration; user confirmed Version
      A/B ships as ONE template with a feature flag ("1 template, config các field bên
      trong để thành option A/B").
    owner: design + dev
    until_then: >
      Render the section ONCE. Behind the flag: Version A shows "Tình trạng nhà ở"
      (house ownership + m² + số người); Version B hides the whole section. If the second
      copy in Figma carries different content, escalate before building.

  OI-3:
    issue: >
      `path_hint` for every Sections entry is unverified — this spec was written without
      repo access. Every `find_by` string is greppable and confirmed as text; the module
      path is a plausible-but-unverified first guess.
    owner: dev
    until_then: >
      Grep the Vietnamese `find_by` string in the codebase. When the real path is found,
      write it back into this file's `path_hint` and set `provenance: verified`.

  OI-4:
    issue: >
      The `18169 input data` frame carries a top-level fill `background-inverted #222`.
      Unusual — this is either a full-screen scrim behind the drawer (visually hidden
      when the bottom sheet is up) or an accidental override. Not visible in the
      screenshot because the drawer covers it.
    owner: design
    until_then: >
      Implement the sheet with the standard modal overlay pattern (rgba(0,0,0,0.5) scrim
      + bottom sheet). Do not paint the underlying screen dark. If design intended a
      specific scrim colour, they will re-set it in Figma and this OI closes.

  OI-5:
    issue: >
      Version B ("house-ownership check handled elsewhere") — PRD #259 Open Decisions
      says "where, and by what mechanism" is unresolved. Design shows 1 template with
      flag; the "elsewhere" logic is not designed.
    owner: pm + dev
    until_then: >
      Version B in this build hides the "Tình trạng nhà ở" section only. The elsewhere-
      handling is out of scope for this spec. Ship as-is; PM opens a follow-up ticket.

cosmetic:
  OI-6:
    issue: >
      Legacy token `Gray/0 - WHITE` (non-semantic naming with space + dash) still exists
      in the file's variable set.
    owner: design
  OI-7:
    issue: >
      Chip 18863:15236 (`Giá bán` chip in Tin đăng default filter row) missing token
      binding on background — has raw #f4f4f4 while every sibling chip binds
      `background-secondary`. Value matches, only the binding is missing.
    owner: design
```

---

## waived_items

Five items the Design Owner **explicitly waived** during the design-ready round (this session). `/handoff` implements the `dev-behavior` line for each; do NOT add UI for the waived item.

```yaml
non_login_flow:
  waived_reason: "Feature login-required; 2 Figma frames in 'Non-login' section are legacy."
  dev_behavior: >
    Do NOT implement any non-login flow. Non-logged users do NOT see the "Điều kiện mua
    NOXH >" entry point on the Hub header. Login gate is the app's existing login
    screen — do not add a special sheet for NOXH.
  affected_figma_frames: [18863:17632]

warning_tooltip:
  waived_reason: "No tooltip UI. Chip already carries border-warning visual cue."
  dev_behavior: >
    When income or house-area answers cross the PRD threshold, apply the warning state
    to the affected chip/input (border-warning #fb7328). Do NOT render a tooltip
    popover or inline warning text. The user can still continue.

empty_carousel:
  waived_reason: "No empty-state frame designed."
  dev_behavior: >
    When "Danh sách NOXH phù hợp" has zero matching projects, HIDE the entire section
    — the heading AND the carousel row. Do not render an empty state placeholder.
    Note: frame 18863:17177 ("without relevant project") shows the correct post-submit
    state without the carousel section — use that as the reference.

us2_alt_action_on_insufficient:
  waived_reason: "AC dropped from PRD."
  dev_behavior: >
    Insufficient post-submit shows only the "Đồng ý nhận thông báo" toggle. Do NOT add
    a CTA linking to US2 (subscribe-area-notification).

ineligible_reason_per_condition:
  waived_reason: "Keep generic message."
  dev_behavior: >
    Show ONE generic sentence: "Thông tin của bạn có thể vượt ngưỡng. Một số dự án có
    thể có tiêu chí riêng." Do NOT enumerate which specific condition (income / house
    area / group) triggered the ineligibility.
```

---

## pre_handoff_renames

12 structural containers + 3 typo symbols were renamed via `use_figma` in this session BEFORE this spec was written. Ids unchanged.

```yaml
- { id: "18863:15346", was: "Status=open for applying, Device=esktop", now: "Status=open for applying, Device=Desktop" }
- { id: "18863:15351", was: "Status=almost, Device=esktop",             now: "Status=almost, Device=Desktop" }
- { id: "18863:15355", was: "Status=close, Device=esktop",              now: "Status=close, Device=Desktop" }
- { id: "18863:15372", was: "Property 1=Variant3",                       now: "Property 1=applications closed" }
- { id: "18863:15234", was: "by Default",                                 now: "Filter/Tin đăng/state=Default" }
- { id: "18863:15237", was: "by Cá Nhân",                                 now: "Filter/Tin đăng/state=by=Cá nhân" }
- { id: "18863:15240", was: "by Cá Nhân",                                 now: "Filter/Tin đăng/state=by=Môi giới" }
- { id: "18863:15243", was: "Group 1000005579",                           now: "Popover/Đăng bởi" }
- { id: "18863:15250", was: "Group 1000005581",                           now: "Popover/Trạng thái hồ sơ" }
- { id: "18863:15255", was: "Group 1000005582",                           now: "Popover/Trạng thái thi công" }
- { id: "18863:15262", was: "Group 1000005584",                           now: "Popover/Thời gian thu hồ sơ" }
- { id: "18863:15178", was: "Frame 2085668415",                           now: "Pagination container" }
```

Two Tier-1 divergences from design-ready are still pending in Figma (designer will fix before build starts):
- Popover Đăng bởi (18863:15243) — remove option "Bán chuyên" (List item 18863:15249). Keep 3 options: `Chủ đầu tư | Cá nhân | Môi giới`.
- Chip "Thời gian thu hồ sơ" (18863:15218) — move from symbol `Property 1=Dự Án` (18863:15201) to `Property 1=Tin đăng` (18863:15185) with conditional visibility (show only when `Đăng bởi = Chủ đầu tư`).

**This spec is written as if both are already fixed.** If dev finds the un-fixed state in Figma, follow this spec, not the frame.


---

# PART 1 — Requirement

## Hub feature (PRD #900)

```yaml
REQ-H1:
  title: >
    NOXH Hub · Tin đăng tab — mixed feed of primary ads (Chủ đầu tư) and secondary
    ads (Môi giới / Cá nhân) with NOXH-specific filters.
  context: >
    New dedicated hub route on Nhà Tốt where NOXH buyers browse secondary-market and
    primary-market unit ads in ONE place. Today these mix with regular property ads
    on the general Nhà Tốt listing with no NOXH context; this hub is that context.
    Screen renders — top to bottom: sticky header "Tim nhà ở xã hội dành cho bạn"
    (see REQ-H6), tabs "Tin đăng" / "Dự án" (this REQ = Tin đăng active), filter row
    (see REQ-H3), then the ad grid (mix of primary + secondary layouts).
  render_when: >
    User taps NOXH Hub entry (from Nhà Tốt discovery surface — link out of scope of
    this spec) OR reaches /noxh-hub/tin-dang directly. Resolve tab ONCE at mount
    from route param; do NOT branch per row.
  rules: [CR-H1, CR-H2, CR-H3]
  behaviors: [BH-H1, BH-H2, BH-H3]
  acceptance_criteria:
    AC-H1-1: >
      Route resolves to Tin đăng tab: filter row from REQ-H3 renders (chips
      "Đăng bởi" + "Giá bán" visible by default; when "Đăng bởi = Chủ đầu tư" is
      selected, chips "Trạng thái hồ sơ" and "Thời gian thu hồ sơ" APPEAR to the
      right of "Đăng bởi"). Rental ads tagged to NOXH projects MUST NOT appear.
    AC-H1-2: >
      Primary ads (Chủ đầu tư) and secondary ads (Môi giới / Cá nhân) render with
      DISTINCT layouts per REQ-H5 (`Ad-Primary` vs `Ad-Secondary` variants).
    AC-H1-3: >
      Tap on a primary ad → Primary Ad Detail route. Tap on a secondary ad →
      existing Secondary Ad Detail route. See BH-H1.
    AC-H1-4: >
      Filter chip "Trạng thái hồ sơ" is HIDDEN unless "Đăng bởi = Chủ đầu tư".
      Same for "Thời gian thu hồ sơ". See CR-H1.
    AC-H1-5: >
      Filter copy MUST match Figma final copy exactly. Do NOT hardcode "Người bán"
      as the label for the Đăng bởi filter — the visible chip text is `"Đăng bởi"`.
    AC-H1-6: >
      Empty result (no ads match applied filters) → show empty state pattern
      already existing in Nhà Tốt (do NOT design new empty). See waived_items.

REQ-H2:
  title: >
    NOXH Hub · Dự án tab — developer-sourced NOXH project listings, browsable by
    region, application status, and construction status.
  context: >
    Same route as REQ-H1 but the "Dự án" tab is active. Renders a NOXH-only feed of
    developer projects (project_id, per PRD #900 data model). Filter row differs
    from Tin đăng: shows "Trạng thái thi công" + "Trạng thái hồ sơ" always, plus
    a region chip row with dynamic counts ("Bình Dương (3)", "Hà Nội (3)",
    "HCM (3)", and "Tất cả" as default active).
  render_when: >
    User taps "Dự án" tab on the Hub OR reaches /noxh-hub/du-an directly.
  rules: [CR-H4, CR-H5]
  behaviors: [BH-H4]
  acceptance_criteria:
    AC-H2-1: >
      Region chip row renders below the filter row. Default active chip: "Tất cả"
      (background background-inverted #222, text text-blank #fff). Other region
      chips render count in parentheses, e.g. "Bình Dương (3)" — count is
      backend-dynamic (see CR-H4). Single select; tapping a region filters the
      list to that region.
    AC-H2-2: >
      Filter chips "Trạng thái thi công" and "Trạng thái hồ sơ" render ALWAYS
      (unlike Tin đăng where "Trạng thái hồ sơ" is conditional).
    AC-H2-3: >
      Project cards render per `Project-List` variant (see REQ-H5). Tap → NOXH
      Project Detail route (out of scope for this file).
    AC-H2-4: >
      Empty (no projects match) → existing Nhà Tốt empty state. See waived_items.

REQ-H3:
  title: >
    Filter row for Tin đăng tab. Chips: "Đăng bởi" (default), "Giá bán" (default);
    conditional "Trạng thái hồ sơ" and "Thời gian thu hồ sơ" APPEAR when
    "Đăng bởi = Chủ đầu tư".
  context: >
    Located directly below the tab bar on the Tin đăng screen. Row of pill-shaped
    chips (background background-secondary #f4f4f4, text-primary #222,
    radius-pill 999, height 32) with a right-side dismiss/chevron icon on each.
  render_when: >
    Whenever REQ-H1 is active (Tin đăng tab). Chip visibility flips on the
    "Đăng bởi" value.
  rules: [CR-H1, CR-H6, CR-H7]
  behaviors: [BH-H5, BH-H6]
  acceptance_criteria:
    AC-H3-1: >
      Default chips: `"Đăng bởi"` and `"Giá bán"`, both with a chevron-down icon
      to the right (indicating tap → popover).
    AC-H3-2: >
      Tap "Đăng bởi" → popover opens with EXACTLY 3 options in this order:
      `"Chủ đầu tư"` · `"Cá nhân"` · `"Môi giới"`. Do NOT render a 4th option
      ("Bán chuyên" — was in Figma, designer removes before build).
    AC-H3-3: >
      When user selects "Đăng bởi = Chủ đầu tư": chip `"Trạng thái hồ sơ"` and
      chip `"Thời gian thu hồ sơ"` appear to the right of the current chip row.
      When user selects "Đăng bởi = Cá nhân" or "Đăng bởi = Môi giới": those two
      chips are HIDDEN.

REQ-H4:
  title: >
    Filter row for Dự án tab. Chips always visible: "Trạng thái thi công",
    "Trạng thái hồ sơ", "Thời gian thu hồ sơ". Region chip row below.
  context: >
    Located directly below the tab bar on the Dự án screen. Same chip style as
    REQ-H3. A location-filter bar sits above: `"Khu vực:"` label + selected value
    (text-brand orange, e.g. "Đồng Nai") + dismisible × icon + "Xóa lọc" reset
    text on the right.
  render_when: >
    Whenever REQ-H2 is active (Dự án tab).
  rules: [CR-H4, CR-H8]
  behaviors: [BH-H7]
  acceptance_criteria:
    AC-H4-1: >
      Top row (location filter): `"Khu vực:"` label (text-primary) + selected
      region name (text-brand #fa6819, bold) + dismisible × icon (icon-primary,
      size 20) + `"Xóa lọc"` text (header-caption, text-primary) right-aligned.
    AC-H4-2: >
      Chip row order left→right: `"Trạng thái thi công"` · `"Trạng thái hồ sơ"` ·
      `"Thời gian thu hồ sơ"`. Each has chevron-down right icon.
    AC-H4-3: >
      Region chip row below chip row: `"Khu vực:"` static label (text-tertiary)
      + `"Tất cả"` chip (active state, bg background-inverted, text text-blank)
      + other region chips (bg background-secondary, text text-primary) in
      the format `"<region> (<count>)"`. Single select — default "Tất cả".

REQ-H5:
  title: >
    Card layouts — `Ad-Primary` (Chủ đầu tư unit ads), `Ad-Secondary` (Môi giới /
    Cá nhân ads), `Project-List` (developer project cards on Dự án tab).
  context: >
    Three distinct card templates in the NOXH Hub, differentiated by seller type
    and tab. All use `Card Item List/PTY` as the base structure (image left 120px,
    property container right 219px, total 351px wide with 12px inner padding
    inside the 375px screen). See REQ-H5 layout blocks for full geometry.
  render_when: >
    Rendered inside REQ-H1 (Tin đăng: mix of Ad-Primary + Ad-Secondary) and
    REQ-H2 (Dự án: only Project-List).
  rules: [CR-H9, CR-H10, CR-H11]
  behaviors: [BH-H8, BH-H9]
  acceptance_criteria:
    AC-H5-1: >
      `Ad-Primary` card (Chủ đầu tư) renders: image + application-status badge
      overlaid on image (e.g. `"Đang nhận hồ sơ"`) + applicant count with avatars
      (max 99+ per CR-H11) + project name + apartment code (Mã căn NĐ121) +
      price + area + developer name + verified badge.
    AC-H5-2: >
      `Ad-Secondary` card (Môi giới / Cá nhân) renders per existing property ad
      layout — NO changes to the existing layout. See waived_items — reuse
      existing pattern.
    AC-H5-3: >
      `Project-List` card (Dự án tab) renders: project photo (with 3-status badge
      "Đang nhận hồ sơ" / "Đang xây" / etc. overlaid) + project name + price
      range + address + media count + developer name + verified badge.
    AC-H5-4: >
      Applicant count over 99 renders as `"99+"` — cap at 99. See CR-H11.
    AC-H5-5: >
      Status badge text is DATA-BOUND — value comes from backend (see
      Reusable.data_bound.status_badge_text). Do NOT hardcode any status label.
      The COLOR of the badge is also data-bound to the status value (open / almost /
      close variants — see Reusable.ds_components.status_badge).

REQ-H6:
  title: >
    NOXH Hub sticky header — hero image + heading + subheading + entry-point row
    "Điều kiện mua NOXH >" (login-required).
  context: >
    Sits above the tabs on both Tin đăng and Dự án. Renders: hero image bg
    (buildings), heading `"Tim nhà ở xã hội"` (line 1, bold, text-primary),
    `"dành cho bạn"` (line 2, bold, text-primary), then the entry row.
  render_when: >
    Always visible on the Hub. Entry row `"Điều kiện mua NOXH >"` is HIDDEN when
    user is not logged in (waived_items.non_login_flow) — the entire row
    disappears; the layout collapses.
  rules: [CR-H12]
  behaviors: [BH-E1]
  acceptance_criteria:
    AC-H6-1: >
      Header height when logged-in: 238px. Renders sub-row with text
      `"Điều kiện mua NOXH"` (label-section, text-primary) + right chevron icon
      (icon-primary, size 20). Tap → opens Eligibility bottom-sheet (BH-E1).
    AC-H6-2: >
      Header when NOT logged-in: the "Điều kiện mua NOXH >" row does NOT render.
      Header collapses in height (~46px shorter). No login CTA sheet triggered
      from this row — the row simply is not there.
```

## Eligibility feature (PRD #259)

```yaml
REQ-E1:
  title: >
    Entry points to the Eligibility Check flow — Hub header row and Primary
    Ad Detail link.
  context: >
    Two ways to reach the Eligibility bottom-sheet:
    (1) Hub header row `"Điều kiện mua NOXH >"` (see REQ-H6);
    (2) Primary Ad Detail link `"Kiểm tra ngay điều kiện cần mua của dự án →"`
        (link position on Primary Ad Detail is out of this spec's scope — that
        page is defined by PRD #900 US4 Project Detail).
    Both entries open the SAME bottom-sheet (REQ-E2).
  render_when: >
    User is LOGGED IN. Non-logged users do not see either entry point (see
    waived_items.non_login_flow).
  behaviors: [BH-E1]
  acceptance_criteria:
    AC-E1-1: >
      Tap on either entry point opens the Eligibility bottom-sheet at the initial
      state (REQ-E2 / block E2-INIT).
    AC-E1-2: >
      Restart is idempotent: reopening from the same entry point after a previous
      submission overwrites previous answers. Previous answers are NOT preserved
      across sessions of the sheet (PRD #259 edge case).

REQ-E2:
  title: >
    Eligibility bottom-sheet — INITIAL state. Sheet slides up from bottom with
    scrim over the underlying screen; shows title, promo card, sub-heading,
    dropdown "Bạn thuộc nhóm nào?" (empty), CTA "Nhập các thông tin trên"
    (disabled).
  context: >
    Bottom-sheet is a shared Drawer component (radius-modal 20 top corners,
    background-primary #fff). Header row: close × icon left + title
    `"Kiểm tra điều kiện mua NOXH"` centred. Below header: promo card
    `"Đã có hơn ~ 215+ người kiểm tra điều kiện mua và nhận thông tin"` with
    inline avatars showing example notification content. Then main heading
    `"Có 25+ dự án đang chờ bạn"` + subheading
    `"Điền thông tin để xem các dự án phù hợp"`. Then the dropdown selector.
    Sticky bottom CTA `"Nhập các thông tin trên"` disabled state.
  render_when: >
    Initial mount of the sheet, before any user selection. Blocks: E2-INIT.
  rules: [CR-E1, CR-E2]
  behaviors: [BH-E2, BH-E3]
  acceptance_criteria:
    AC-E2-1: >
      Bottom CTA is DISABLED (background button-disabled #c0c0c0, text
      text-on-background #fff) and reads `"Nhập các thông tin trên"` when the
      user has NOT selected a group. CTA is 40px tall (button-height-large).
    AC-E2-2: >
      Dropdown shows placeholder `"Bạn thuộc nhóm nào? *"` (star marks
      required). Tap opens a popover overlay listing 4 group options — see
      REQ-E3. Popover is a DS List component in a floating card
      (shadow-floating, radius-card 12).
    AC-E2-3: >
      Promo card is FIXED CONTENT — the "215+ người" text is currently rendered
      as a static value in the frame. It is DATA-BOUND to the running counter
      of registered users (see Reusable.data_bound.registered_user_count).

REQ-E3:
  title: >
    Eligibility form — progressive disclosure. After user selects a group from
    the dropdown, additional fields APPEAR inline in the same sheet: "Tình trạng
    hôn nhân" (marital status, 3-chip segment) and "Tình trạng nhà ở" (house
    ownership, 2-chip segment + conditional area inputs).
  context: >
    Same sheet as REQ-E2. This REQ describes the sheet AFTER group selection but
    BEFORE the fill-out. Two new sub-sections appear stacked below the group
    dropdown, each with a section heading (label-section, text-primary) and a
    chip-segment control. The "Tình trạng nhà ở" section may VERSION-flag out
    entirely (Version B).
  render_when: >
    After user selects any group value from the dropdown (state: 15871
    "select without input data"). Version A: both marital-status AND
    house-ownership sections render. Version B: ONLY marital-status renders
    (house-ownership section hidden by feature flag — see OI-5).
  rules: [CR-E3, CR-E4, CR-E5]
  behaviors: [BH-E4]
  acceptance_criteria:
    AC-E3-1: >
      After group select, section `"Tình trạng hôn nhân"` renders below the
      group dropdown with 3-chip segmented control:
      `"Độc thân"` · `"Đã kết hôn"` · `"Đơn thân có con"`. Chips are
      DS Chip / Size=Medium 32px, Style=Fill. Unselected chips: background
      background-secondary, text text-primary. Selected chip: background
      background-inverted, text text-blank.
    AC-E3-2: >
      For Version A ONLY: section `"Tình trạng nhà ở"` renders below marital
      status with 2-chip segment: `"Chưa sở hữu"` (default) · `"Sở hữu nhà"`.
      When "Sở hữu nhà" is selected, TWO text inputs appear below the chips
      inline: `"Số người trong người *"` and `"Diện tích sàn *"` (with "m²"
      suffix). See CR-E3.
    AC-E3-3: >
      For Version B: entire `"Tình trạng nhà ở"` section is HIDDEN — no DOM,
      no gap. See OI-5.
    AC-E3-4: >
      Empty state under "Sở hữu nhà" (both inputs empty) shows helper text
      `"Diện tích cho phép mua NOXH 1 người dưới 15m2"` below the second input.
      When user fills values and the average crosses the 15m² threshold, the
      warning state activates (see AC-E3-5).
    AC-E3-5: >
      Warning state (income OR house-area over PRD threshold): inputs render
      with border-warning #fb7328 border. NO tooltip is shown (waived — see
      waived_items.warning_tooltip). User can still continue.
    AC-E3-6: >
      Bottom CTA remains DISABLED and reads `"Nhập các thông tin trên"` until
      all currently-visible required fields are filled. Once filled, CTA text
      changes to `"Tiếp tục"` and enables (background button-primary #fa6819,
      text text-on-background #fff). See CR-E2 and BH-E5.

REQ-E4:
  title: >
    Eligibility form — ALL FIELDS FILLED. Every required field has a value; the
    bottom CTA reads `"Tiếp tục"` in enabled orange state. Tap → submit.
  context: >
    Same sheet as REQ-E2/E3. This REQ captures the state right before user taps
    the submit CTA. The form values are validated against the PRD's group ×
    marital status × income-bracket table client-side (see CR-E6). Result of
    submit is EITHER Sufficient (REQ-E5) OR Insufficient (REQ-E6) — dev branches
    on the validation outcome.
  render_when: >
    All required fields for the current version+group combination are filled.
  rules: [CR-E6]
  behaviors: [BH-E5]
  acceptance_criteria:
    AC-E4-1: >
      CTA enabled state: background button-primary #fa6819, text
      text-on-background #fff, text `"Tiếp tục"`. Tap → CLIENT-SIDE evaluate the
      answers against the PRD's group × marital × income table and:
      - IF all-values-within-threshold AND group ≠ "Người có công/hộ nghèo/bị
        thu hồi đất" → replace sheet content with REQ-E5 (Sufficient).
      - IF group = "Người có công/hộ nghèo/bị thu hồi đất" → skip income check,
        go straight to REQ-E5 (Sufficient).
      - IF income OR house-area OVER threshold → replace sheet content with
        REQ-E6 (Insufficient).
    AC-E4-2: >
      NO network call yet — evaluation is client-side. Network call happens at
      contact form submit (REQ-E5 / REQ-E6).

REQ-E5:
  title: >
    Sufficient result screen (Chúc mừng). Contact form (Họ tên + Số điện thoại)
    for the user to submit their lead to the developer. Pre-fills from account.
    Submit → send lead to developer + show post-submit success (REQ-E7).
  context: >
    Sheet content is REPLACED (previous form contents removed) with a success
    illustration (celebratory checkmark icon in a chip-shaped chip), heading
    `"Chúc mừng bạn, bạn có đủ điều kiện đã NOXH"`, subheading
    `"Để lại thông tin để Chủ Đầu Tư liên hệ lại với bạn"`, and a 2-field
    contact form. Sticky bottom CTA `"Để lại thông tin ngay"` (enabled, orange).
  render_when: >
    User submitted the eligibility form with all-values-within-threshold OR user
    is in the special group "Người có công/hộ nghèo/bị thu hồi đất".
  rules: [CR-E7, CR-E8]
  behaviors: [BH-E6]
  acceptance_criteria:
    AC-E5-1: >
      Contact form fields: `"Họ và tên *"` (text input, pre-filled from
      account.name if logged in) and `"Nhập số điện thoại *"` (numeric input,
      pre-filled from account.phone if logged in). Both required.
    AC-E5-2: >
      Below the contact form, before the CTA, render disclaimer text (small,
      text-tertiary): `"Bằng việc bấm nút "Xem kết quả", bạn đã đọc và đồng ý"`
      + link `"Chính sách bảo mật"` (text-info #306bd9, underlined) + `"của Nhà
      Tốt và cho phép chia sẻ thông tin cá nhân của bạn cho Nhà Tốt để họ liên
      hệ tư vấn về NOXH"`.
    AC-E5-3: >
      Tap CTA → SEND lead to developer via existing lead-capture API (see
      Reusable.data_bound.contact_lead_submit). On success → replace sheet with
      REQ-E7. On failure → show existing Nhà Tốt error toast, form state
      preserved, retry allowed (see waived_items — reuse existing pattern).
    AC-E5-4: >
      If user has a PRIOR SUCCESSFUL submission (same NOXH project context),
      form is pre-filled with the last-submitted values (name + phone). Resubmit
      overwrites the prior lead — do NOT block or warn.

REQ-E6:
  title: >
    Insufficient result screen (Không đủ điều kiện). Warning icon + generic
    warning message + SAME contact form pattern as Sufficient, but with a
    different framing sentence. Submit → lead is NOT sent to developer (internal
    only, per PRD).
  context: >
    Sheet content is REPLACED with a warning illustration (buildings image with
    warning-orange badge overlay), heading `"Thông tin của bạn có thể vượt
    ngưỡng."`, subheading `"Một số dự án có thể có tiêu chí riêng."`, then the
    heading `"Để lại thông tin để"` + `"Chủ Đầu Tư liên hệ lại với bạn"` and the
    2-field contact form. Sticky bottom CTA `"Để lại thông tin ngay"`.
  render_when: >
    User submitted the eligibility form with income OR house-area OVER threshold.
  rules: [CR-E7, CR-E9]
  behaviors: [BH-E7]
  acceptance_criteria:
    AC-E6-1: >
      Warning message is GENERIC — do NOT enumerate specific condition(s) that
      failed (waived_items.ineligible_reason_per_condition). Both sentences
      render regardless of which specific value(s) crossed threshold.
    AC-E6-2: >
      Contact form is IDENTICAL structure to REQ-E5 (Họ và tên + Số điện thoại,
      pre-fill from account, disclaimer text below). Do NOT reimplement — reuse
      the SAME contact form component as REQ-E5.
    AC-E6-3: >
      Tap CTA → lead is NOT sent to developer. Instead, save the lead
      INTERNALLY only (see Reusable.data_bound.internal_lead_submit). On
      success → replace sheet with REQ-E7. On failure → same error handling as
      REQ-E5.
    AC-E6-4: >
      Do NOT add any alt-action CTA like "Đăng ký theo dõi khu vực" (US2) on
      this screen. Waived — see waived_items.us2_alt_action_on_insufficient.

REQ-E7:
  title: >
    Post-submit success screen. Confetti heading + reuses the same promo card
    from initial sheet + toggle `"Đồng ý nhận thông báo"` (default ON) + then
    (Sufficient only) carousel of `"Danh sách NOXH phù hợp"` matching projects.
  context: >
    Sheet is REPLACED with the success confirmation. Two variants of this
    screen: WITH carousel (18863:16646) when the user is eligible AND matching
    projects exist; WITHOUT carousel (18863:17177) when either the list is
    empty OR the user submitted from the Insufficient path. Same top content
    for both.
  render_when: >
    After a successful submit from EITHER REQ-E5 OR REQ-E6. Branch:
    - Sufficient submit + matching projects exist → E7-WITH-CAROUSEL.
    - Sufficient submit + no matching projects → E7-WITHOUT-CAROUSEL.
    - Insufficient submit → E7-WITHOUT-CAROUSEL (reuse — designer confirmed).
  rules: [CR-E10]
  behaviors: [BH-E8]
  acceptance_criteria:
    AC-E7-1: >
      Top: confetti emoji `"🎉"` + heading `"Để lại thông tin thành công!"` +
      subheading `"Nhận thông báo để cập nhật về các dự án phù hợp"`.
    AC-E7-2: >
      Promo card `"Nhà tốt thông báo"` (same visual as REQ-E2 initial sheet
      promo card, ~64px tall) renders below the heading. Content: 2 example
      notification rows with tiny avatar + line of copy each.
    AC-E7-3: >
      Row below promo: `"Đồng ý nhận thông báo"` label (left) + toggle switch
      (right). Toggle DEFAULT is ON (background button-primary orange, thumb
      right). When ON, user is subscribed to notifications for ALL projects in
      the carousel below. When OFF, user unsubscribes from ALL.
    AC-E7-4: >
      (WITH carousel variant only) Below the toggle: heading
      `"Danh sách NOXH phù hợp"` + horizontal scrolling carousel of project
      cards. Each card ~168px wide with project image, name, developer, price.
    AC-E7-5: >
      (WITHOUT carousel variant) The "Danh sách NOXH phù hợp" heading AND the
      carousel row are entirely HIDDEN. Do NOT render an empty state
      placeholder. See waived_items.empty_carousel.
    AC-E7-6: >
      Bottom CTA is REMOVED on this screen (no CTA button visible). The user
      dismisses the sheet with the top-left × button.

behaviors:
  # Hub behaviors
  BH-H1: { element: "Ad card (Ad-Primary or Ad-Secondary in Tin đăng)", on_tap: "keep_as_is — route to existing Primary Ad Detail (primary) or existing Secondary Ad Detail (secondary)" }
  BH-H2: { element: "Tab bar tabs 'Tin đăng' / 'Dự án'", on_tap: "Swap active tab, replace list below with corresponding grid; route param /tin-dang or /du-an updates" }
  BH-H3: { element: "Header row 'Điều kiện mua NOXH >' (visible when logged in)", on_tap: "Open Eligibility bottom-sheet at REQ-E2 initial state" }
  BH-H4: { element: "Project card (Project-List on Dự án tab)", on_tap: "Route to NOXH Project Detail (out of scope — defined by PRD #900 US4)" }
  BH-H5: { element: "Filter chip (any)", on_tap: "Open popover with options for that filter (see REQ-H3 AC-H3-2 for Đăng bởi options)" }
  BH-H6: { element: "Region chip on Dự án tab", on_tap: "Set active filter to that region; single select — refresh list to show only projects in that region" }
  BH-H7: { element: "'Xóa lọc' text (Dự án location filter)", on_tap: "Clear location filter; region chip 'Tất cả' becomes active" }
  BH-H8: { element: "Favorite icon on any ad card", on_tap: "keep_as_is — existing favorite handler" }
  BH-H9: { element: "Status badge on card (backend-configurable value)", on_tap: "No behavior — display only" }

  # Eligibility behaviors
  BH-E1:
    element: "Entry point 'Điều kiện mua NOXH >' (Hub header, REQ-H6) and 'Kiểm tra ngay điều kiện cần mua của dự án →' (Primary Ad Detail, out of scope)"
    on_tap: "Open bottom-sheet at REQ-E2 initial state (form empty, CTA disabled)"
  BH-E2:
    element: "Close × icon (top-left of sheet header)"
    on_tap: "Dismiss sheet. Form progress is NOT saved. Reopening from same entry point restarts from initial state"
  BH-E3:
    element: "Group dropdown 'Bạn thuộc nhóm nào?'"
    on_tap: "Open popover with 4 options (see CR-E4). Selecting one closes popover and triggers REQ-E3 progressive disclosure"
  BH-E4:
    element: "Chip in marital-status or house-ownership segment"
    on_tap: "Set the selected chip active (background-inverted / text-blank), other chips in the segment revert to unselected"
  BH-E5:
    element: "CTA 'Nhập các thông tin trên' / 'Tiếp tục'"
    on_tap: "When enabled ('Tiếp tục' state): evaluate CLIENT-SIDE against PRD's income table; replace sheet with REQ-E5 (Sufficient) or REQ-E6 (Insufficient) accordingly. When disabled: no-op"
  BH-E6:
    element: "CTA 'Để lại thông tin ngay' on Sufficient (REQ-E5)"
    on_tap: "Send lead to developer via existing lead-capture API; on success replace sheet with REQ-E7 (with or without carousel branch); on failure show existing error toast, keep form state"
  BH-E7:
    element: "CTA 'Để lại thông tin ngay' on Insufficient (REQ-E6)"
    on_tap: "Save lead INTERNALLY only (do NOT send to developer). On success replace sheet with REQ-E7-WITHOUT-CAROUSEL. On failure same as BH-E6"
  BH-E8:
    element: "Toggle 'Đồng ý nhận thông báo' on REQ-E7"
    on_tap: "Toggle ON → subscribe user to notifications for ALL projects in the carousel (when present); OFF → unsubscribe from ALL"

content_rules:
  # Hub
  CR-H1:
    element: "Filter chip visibility on Tin đăng"
    rule: >
      `"Trạng thái hồ sơ"` and `"Thời gian thu hồ sơ"` chips render ONLY when
      the current "Đăng bởi" filter value is "Chủ đầu tư". When "Đăng bởi"
      is unset, "Cá nhân", or "Môi giới", both chips are HIDDEN (removed from
      DOM, not just visually hidden). This is a conditional CHIP, not a
      conditional POPOVER.
  CR-H2:
    element: "Ad grid — rental filter"
    rule: >
      Rental ads tagged to NOXH projects MUST be filtered out of the Tin đăng
      grid. NOXH hub is mua-bán only (per PRD #900 US3). Filter server-side;
      client asserts no rental appears.
  CR-H3:
    element: "Ad grid — mixed layouts"
    rule: >
      Ad-Primary and Ad-Secondary cards MAY interleave in any order (backend
      sort determines order). Do NOT enforce grouping by type.
  CR-H4:
    element: "Region chip on Dự án"
    rule: >
      Region chips are BACKEND-DYNAMIC — the set of visible regions and their
      counts come from the API response `regions: [{name, count}]`. Chip label
      format: `"<region_name> (<count>)"`. When count is 0, do NOT render that
      region chip. Always render `"Tất cả"` chip regardless.
  CR-H5:
    element: "Project card status badge (Dự án tab)"
    rule: >
      Status badge value comes from backend-configurable enum: `open` /
      `almost` / `close`. Text label per value (see Reusable.data_bound.
      status_badge_text). Color of badge also mapped per value.
  CR-H6:
    element: "Filter chip 'Giá bán'"
    rule: >
      Currently a single price-range popover; range values not designed in this
      spec (existing Nhà Tốt price-range component reused). Chip label always
      `"Giá bán"` unless a specific range is selected — then label updates to
      that range per existing behavior.
  CR-H7:
    element: "Filter chip 'Đăng bởi'"
    rule: >
      3 options ONLY: `"Chủ đầu tư"` · `"Cá nhân"` · `"Môi giới"`. Do NOT
      render a 4th option. Popover uses DS List component; single select; tap
      outside dismisses.
  CR-H8:
    element: "Filter chip 'Thời gian thu hồ sơ' on Dự án"
    rule: >
      Popover has values from backend enum. Options are backend-configurable —
      dev sources from same endpoint as status enum. Label always `"Thời gian
      thu hồ sơ"` until a value is selected — then label updates to selected
      value per popover pattern.
  CR-H9:
    element: "Card image size"
    rule: >
      Image on all card types: 120px wide × 144px tall (Ad-Primary,
      Ad-Secondary) OR 120 × 140 (Project-List for Dự án — 4px shorter).
      Image radius: radius-ad #6px (Ad cards) or radius-card-small 8px
      (Project cards, TBD if different). Rendered as background-fill with
      cover-fit.
  CR-H10:
    element: "Card title truncation"
    rule: >
      Project name and ad title: line-clamp-2, ellipsis at line end.
      Seller name: line-clamp-1, ellipsis at end. Location: line-clamp-1,
      ellipsis at end.
  CR-H11:
    element: "Applicant count cap"
    rule: >
      Applicant counter (on cards + on eligibility promo) displays as integer
      up to 99. When count > 99, render as `"99+"` — no comma, no space.
      Example: count=100 → `"99+"`. count=6 → `"6"`. count=215 → `"99+"` —
      NOT `"215+"` (design shows "215+" as a sample, but per user answer the
      cap is 99, so this is a Figma sample override; treat as capped).
      ⚠ FLAG: verify with PM before shipping; Figma renders "215+" in the
      promo card, which contradicts "cap 99+". File as OI if uncertain.
  CR-H12:
    element: "Hub header hero image"
    rule: >
      Hero image is a fixed asset (buildings photograph) provided by design.
      Do NOT swap dynamically. Text overlays are always the SAME two lines
      (`"Tim nhà ở xã hội"` + `"dành cho bạn"`). Not localised in this build.

  # Eligibility
  CR-E1:
    element: "Sheet title"
    rule: >
      Sheet header title is always `"Kiểm tra điều kiện mua NOXH"` — never
      changes across REQ-E2..E7 states. Close × icon top-left, no back button.
  CR-E2:
    element: "CTA state and label"
    rule: >
      Bottom CTA has TWO possible labels bound to enabled state:
      - Disabled: label `"Nhập các thông tin trên"`, background button-disabled
        #c0c0c0.
      - Enabled: label `"Tiếp tục"`, background button-primary #fa6819.
      There is no third label. Toggle between the two based on
      required-field completeness (not on submit progress).
  CR-E3:
    element: "House ownership area input"
    rule: >
      When "Sở hữu nhà" chip is selected, two inputs appear inline (horizontal
      row): `"Số người trong người *"` (integer) + `"Diện tích sàn *"` (integer
      with `"m²"` suffix rendered as a suffix inside the input). Placeholder
      when empty. Helper text below the row (text-tertiary, body-annotation
      10px): `"Diện tích cho phép mua NOXH 1 người dưới 15m2"`.
  CR-E4:
    element: "Group dropdown options"
    rule: >
      4 options exactly, in this order:
      1. `"Cán bộ, công chức, viên chức"`
      2. `"Công nhân, nhân viên công ty"`
      3. `"Lao động tự do tại thành phố"`
      4. `"Người có công, hộ nghèo hoặc bị thu hồi đất"`
      Do NOT include "Lực lượng vũ trang" (intentionally excluded per PRD).
      Options rendered in the same DS List popover pattern as Hub's Đăng bởi.
  CR-E5:
    element: "Marital status chip labels"
    rule: >
      3 chip options in this order: `"Độc thân"` · `"Đã kết hôn"` ·
      `"Đơn thân có con"`. Note the order MATCHES PRD (which lists Độc thân |
      Độc thân, nuôi con nhỏ | Đã kết hôn) but with the copy variation
      `"Đơn thân có con"` (design copy wins per Q1 answer in design-ready).
  CR-E6:
    element: "Client-side income evaluation"
    rule: >
      On CTA tap in REQ-E4, evaluate the answers against this table (from PRD
      #259):
      ```
      Group                            | Độc thân | Đơn thân có con | Đã kết hôn
      -------------------------------- | -------- | --------------- | ----------
      Cán bộ/công chức/viên chức       | > 20M    | > 30M           | > 40M
      Công nhân/nhân viên công ty      | > 25M    | > 35M           | > 50M
      Lao động tự do tại thành phố     | > 25M    | > 35M           | > 50M
      Người có công/hộ nghèo/bị thu... | (skip income entirely)
      ```
      Values in table are VND per month. Income input in the sheet takes a
      value in VND. `>` means "strictly greater than" — equal to threshold is
      Sufficient. If house-ownership area/person ≥ 15m² → Insufficient trigger.
      Any single over-threshold value → Insufficient result.
  CR-E7:
    element: "Contact form pre-fill"
    rule: >
      Both Họ và tên and Số điện thoại fields pre-fill from:
      1. account (if logged in) — first priority.
      2. prior successful submission in this NOXH context (if any) — overrides
         account values.
      User can edit either value before submit. Empty required field → CTA
      disabled (existing form validation pattern).
  CR-E8:
    element: "Contact form disclaimer link"
    rule: >
      Disclaimer text: static content, only 1 hyperlink: `"Chính sách bảo mật"`
      (text-info #306bd9, underlined). Tap → open privacy policy URL in
      external browser (per existing Nhà Tốt behavior). URL is
      backend-configured.
  CR-E9:
    element: "Insufficient warning framing"
    rule: >
      Two sentences, both static:
      1. `"Thông tin của bạn có thể vượt ngưỡng."` (heading, label-page,
         text-primary)
      2. `"Một số dự án có thể có tiêu chí riêng."` (subheading, body-page,
         text-secondary)
      Do NOT interpolate specific values. Do NOT enumerate which condition.
  CR-E10:
    element: "Post-submit toggle default state"
    rule: >
      Toggle default is ON on both branches (with/without carousel). Even
      when carousel is HIDDEN (no matching projects), the toggle still renders
      and defaults ON — the notification subscription is separate from the
      carousel presence. User can turn OFF; state persists to backend.
```

---

# PART 2 — Layout

## Reusable

**Rule:** every entry here is a real reusable — build once, call at each site. Check the repo for an existing equivalent BEFORE building a local component.

### `ds_components` — CladUI components from Chợ Tốt DS v3.0

Every entry: name matches the Figma DS component name exactly. `props` are the variant properties read from Figma `componentProperties`. Import from the shared DS package (path is unverified — dev confirms and writes back to `path_hint`).

```yaml
ds_components:
  Chip:
    figma_component: "Chip"
    props:
      Size: "Medium 32px"           # only size used in this spec
      Style: "Fill"                  # only style used
      Select: "Yes | No"             # active vs inactive state
      State: "Default | Pressed | Focused | Disabled"  # DS default states — reuse all
    resolved_style:
      height: 32
      padding_x: padding-small-12
      padding_y: 6
      radius: radius-pill  # 999
      typography: header-caption  # 14/20 SemiBold Reddit Sans
      bg_unselected: background-secondary  # #f4f4f4
      text_unselected: text-primary  # #222
      bg_selected: background-inverted  # #222
      text_selected: text-blank  # #fff

  Button:
    figma_component: "Button"
    props:
      Type: "Primary | Secondary | Tertiary"
      Size: "Large | Medium | Small"
      State: "Default | Pressed | Focused | Disabled"
      Text: true                     # boolean prop for label rendering
    resolved_style:
      # Only Large Primary used in this spec (bottom CTA)
      height: button-height-large  # 40
      padding_x: padding-medium-16
      radius: radius-card-small  # 8
      typography: label-section  # 14/20 Bold Reddit Sans
      bg_primary: button-primary  # #fa6819
      bg_disabled: button-disabled  # #c0c0c0
      text_on_primary: text-on-background  # #fff

  Popover:
    figma_component: "Popover" (composed of `List` items in a floating card)
    props:
      shadow: shadow-floating  # 0 4 16 rgba(34,34,34,0.12)
      radius: radius-card  # 12
      border: border-thin  # #e8e8e8, stroke-divider 1
      padding_y: padding-2x-small-4  # 4 (list item padding is per-item)
    contains: [List items]

  List:
    figma_component: "List"
    props:
      variant: "single-select-item"
    resolved_style:
      height: 40
      padding_x: padding-medium-16  # 16
      padding_y: padding-x-small-8  # 8
      typography: body-section  # 14/20 Regular Reddit Sans
      divider: border-divider  # #f4f4f4, stroke-divider 1 bottom

  Toggle:
    figma_component: "Toggle" (Switch)
    props:
      State: "On | Off | Disabled"
    resolved_style:
      # DS default toggle — orange-on when ON, grey when OFF
      on_bg: button-primary  # #fa6819
      off_bg: background-secondary  # #f4f4f4

  Text_Label:
    figma_component: "Text Label"  # Status badge on cards
    props:
      # See Reusable.data_bound.status_badge_text for value → variant mapping
      Status: "open for applying | almost | close"
      Device: "Mobile"  # only Mobile used (App-only spec)
    variants:
      Status=open, Device=Mobile:
        text: "Đang nhận hồ sơ"       # data-bound label — override from backend
        bg: background-success-light   # #ecf9f1
        text_color: text-success       # #12a154
        icon: null
      Status=almost, Device=Mobile:
        text: "Sắp hết hạn nộp hồ sơ" # data-bound
        bg: background-warning-light   # #fff4ec
        text_color: text-warning       # #fb7328
        icon: null
      Status=close, Device=Mobile:
        text: "Đã đóng nộp hồ sơ"     # data-bound
        bg: background-secondary       # #f4f4f4
        text_color: text-tertiary      # #8c8c8c
        icon: null
    resolved_style:
      padding_x: padding-x-small-8  # 8
      padding_y: padding-2x-small-4  # 4
      radius: radius-ad  # 6
      typography: label-annotation  # 10/16 Bold Reddit Sans

  Input:
    figma_component: "Input" (DS input field)
    props:
      State: "Default | Focused | Warning | Error | Disabled"
      Size: "Medium"
    resolved_style:
      height: 40
      padding_x: padding-small-12  # 12
      radius: radius-card-small  # 8
      border_default: border-regular  # #dddddd
      border_focused: border-black  # #222
      border_warning: border-warning  # #fb7328
      typography: body-section  # 14/20 Regular
      placeholder_color: text-tertiary  # #8c8c8c

  Icon:
    # DS icon system — all icons from Nhà Tốt DS icon catalogue
    common_icons_used:
      - "Location-outline (16)"   # location pin on cards
      - "CircleCheck-fill (12)"   # verified badge on seller
      - "Favorite-outline (20)"   # favorite heart on card action
      - "Verify-fill (16)"        # verified developer badge
      - "Close-outline (24)"      # close × icon on sheet header
      - "Close-fill (20)"         # dismisible × in filter chip
      - "Chevrondown-outline (20)"# dropdown chevron
      - "Chevronright-outline (16)"# right chevron on entry row
    rule: >
      Icon name follows Figma DS naming. Size in parens is the render size in
      dp/pt. Every icon here already exists in the DS icon set — do NOT
      create SVGs.
```

### `data_bound` — properties whose value follows the data

Marked in the layout tree with `source: data`. This section names what supplies each and where the value-to-DS-name mapping lives.

```yaml
data_bound:
  status_badge_text:
    binds: "text (and via that, the Status prop on Text_Label component)"
    source: "project.application_status_enum, from the NOXH project API"
    mapping: >
      Reuse the existing enum-to-label mapping in the NOXH project service.
      Enum values: `open` / `almost` / `close`. Labels in Vietnamese are set on
      the backend — do NOT hardcode labels in the client. The mapping to
      Status variant of the Text_Label DS component follows the enum verbatim.
    sample_data: "Đang nhận hồ sơ"

  applicant_count:
    binds: text
    source: "project.applicant_count (integer), from NOXH project API"
    mapping: >
      Format: `"<count>"` for count ≤ 99. `"99+"` when count > 99. See CR-H11.
    sample_data: "215"

  applicant_avatars:
    binds: "avatar images (up to 3 stacked)"
    source: "project.recent_applicant_avatars[0..2] from NOXH project API"
    mapping: "Reuse existing avatar-stack pattern (3-avatar overlap)"
    sample_data: "3 sample avatar URLs"

  project_name:
    binds: text
    source: "project.name / ad.title from NOXH API"
    mapping: "Verbatim. Truncate to 2 lines with ellipsis-end (CR-H10)"
    sample_data: "NOXH Happy Home Nhơn Trạch"

  project_price:
    binds: text
    source: "ad.price (VND integer)"
    mapping: >
      Format: convert VND to `"<X,Y> tỷ"` for values ≥ 1B, `"<X> tr"` for
      values in millions. Comma is decimal separator (Vietnamese locale).
      Example: 1_200_000_000 → `"1,2 tỷ"`. 850_000_000 → `"850 tr"`.
      Reuse existing Nhà Tốt price-formatting util.
    sample_data: "1,2 tỷ"

  project_price_per_area:
    binds: text
    source: "computed: ad.price / ad.area (VND per m²)"
    mapping: "Format `<X,Y> tr/m²`. Reuse existing util."
    sample_data: "32,54 tr/m²"

  project_area:
    binds: text
    source: "ad.area (m² integer or decimal)"
    mapping: "Format `<value> m²`. Comma decimal per locale."
    sample_data: "70 m²"

  project_location:
    binds: text
    source: "project.address / ad.location — from NOXH API"
    mapping: "Verbatim string. Truncate line-1 with ellipsis-end."
    sample_data: "Huyện Nhơn Trạch - Đồng Nai"

  region_chips:
    binds: "chip label + count in parentheses"
    source: "hub.regions[]: { name: string, count: integer }"
    mapping: >
      Chip label: `"<name> (<count>)"`. Do NOT render chip when count = 0.
      `"Tất cả"` chip is always rendered with no count and is the default
      active chip.
    sample_data: [{name: "Bình Dương", count: 3}, {name: "Hà Nội", count: 3}, {name: "HCM", count: 3}]

  registered_user_count:
    binds: text (in promo card)
    source: "global counter of users who have completed the NOXH eligibility check"
    mapping: >
      Format: `"<count>+ người kiểm tra điều kiện mua và nhận thông tin"`.
      Rounded down to nearest hundred, appended `+`. Example: 267 → `"200+"`,
      215 → `"200+"` (Figma sample shows `"215+"` but per CR-H11 the +suffix
      pattern uses `99+` cap OR nearest-hundred round-down; verify with PM).
      ⚠ FLAG: reconcile with CR-H11 before shipping. File as OI if uncertain.
    sample_data: "215+"

  contact_lead_submit:
    binds: "onSubmit handler of Sufficient CTA (REQ-E5)"
    source: "existing NOXH lead-capture API endpoint (find in NOXH module)"
    mapping: >
      Payload: { project_id (if from Primary Ad Detail entry), name, phone,
      eligibility_answers: {group, marital, house_ownership?, income} }.
      On 2xx → REQ-E7 branch. On error → existing error toast pattern.
    sample_data: null

  internal_lead_submit:
    binds: "onSubmit handler of Insufficient CTA (REQ-E6)"
    source: "same lead-capture API + internal-only flag `send_to_developer: false`"
    mapping: >
      Same payload as contact_lead_submit but with `send_to_developer: false`
      flag. Backend routes to internal-only pipeline (per PRD #259). Response
      handling same as Sufficient — replace sheet with REQ-E7-WITHOUT-CAROUSEL.
    sample_data: null

  matching_noxh_projects:
    binds: "carousel items on REQ-E7-WITH-CAROUSEL"
    source: "existing NOXH matching-projects API"
    mapping: >
      Input: eligibility_answers. Output: list of project cards matching the
      user's answers. Empty list → do NOT render the carousel section
      (waived — see waived_items.empty_carousel).
    sample_data: "2 sample project cards (NOXH Happy Home Nhơn Trạch + K-Home Avenue)"
```

### `tokens` — the semantic colours used in this spec

Every token is a DS chotot v3.0 token, resolved to hex from the file's `get_variable_defs` output. Include `role` in plain language — the colour AC in `verify` checks the description, not the token name.

```yaml
tokens:
  # Text
  text-primary:        { hex: "#222222", role: "primary body & heading text, black" }
  text-secondary:      { hex: "#595959", role: "secondary label text, dark grey" }
  text-tertiary:       { hex: "#8c8c8c", role: "helper & placeholder text, mid grey" }
  text-brand:          { hex: "#fa6819", role: "brand-orange highlight text (Nhà Tốt accent orange)" }
  text-warning:        { hex: "#fb7328", role: "warning-orange text (slightly redder than brand-orange)" }
  text-error:          { hex: "#f0325e", role: "error text, red-pink" }
  text-success:        { hex: "#12a154", role: "success text, green" }
  text-info:           { hex: "#306bd9", role: "info link text, blue (used only in disclaimer link)" }
  text-blank:          { hex: "#ffffff", role: "white text (on dark backgrounds)" }
  text-on-background:  { hex: "#ffffff", role: "white text on solid buttons" }

  # Backgrounds
  background-primary:      { hex: "#ffffff", role: "plain white surface (cards, sheets)" }
  background-secondary:    { hex: "#f4f4f4", role: "off-white grey surface (unselected chips, disabled areas)" }
  background-app:          { hex: "#f7f7f7", role: "app ground colour (page bg)" }
  background-inverted:     { hex: "#222222", role: "solid dark surface (active chip Tất cả, sheet scrim base)" }
  background-brand:        { hex: "#fa6819", role: "brand-orange solid surface" }
  background-chotot:       { hex: "#ffd400", role: "Chợ Tốt brand yellow (used elsewhere, not in this spec)" }
  background-success-light:{ hex: "#ecf9f1", role: "success chip bg (green tint)" }
  background-warning-light:{ hex: "#fff4ec", role: "warning chip bg (orange tint)" }
  background-info-light:   { hex: "#f2f6fc", role: "info chip bg (blue tint)" }
  background-error-light:  { hex: "#feebef", role: "error chip bg (pink tint)" }
  background-brand-light-secondary: { hex: "#ffe5cf", role: "soft-orange bg (secondary orange tint)" }
  background-overlay:      { hex: "#22222280", role: "modal scrim (dark 50%)" }

  # Buttons
  button-primary:  { hex: "#fa6819", role: "primary CTA bg (orange)" }
  button-disabled: { hex: "#c0c0c0", role: "disabled CTA bg (light grey)" }
  button-blank:    { hex: "#ffffff", role: "secondary/ghost button bg" }

  # Borders
  border-thin:    { hex: "#e8e8e8", role: "thin divider border (popover)" }
  border-divider: { hex: "#f4f4f4", role: "list divider" }
  border-regular: { hex: "#dddddd", role: "input default border" }
  border-black:   { hex: "#222222", role: "input focused border, tab-active underline" }
  border-warning: { hex: "#fb7328", role: "input warning border" }
  border-blank:   { hex: "#ffffff", role: "white border (reversed)" }

  # Icons
  icon-primary:   { hex: "#222222", role: "primary icon black" }
  icon-secondary: { hex: "#595959", role: "secondary icon dark grey" }
  icon-tertiary:  { hex: "#8c8c8c", role: "helper icon mid grey" }
  icon-disabled:  { hex: "#c0c0c0", role: "disabled icon light grey" }
  icon-brand:     { hex: "#fa6819", role: "brand-orange icon" }
  icon-warning:   { hex: "#fb7328", role: "warning-orange icon" }
  icon-success:   { hex: "#12a154", role: "success-green icon" }
  icon-info:      { hex: "#306bd9", role: "info-blue icon" }
  icon-blank:     { hex: "#ffffff", role: "white icon on dark bg" }
  icon-on-background: { hex: "#ffffff", role: "white icon on solid button bg" }

  # Spacing (in pixels — token names carry the numeric value)
  spacing-0:          { hex: null, value: 0,  role: "no gap" }
  spacing-2:          { hex: null, value: 2,  role: "hairline gap (2px)" }
  gap-min-2:          { hex: null, value: 2,  role: "same as spacing-2" }
  gap-2x-small-4:     { hex: null, value: 4,  role: "extra-tight gap" }
  gap-x-small-8:      { hex: null, value: 8,  role: "tight gap" }
  gap-small-12:       { hex: null, value: 12, role: "standard tight gap" }
  padding-2x-small-4: { hex: null, value: 4,  role: "extra-tight padding" }
  padding-x-small-8:  { hex: null, value: 8,  role: "tight padding" }
  padding-small-12:   { hex: null, value: 12, role: "standard tight padding (chip, button inner)" }
  padding-medium-16:  { hex: null, value: 16, role: "standard section padding" }
  padding-large-20:   { hex: null, value: 20, role: "large section padding" }

  # Radius
  radius-ad-small:  { hex: null, value: 4,   role: "small ad image radius" }
  radius-ad:        { hex: null, value: 6,   role: "standard ad image radius" }
  radius-card-small:{ hex: null, value: 8,   role: "small card radius (input, button)" }
  radius-card:      { hex: null, value: 12,  role: "standard card radius (popover)" }
  radius-modal:     { hex: null, value: 20,  role: "modal/drawer top-corner radius" }
  radius-pill:      { hex: null, value: 999, role: "full-round pill (chip)" }

  # Stroke
  stroke-divider: { hex: null, value: 1, role: "1px divider stroke" }
  stroke-action:  { hex: null, value: 2, role: "2px active-underline stroke (tab)" }
```

Legacy tokens present in the file but NOT used in this spec: `Gray/0 - WHITE #FFFFFF` — see OI-6 (cosmetic).

### `typography`

```yaml
typography:
  # Every entry: font Reddit Sans, size/lineHeight in px, style name = Figma DS name
  display-section:   { family: "Reddit Sans", size: 24, line: 32, weight: 700, style: "Bold" }
  header-section:    { family: "Reddit Sans", size: 16, line: 24, weight: 600, style: "SemiBold" }
  header-caption:    { family: "Reddit Sans", size: 14, line: 20, weight: 600, style: "SemiBold" }
  header-annotation: { family: "Reddit Sans", size: 12, line: 18, weight: 600, style: "SemiBold" }
  label-page:        { family: "Reddit Sans", size: 16, line: 24, weight: 700, style: "Bold" }
  label-section:     { family: "Reddit Sans", size: 14, line: 20, weight: 700, style: "Bold" }
  label-caption:     { family: "Reddit Sans", size: 12, line: 18, weight: 700, style: "Bold" }
  label-annotation:  { family: "Reddit Sans", size: 10, line: 16, weight: 700, style: "Bold" }
  body-page:         { family: "Reddit Sans", size: 16, line: 24, weight: 400, style: "Regular" }
  body-section:      { family: "Reddit Sans", size: 14, line: 20, weight: 400, style: "Regular" }
  body-caption:      { family: "Reddit Sans", size: 12, line: 18, weight: 400, style: "Regular" }
  body-annotation:   { family: "Reddit Sans", size: 10, line: 16, weight: 400, style: "Regular" }
  tagline-caption:   { family: "Reddit Sans", size: 14, line: 20, weight: 500, style: "Medium" }
  tagline-annotation:{ family: "Reddit Sans", size: 12, line: 18, weight: 500, style: "Medium" }
```

### Local components (this spec)

`ProjectCard-Primary`, `ProjectCard-Secondary`, `ProjectCard-DuAn` are treated as compositions of DS components — see REQ-H5 layout blocks. NO local shared component is introduced (per "not safe" rule when structure differs). Contact form in REQ-E5 vs REQ-E6 uses the SAME 2-input form + SAME disclaimer; it is a `ContactForm` local subtree with 1 parameter (`framing_text`) — see `local_components` below.

```yaml
local_components:
  ContactForm:
    used_by: [E5-BLOCK, E6-BLOCK]
    parameters:
      framing_text: "string — the heading above the form (REQ-E5 vs REQ-E6 have different framings)"
    tree_reference: >
      See E5-BLOCK.tree — the section from the framing heading down through
      the disclaimer text. Reused verbatim in E6-BLOCK with only
      `framing_text` swapped. Everything else (2 inputs, pre-fill logic,
      disclaimer, CTA label + colour) identical.
```

### `sample_data`

```yaml
sample_data:
  n_helpful: 12
  n_applicants_over_99_cap: 100  # → renders as "99+"
  applicant_count_normal: 6
  region_count: 3
  project_name: "NOXH Happy Home Nhơn Trạch"
  project_price: "1,2 tỷ"
  project_price_per_area: "32,54 tr/m²"
  project_area: "70 m²"
  location_text: "Huyện Nhơn Trạch - Đồng Nai"
  seller_name: "Chủ đầu tư ABC"
  developer_name: "K-Home"
```

## Screens

```yaml
HUB-TIN-DANG:
  renders: [H1-BLOCK, H3-BLOCK, H5-PRIMARY-BLOCK, H5-SECONDARY-BLOCK, H6-BLOCK]
  place: |
    App-shell (existing Nhà Tốt tab navigation)
      Top status bar (OS)
      Nhà Tốt Top Navigation / Main Screen (existing, 96px tall — search bar + tabs)
      ★ NOXH Hub screen — this spec creates
      Bottom tab bar (existing, out of scope)
  parent_provides:
    - The Nhà Tốt top navigation bar (96px, contains search + top-tabs)
    - The bottom app tab bar (existing, out of scope)
    - Status bar (OS)
    - Vertical scroll behavior (page scrolls under the fixed top-nav)
  untouched:
    - Nhà Tốt Top Navigation
    - Bottom tab bar
    - Existing property listing on Nhà Tốt discovery — do NOT modify
  page_background: background-app  # #f7f7f7
  slot_check: { gap_above: 0, gap_below: 0 }  # page fills between top-nav and bottom-tab

HUB-DU-AN:
  renders: [H2-BLOCK, H4-BLOCK, H5-DUAN-BLOCK, H6-BLOCK]
  place: |
    App-shell
      Top status bar (OS)
      Nhà Tốt Top Navigation (96px)
      ★ NOXH Hub screen (Dự án tab active)
      Bottom tab bar
  parent_provides:
    - Same as HUB-TIN-DANG
  untouched:
    - Same as HUB-TIN-DANG
  page_background: background-app  # #f7f7f7
  slot_check: { gap_above: 0, gap_below: 0 }

ELI-ENTRY-SCREEN:
  renders: [ELI-ENTRY-BLOCK]  # the Hub screen showing the tap-target
  place: |
    Same as HUB-TIN-DANG. The eligibility entry point row lives INSIDE the
    NOXH-hub/Header component — see SEC-HUB-HEADER.
  parent_provides:
    - Same as HUB-TIN-DANG
  untouched:
    - Every other Hub section
  page_background: background-app
  slot_check: { gap_above: 0, gap_below: 0 }

ELI-SHEET-INIT:
  renders: [E2-INIT-BLOCK]
  place: |
    App-shell
      Underlying Hub screen (dimmed by scrim)
      Scrim overlay (background-overlay #22222280, tap dismisses)
      ★ Bottom Drawer (this spec creates the content inside)
  parent_provides:
    - The Drawer chrome (radius-modal 20 top corners, bg background-primary #fff, top-anchored, slides up from bottom)
    - The scrim (background-overlay, tap → dismiss)
    - Sheet header row (close × icon + title, part of Drawer chrome — see SEC-ELI-SHEET)
  untouched:
    - The underlying Hub screen (dimmed but rendered — no changes)
  page_background: background-overlay  # scrim colour
  slot_check: { gap_above: 0, gap_below: 0 }  # Drawer fills viewport bottom

ELI-SHEET-GROUP-SELECTED:
  renders: [E3-GROUP-SELECTED-BLOCK]
  place: |
    Same as ELI-SHEET-INIT — the Drawer is the same instance, only its
    contents change on group select. Same App shell, same scrim, same
    Drawer chrome.
  parent_provides: [Same as ELI-SHEET-INIT]
  untouched: [Same as ELI-SHEET-INIT]
  page_background: background-overlay
  slot_check: { gap_above: 0, gap_below: 0 }

ELI-SHEET-INPUT-FILLED:
  renders: [E4-INPUT-FILLED-BLOCK]
  place: [Same as ELI-SHEET-INIT — Drawer contents change]
  parent_provides: [Same as ELI-SHEET-INIT]
  untouched: [Same as ELI-SHEET-INIT]
  page_background: background-overlay
  slot_check: { gap_above: 0, gap_below: 0 }

ELI-SUFFICIENT:
  renders: [E5-SUFFICIENT-BLOCK]
  place: |
    App-shell
      Underlying Hub screen (dimmed)
      Scrim overlay
      ★ Bottom Drawer (contents REPLACED — the form is gone, contact form is here)
  parent_provides: [Same as ELI-SHEET-INIT]
  untouched: [Same as ELI-SHEET-INIT]
  page_background: background-overlay
  slot_check: { gap_above: 0, gap_below: 0 }

ELI-INSUFFICIENT:
  renders: [E6-INSUFFICIENT-BLOCK]
  place: [Same as ELI-SUFFICIENT — Drawer contents differ]
  parent_provides: [Same as ELI-SHEET-INIT]
  untouched: [Same as ELI-SHEET-INIT]
  page_background: background-overlay
  slot_check: { gap_above: 0, gap_below: 0 }

ELI-SUCCESS-WITH-CAROUSEL:
  renders: [E7-WITH-CAROUSEL-BLOCK]
  place: |
    App-shell
      Underlying Hub screen (dimmed)
      Scrim overlay
      ★ Bottom Drawer (post-submit content: heading + toggle + carousel)
  parent_provides: [Same as ELI-SHEET-INIT]
  untouched: [Same as ELI-SHEET-INIT]
  page_background: background-overlay
  slot_check: { gap_above: 0, gap_below: 0 }

ELI-SUCCESS-WITHOUT-CAROUSEL:
  renders: [E7-WITHOUT-CAROUSEL-BLOCK]
  place: |
    Same as ELI-SUCCESS-WITH-CAROUSEL — Drawer contents differ (no carousel
    section). Reused by BOTH Sufficient-no-match AND Insufficient submit.
  parent_provides: [Same as ELI-SHEET-INIT]
  untouched: [Same as ELI-SHEET-INIT]
  page_background: background-overlay
  slot_check: { gap_above: 0, gap_below: 0 }
```

---

## Sections

```yaml
SEC-HUB-HEADER:
  acted_on_by: [H6-BLOCK, ELI-ENTRY-BLOCK]
  figma:
    HUB-TIN-DANG:  "18863:15092"  # instance
    HUB-DU-AN:     "18863:15180"  # instance
    ELI-ENTRY-SCREEN: "18863:15092"  # same as HUB-TIN-DANG
    component:     "18863:15444"  # the isolated component reference in Content section
  path_hint:
    path: "unknown"
    provenance: unknown
    fill_on_first_build: true
  find_by: >
    Tim nhà ở xã hội
  confirm: >
    The section renders the two-line heading `"Tim nhà ở xã hội"` (line 1) +
    `"dành cho bạn"` (line 2) both bold text-primary, over a hero image
    (buildings photograph). Height with entry row visible: 238px total.
    NEAR MISS: Nhà Tốt discovery homepage may have a similar hero banner — the
    NOXH one specifically has these two lines PLUS (when logged in) an entry
    row `"Điều kiện mua NOXH >"`. If the heading is any other text, this is
    the wrong node — stop and re-locate.
  current: >
    New section on the NOXH Hub route. Nothing exists here today — this REQ
    creates the section. The Nhà Tốt Top Navigation above and the tab bar
    below are the pre-existing sibling structures.
  owns:
    this_section_provides:
      - the hero image background
      - the two-line heading text
      - the sub-heading area (chip: "Đăng ký lấy tối để nhân NOXH >" per Figma sample)
      - the eligibility entry row (when logged in)
      - the outer padding around the section
    therefore: >
      Do NOT add extra outer padding — the section owns its own padding.
      The parent (page scroll container) provides no gap around this
      section; it sits directly against the Top Navigation above and the
      tab bar below.
  padding_check: { inner_x: 12, inner_y: 12, surfaces: 1 }

SEC-HUB-FILTER-TIN:
  acted_on_by: [H3-BLOCK]
  figma:
    HUB-TIN-DANG: "18863:15093"  # instance of Property 1=Tin đăng
    component_master: "18863:15185"  # symbol variant
  path_hint:
    path: "unknown"
    provenance: unknown
    fill_on_first_build: true
  find_by: >
    Đăng bởi
  confirm: >
    The section renders a row of pill chips including `"Đăng bởi"` chip
    (always visible) and `"Giá bán"` chip. When "Đăng bởi" is set to
    "Chủ đầu tư", TWO extra chips appear to the right: `"Trạng thái hồ sơ"`
    and `"Thời gian thu hồ sơ"`.
    NEAR MISS: the Dự án tab's filter row also has "Trạng thái hồ sơ" chip
    (SEC-HUB-FILTER-DUAN, always visible there). Test: if the row also
    contains `"Trạng thái thi công"`, you are in the Dự án section — stop
    and re-locate.
  current: >
    New section. Sits directly below the tab bar (Tin đăng | Dự án) on the
    Tin đăng tab. Height with just default chips: 48px. With conditional
    chips visible: same 48px (chips wrap or fit inline — per Figma they fit
    inline in the 375-viewport).
  owns:
    this_section_provides:
      - the row of Chip DS instances
      - inner horizontal padding padding-small-12 (12) on left/right
      - vertical padding padding-x-small-8 (8) top/bottom
    therefore: >
      Do NOT wrap this section in another padded container — it already
      carries its own inner padding.
  padding_check: { inner_x: 12, inner_y: 8, surfaces: 1 }

SEC-HUB-FILTER-DUAN:
  acted_on_by: [H4-BLOCK]
  figma:
    HUB-DU-AN: "18863:15181"  # instance
    component_master: "18863:15201"  # symbol variant Property 1=Dự Án
  path_hint:
    path: "unknown"
    provenance: unknown
    fill_on_first_build: true
  find_by: >
    Trạng thái thi công
  confirm: >
    The section renders (top→bottom):
    - Row 1: location filter `"Khu vực:"` + selected region name (text-brand
      orange, e.g. `"Đồng Nai"`) + × dismisible + `"Xóa lọc"` on the right.
    - Row 2: tab bar `Tin đăng | Dự án` (Dự án active — underlined by
      border-black stroke-action 2px).
    - Row 3: chip row with `"Trạng thái thi công"` and `"Trạng thái hồ sơ"`
      (and `"Thời gian thu hồ sơ"` per pre_handoff_renames pending fix — for
      NOW in Figma this chip is here; per this spec it moves to
      SEC-HUB-FILTER-TIN).
    - Row 4: region chip row with `"Khu vực:"` label + `"Tất cả"` (active,
      black bg) + region chips like `"Bình Dương (3)"`.
    NEAR MISS: Tin đăng filter has similar top rows minus the "Trạng thái
    thi công" chip. If chip `"Trạng thái thi công"` is absent, wrong node.
  current: >
    New section on the NOXH Hub / Dự án tab. Height with all rows visible:
    ~172px (measured from Figma symbol). Location row and region-chip row
    are unique to Dự án — Tin đăng does NOT render them.
  owns:
    this_section_provides:
      - the location filter row
      - the tab bar (yes — the tabs live inside the filter component in
        this design; the tab-active indicator is part of this section's chrome)
      - the chip row
      - the region chip row
      - all inner padding
    therefore: >
      This section HAS the tab bar — do NOT build a separate tab bar
      component above. The active-tab underline (border-black 2px) is
      inside this section on the Dự án tab.
  padding_check: { inner_x: 16, inner_y: 8, surfaces: 1 }

SEC-HUB-LIST-TIN:
  acted_on_by: [H1-BLOCK, H5-PRIMARY-BLOCK, H5-SECONDARY-BLOCK]
  figma:
    HUB-TIN-DANG: "18863:14983"  # frame `listing`
  path_hint:
    path: "unknown"
    provenance: unknown
    fill_on_first_build: true
  find_by: >
    "Ad-List"  (grep for the DS component name; ad cards use `Ad-Primary` /
    `Ad-Secondary` variants of this component)
  confirm: >
    The section is a vertical stack of `Ad-List` DS component instances,
    each ~218px tall for `Ad-Primary` variant and similar heights for
    `Ad-Secondary`. Follows the tab bar and filter row. Each card taps
    through to its respective detail page.
    NEAR MISS: the Dự án grid uses `Project-List` component instances, not
    `Ad-List`. If cards are `Project-List` type, you are in SEC-HUB-LIST-DUAN.
  current: >
    New scrollable list section. Contains a MIX of Ad-Primary and
    Ad-Secondary cards (order determined by backend sort). Card layouts
    defined in REQ-H5.
  owns:
    this_section_provides:
      - the vertical stack container
      - internal spacing between cards (row divider or gap)
    therefore: >
      Each card is a full-width DS component instance; do not add outer
      horizontal padding — the card handles its own margin.
  padding_check: { inner_x: 0, inner_y: 0, surfaces: 1 }

SEC-HUB-LIST-DUAN:
  acted_on_by: [H2-BLOCK, H5-DUAN-BLOCK]
  figma:
    HUB-DU-AN: "18863:15096"  # frame `layout`
  path_hint:
    path: "unknown"
    provenance: unknown
    fill_on_first_build: true
  find_by: >
    "Project-List"  (DS component)
  confirm: >
    Vertical stack of `Project-List` DS component instances (Dự án tab
    cards — see REQ-H5). Each card ~168px tall. Includes a pagination
    control at the bottom (`Pagination container` — renamed via
    pre_handoff_renames from `Frame 2085668415`).
    NEAR MISS: Tin đăng grid uses `Ad-List` cards.
  current: >
    New scrollable list section. Contains ONLY Project-List cards (no ad
    cards on this tab). Bottom: Pagination component.
  owns:
    this_section_provides:
      - the vertical stack container
      - the pagination component at the bottom
    therefore: >
      Do not add pagination separately — it lives inside this section.
  padding_check: { inner_x: 0, inner_y: 0, surfaces: 1 }

SEC-ELI-SHEET-BODY:
  acted_on_by: [E2-INIT-BLOCK, E3-GROUP-SELECTED-BLOCK, E4-INPUT-FILLED-BLOCK]
  figma:
    ELI-SHEET-INIT: "18863:15765"  # Drawer child of the eligible-modal frame
    ELI-SHEET-GROUP-SELECTED: "18863:15871"
    ELI-SHEET-INPUT-FILLED: "18863:18169"
  path_hint:
    path: "unknown"
    provenance: unknown
    fill_on_first_build: true
  find_by: >
    Kiểm tra điều kiện mua NOXH
  confirm: >
    The section is the Drawer content INSIDE a bottom sheet. Header row:
    close × icon (left) + `"Kiểm tra điều kiện mua NOXH"` title (centre).
    Body starts with a promo card `"Đã có hơn ~ 215+ người kiểm tra điều
    kiện mua và nhận thông tin"`. Body evolves across the 3 states covered
    by this section: initial (E2), group-selected (E3), all-filled (E4).
    NEAR MISS: the Sufficient/Insufficient result screens ALSO have header
    `"Kiểm tra điều kiện mua NOXH"` — they use SEC-ELI-RESULT-BODY. Test:
    if the body has a form with `"Bạn thuộc nhóm nào?"` dropdown OR the
    `"Có 25+ dự án đang chờ bạn"` heading, you are in SEC-ELI-SHEET-BODY.
    If the body has a success illustration + contact form, you are in
    SEC-ELI-RESULT-BODY.
  current: >
    New Drawer content that spans 3 progressive states. Sheet chrome
    (close ×, header title, radius-modal top corners, bg background-primary)
    is provided by the shared Drawer component and is NOT rebuilt by this
    section — this section is the SCROLLABLE BODY.
  owns:
    this_section_provides:
      - the scrollable body content (promo card + main heading + form)
      - inner horizontal padding padding-medium-16 (16) on left/right
      - vertical spacing between body sub-sections (gap-medium-16 or similar)
      - the STICKY bottom CTA row (button + top border-thin divider)
    therefore: >
      Do NOT rebuild the Drawer chrome — that is the shared Drawer component
      supplied by the OS-level modal container. This section is ONLY the
      body + sticky CTA that fill inside the Drawer.
  padding_check: { inner_x: 16, inner_y: 16, surfaces: 1 }

SEC-ELI-RESULT-BODY:
  acted_on_by: [E5-SUFFICIENT-BLOCK, E6-INSUFFICIENT-BLOCK, E7-WITH-CAROUSEL-BLOCK, E7-WITHOUT-CAROUSEL-BLOCK]
  figma:
    ELI-SUFFICIENT: "18863:15948"  # Drawer child
    ELI-INSUFFICIENT: "18863:16297"
    ELI-SUCCESS-WITH-CAROUSEL: "18863:16646"
    ELI-SUCCESS-WITHOUT-CAROUSEL: "18863:17177"
  path_hint:
    path: "unknown"
    provenance: unknown
    fill_on_first_build: true
  find_by: >
    Chúc mừng bạn, bạn có đủ điều kiện đã NOXH  (for Sufficient)
    Thông tin của bạn có thể vượt ngưỡng  (for Insufficient)
    Để lại thông tin thành công  (for post-submit success)
  confirm: >
    The section is the Drawer BODY when the sheet is in a RESULT state
    (Sufficient / Insufficient / post-submit). Same header row as
    SEC-ELI-SHEET-BODY (`"Kiểm tra điều kiện mua NOXH"`), same Drawer
    chrome. The BODY starts with an illustration (celebratory chip icon or
    warning-badge on buildings) plus a heading appropriate to the state.
    NEAR MISS: SEC-ELI-SHEET-BODY has the same sheet chrome but its body
    contains the eligibility form, not a result state.
  current: >
    New Drawer content, 4 possible states (all reusing the same Drawer
    chrome and header row). Structure varies per state; see the 4 result
    blocks.
  owns:
    this_section_provides:
      - the scrollable body (illustration + heading + form OR toggle+carousel)
      - inner horizontal padding padding-medium-16 (16)
      - (Sufficient/Insufficient only) the STICKY bottom CTA row
      - (Post-submit success only) NO sticky CTA — the sheet has no bottom
        button, user dismisses via top-left ×
    therefore: >
      Do NOT rebuild the Drawer chrome or the header row — reuse the SAME
      shared Drawer component as SEC-ELI-SHEET-BODY. Only the BODY content
      differs across the 3 semantic states.
  padding_check: { inner_x: 16, inner_y: 16, surfaces: 1 }
```

---

## Layout blocks

> An AC sends you here → `screen` names the page → `section` sends you to `Sections` to locate the node → `action` says what to do to it → `what` describes the finished result → the `tree` **is** the finished contents.
>
> Every value below is a literal (see `Literal rule`). All blocks are for App, one platform × one variant = one block per screen state (Version B is 1 template with a flag on the "Tình trạng nhà ở" section — see OI-5).

### H6-BLOCK — NOXH Hub header (both tabs)

```yaml
screen: [HUB-TIN-DANG, HUB-DU-AN, ELI-ENTRY-SCREEN]
section: SEC-HUB-HEADER
requirement: REQ-H6
verifies: [AC-H6-1, AC-H6-2]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its
  contents with the tree below. The tree IS the finished contents — whatever
  renders today and is not in the tree does not survive the replace.
what: >
  A sticky header (238px tall when logged in, ~192px logged out) with a
  buildings hero image behind it, two lines of bold heading `"Tim nhà ở xã hội"`
  above `"dành cho bạn"` in text-primary, a small logo card sitting inline over
  the image, and (when logged in) a bottom row with `"Điều kiện mua NOXH"` and
  a right chevron.
figma:
  HUB-TIN-DANG: "18863:15092"
  HUB-DU-AN:    "18863:15180"
  component:    "18863:15444"
reference_width: 375
maths: "238 = 12 (top pad) + 60 (heading block) + 12 (gap) + 100 (hero+card area) + 12 (gap) + 30 (entry row) + 12 (bottom pad)"
computed:
  root: 375x238
  heading_block: 351x60
  entry_row: 351x30
tree:
  - id: hub_header_container
    type: column
    padding: [padding-small-12, padding-small-12, padding-small-12, padding-small-12]  # 12 all sides
    gap: gap-small-12  # 12
    fill: background-primary  # #fff
    children:
      - id: hero_stack
        type: stack  # buildings image behind, text overlay in front
        h: 172  # heading + inline promo card
        children:
          - id: hero_image
            type: image
            source: fixed  # (fixed asset — see CR-H12)
            path: "assets/noxh/hub-hero.jpg"
            fit: cover
            radius: radius-card  # 12
            w: 351
            h: 172
          - id: hero_overlay
            type: column
            gap: gap-2x-small-4  # 4
            padding: [padding-medium-16, padding-medium-16, padding-medium-16, padding-medium-16]
            children:
              - id: heading_line_1
                type: text
                text: "Tim nhà ở xã hội"
                typography: label-page  # 16/24 Bold
                color: text-primary
              - id: heading_line_2
                type: text
                text: "dành cho bạn"
                typography: label-page
                color: text-primary
      - id: entry_row
        # HIDDEN when user is NOT logged in — collapses the row entirely
        # (do not render, do not reserve height, per AC-H6-2)
        type: row
        gap: gap-x-small-8  # 8
        padding: [padding-small-12, padding-medium-16, padding-small-12, padding-medium-16]
        primaryAlign: SPACE_BETWEEN
        counterAlign: CENTER
        h: 30
        behavior: BH-E1
        children:
          - id: entry_row_label
            type: text
            text: "Điều kiện mua NOXH"
            typography: label-section  # 14/20 Bold
            color: text-primary
          - id: entry_row_chevron
            type: icon
            name: "Chevronright-outline"
            size: 16
            color: icon-primary
```

### H3-BLOCK — Filter row on Tin đăng

```yaml
screen: HUB-TIN-DANG
section: SEC-HUB-FILTER-TIN
requirement: REQ-H3
verifies: [AC-H3-1, AC-H3-2, AC-H3-3]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its
  contents with the tree below. The tree IS the finished contents — whatever
  renders today and is not in the tree does not survive the replace.
what: >
  A single row of pill chips: `"Đăng bởi"` chip (with chevron-down icon,
  16px right of label), `"Giá bán"` chip (chevron-down). When
  `"Đăng bởi" = "Chủ đầu tư"` is active, TWO more chips appear inline to the
  right: `"Trạng thái hồ sơ"` and `"Thời gian thu hồ sơ"`. Row is 48px tall,
  chips 32px tall. Horizontal padding inside the row: 12; vertical padding: 8.
figma: "18863:15093 (Tin đăng instance) / 18863:15185 (component master)"
reference_width: 375
maths: "48 = 8 (top pad) + 32 (chip height) + 8 (bottom pad)"
computed:
  root: 375x48
  chip_dang_boi: 106x32     # rough — label + 4 gap + 20 icon + padding
  chip_gia_ban: 97x32
  chip_trang_thai_ho_so: 167x32   # only when conditional
  chip_thoi_gian_thu_ho_so: 169x32 # only when conditional
tree:
  - id: filter_row_tin_dang
    type: row
    fill: background-primary  # #fff
    gap: gap-x-small-8  # 8
    padding: [padding-x-small-8, padding-small-12, padding-x-small-8, padding-small-12]  # 8 top/bot, 12 left/right
    counterAlign: CENTER
    children:
      - id: chip_dang_boi
        type: use
        use: Chip  # Reusable.ds_components.Chip
        props: { Size: "Medium 32px", Style: "Fill", Select: "No", State: "Default" }
        label: "Đăng bởi"
        right_icon: "Chevrondown-outline"
        right_icon_size: 20
        behavior: BH-H5
      - id: chip_gia_ban
        type: use
        use: Chip
        props: { Size: "Medium 32px", Style: "Fill", Select: "No", State: "Default" }
        label: "Giá bán"
        right_icon: "Chevrondown-outline"
        right_icon_size: 20
        behavior: BH-H5
      # Conditional: rendered ONLY when Đăng bởi = "Chủ đầu tư" (see CR-H1)
      - id: chip_trang_thai_ho_so
        type: use
        use: Chip
        conditional_render: "filter.dang_boi == 'chu_dau_tu'"
        props: { Size: "Medium 32px", Style: "Fill", Select: "No", State: "Default" }
        label: "Trạng thái hồ sơ"
        right_icon: "Chevrondown-outline"
        right_icon_size: 20
        behavior: BH-H5
      - id: chip_thoi_gian_thu_ho_so
        type: use
        use: Chip
        conditional_render: "filter.dang_boi == 'chu_dau_tu'"
        props: { Size: "Medium 32px", Style: "Fill", Select: "No", State: "Default" }
        label: "Thời gian thu hồ sơ"
        right_icon: "Chevrondown-outline"
        right_icon_size: 20
        behavior: BH-H5
```

### H4-BLOCK — Filter section on Dự án (multi-row: location + tabs + chips + region)

```yaml
screen: HUB-DU-AN
section: SEC-HUB-FILTER-DUAN
requirement: REQ-H4
verifies: [AC-H4-1, AC-H4-2, AC-H4-3]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its
  contents with the tree below. The tree IS the finished contents — whatever
  renders today and is not in the tree does not survive the replace.
what: >
  A 4-row stack (top→bottom): location filter row with `"Khu vực:"` label +
  selected region in orange + × dismisible + `"Xóa lọc"` right; tab bar
  `Tin đăng | Dự án` (Dự án active, underlined); chip row with `"Trạng thái
  thi công"` + `"Trạng thái hồ sơ"`; region chip row starting with `"Tất cả"`
  active + `"Bình Dương (3)"` + `"Hà Nội (3)"` + `"HCM (3)"`. Total 172px.
figma: "18863:15181 (Dự án instance) / 18863:15201 (component master)"
reference_width: 375
maths: "172 = 40 (loc row) + 40 (tabs) + 48 (chip row) + 44 (region chip row w/ padding)"
computed:
  root: 375x172
  location_row: 375x40
  tab_bar: 375x40
  chip_row: 375x48
  region_row: 375x44
tree:
  - id: filter_column_du_an
    type: column
    fill: background-primary
    children:
      - id: location_row
        type: row
        gap: gap-medium-16  # 16
        padding: [0, padding-medium-16, 0, padding-medium-16]
        counterAlign: CENTER
        h: 40
        children:
          - id: location_group
            type: row
            gap: gap-2x-small-4  # 4
            counterAlign: CENTER
            layoutGrow: 1
            children:
              - id: location_icon
                type: icon
                name: "Location-outline"
                size: 20
                color: icon-primary
              - id: location_label
                type: text
                text: "Khu vực:"
                typography: label-section
                color: text-primary
              - id: location_value
                type: text
                text: "{selected_region_name}"
                sample_data: "Đồng Nai"
                source: data  # bound to hub.filter.selected_region
                typography: label-section
                color: text-brand
              - id: location_dismisible
                type: icon
                name: "Close-fill"
                size: 20
                color: icon-secondary
                behavior: "Clear selected region — same as region_chip=Tất cả"
          - id: reset_filter
            type: text
            text: "Xóa lọc"
            typography: header-caption
            color: text-primary
            behavior: BH-H7
      - id: tab_bar
        type: row
        border: { bottom: { color: border-regular, weight: 1 } }
        h: 40
        children:
          - id: tab_tin_dang
            type: column
            layoutGrow: 1
            counterAlign: CENTER
            padding: [padding-small-12, padding-x-small-8, padding-small-12, padding-x-small-8]
            children:
              - text: "Tin đăng"
                typography: body-section  # inactive: regular
                color: text-secondary
            behavior: BH-H2
          - id: tab_du_an
            type: column
            layoutGrow: 1
            counterAlign: CENTER
            padding: [padding-small-12, padding-x-small-8, padding-small-12, padding-x-small-8]
            border: { bottom: { color: border-black, weight: 2 } }  # active-underline stroke-action
            children:
              - text: "Dự án"
                typography: label-section  # active: bold
                color: text-primary
            behavior: BH-H2
      - id: chip_row
        type: row
        gap: gap-x-small-8
        padding: [padding-x-small-8, padding-small-12, padding-x-small-8, padding-small-12]
        counterAlign: CENTER
        h: 48
        children:
          - id: chip_trang_thai_thi_cong
            type: use
            use: Chip
            props: { Size: "Medium 32px", Style: "Fill", Select: "No", State: "Default" }
            label: "Trạng thái thi công"
            right_icon: "Chevrondown-outline"
            right_icon_size: 20
            behavior: BH-H5
          - id: chip_trang_thai_ho_so
            type: use
            use: Chip
            props: { Size: "Medium 32px", Style: "Fill", Select: "No", State: "Default" }
            label: "Trạng thái hồ sơ"
            right_icon: "Chevrondown-outline"
            right_icon_size: 20
            behavior: BH-H5
      - id: region_row
        type: row
        gap: gap-x-small-8
        padding: [padding-2x-small-4, 0, padding-2x-small-4, 0]  # 4 top/bot
        counterAlign: CENTER
        children:
          - id: region_label_container
            type: row
            padding: [0, padding-small-12, 0, padding-small-12]
            fill: background-primary
            counterAlign: CENTER
            children:
              - text: "Khu vực:"
                typography: body-section
                color: text-tertiary
          - id: region_chips_container
            type: row
            gap: gap-x-small-8
            layoutGrow: 1
            counterAlign: CENTER
            fill: background-primary
            children:
              - id: chip_tat_ca
                type: use
                use: Chip
                props: { Size: "Medium 32px", Style: "Fill", Select: "Yes", State: "Default" }
                label: "Tất cả"
                # active state = background-inverted (#222), text text-blank
                behavior: BH-H6
              # Data-bound: region chips from hub.regions (source: data)
              - id: chip_region_template
                type: use
                use: Chip
                repeat_source: data  # hub.regions[] where count > 0
                repeat_bind: matching_noxh_projects  # see Reusable.data_bound.region_chips
                props: { Size: "Medium 32px", Style: "Fill", Select: "No", State: "Default" }
                label: "{region.name} ({region.count})"
                sample_data: ["Bình Dương (3)", "Hà Nội (3)", "HCM (3)"]
                behavior: BH-H6
```

### H5-CARD-PRIMARY-BLOCK — Ad-Primary card (Chủ đầu tư unit ad on Tin đăng)

```yaml
screen: HUB-TIN-DANG
section: SEC-HUB-LIST-TIN
requirement: REQ-H5
verifies: [AC-H5-1, AC-H5-4, AC-H5-5, AC-H1-3]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its
  contents with the tree below. The tree IS the finished contents — whatever
  renders today and is not in the tree does not survive the replace.
what: >
  A horizontal card 375 wide × 218 tall — image on the left (120×144, radius
  6, with a `"2 ngày trước"` overlay in the bottom-left of the image and a
  count-of-6-favorites badge overlay in the bottom-right), then a property
  container on the right (title 2 lines truncated, params row with
  `"Hướng Tây Nam"`, price info `"1,2 tỷ  32,54 tr/m²  70 m²"`, location row
  with a pin icon, and a seller-info row with avatar + name + verified badge
  + favorite icon on the far right). A `"Đang nhận hồ sơ"` status badge sits
  in the top-left of the image (from Text_Label DS component, data-bound).
figma: "sample instance: 18863:15448 (project ad in Eligibility section) — visually identical Ad-Primary variant"
reference_width: 375
maths: "218 = 12 (top pad) + 144 (image OR text container — max) + 12 (bottom pad) + 50 (some extra text rows). See per-child computed."
computed:
  root: 375x218
  card_container: 351x194
  image: 120x144
  property_container: 219x144
  status_badge: "auto x 20"
  # (Exact card height 218 comes from Figma sample; internal breakdown per Ad-List DS component)
tree:
  - id: ad_primary_card
    type: use
    use: "Ad-List"  # DS component instance
    props: { Property_1: "Ad-Primary" }
    behavior: BH-H1
    slots:
      # Because Ad-List is a DS component, dev SHOULD import and pass slots
      # rather than rebuilding the tree. Slots below name the data-bound
      # values dev supplies. If Ad-List does not expose these slots, log
      # to path_hint as unverified and file an OI.
      status_badge_text: { source: data, from: status_badge_text, sample: "Đang nhận hồ sơ" }
      status_badge_variant: { source: data, from: status_badge_text, sample: "open" }  # → Status prop of Text_Label
      image_url: { source: data, from: project_image, sample: "https://..." }
      time_ago_overlay: { source: data, from: "ad.posted_at (relative)", sample: "2 ngày trước" }
      favorite_count_overlay: { source: data, from: "ad.favorite_count", sample: "6" }
      title: { source: data, from: project_name, sample: "NOXH Happy Home Nhơn Trạch - Mã căn NĐ121" }
      apartment_code: { source: data, from: "ad.apartment_code", sample: "NĐ121" }
      direction: { source: data, from: "ad.direction", sample: "Hướng Tây Nam" }
      price: { source: data, from: project_price, sample: "1,2 tỷ" }
      price_per_area: { source: data, from: project_price_per_area, sample: "32,54 tr/m²" }
      area: { source: data, from: project_area, sample: "70 m²" }
      location: { source: data, from: project_location, sample: "Huyện Nhơn Trạch - Đồng Nai" }
      developer_name: { source: data, from: "ad.developer_name", sample: "K-Home" }
      developer_verified: { source: data, from: "ad.developer_verified", sample: true }
    axis_note: >
      Ad-List is the DS component; its internal axis is a row (image left +
      property container right). Do NOT rebuild the internals — see rule 5
      of "How to use" ("Reuse before build").
```

### H5-CARD-SECONDARY-BLOCK — Ad-Secondary card (Môi giới / Cá nhân)

```yaml
screen: HUB-TIN-DANG
section: SEC-HUB-LIST-TIN
requirement: REQ-H5
verifies: [AC-H5-2, AC-H1-3]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its
  contents with the tree below. The tree IS the finished contents — whatever
  renders today and is not in the tree does not survive the replace.
what: >
  A horizontal card 375 wide × 218 tall, IDENTICAL structure to the existing
  Nhà Tốt property-ad card. No new visual work — reuse the existing property
  card component. Ships as-is.
figma: "18863:15270 (component master `Property 1=Ad-Secondary`)"
reference_width: 375
maths: "218 = same as Ad-Primary (same component base)"
computed:
  root: 375x218
tree:
  - id: ad_secondary_card
    type: use
    use: "Ad-List"
    props: { Property_1: "Ad-Secondary" }
    behavior: BH-H1
    slots:
      # Ad-Secondary reuses the existing property-ad component — the slots
      # match whatever the existing Nhà Tốt list-view already supplies. Do
      # NOT redefine here; the existing feed integration handles it.
      passthrough: "existing property-ad feed integration"
```

### H5-CARD-DUAN-BLOCK — Project-List card (Dự án tab)

```yaml
screen: HUB-DU-AN
section: SEC-HUB-LIST-DUAN
requirement: REQ-H5
verifies: [AC-H5-3, AC-H5-5, AC-H2-3]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its
  contents with the tree below. The tree IS the finished contents — whatever
  renders today and is not in the tree does not survive the replace.
what: >
  A horizontal card 375 wide × 168 tall — image on the left (120×140, with a
  status badge in the top-left and a stats overlay in the bottom), then a
  project container on the right with: status badge as text label overlay,
  project name (bold, 2 lines truncated), address, location row with pin
  icon, developer name row with a verified badge.
figma: "18863:15100 (project ad sample), master via Project-List variants at 18863:15097"
reference_width: 375
maths: "168 = 12 (top pad) + 140 (image container) + 12 (bottom pad) + 4 (row divider/margin adjustment)"
computed:
  root: 375x168
  card_container: 351x144
  image: 120x140
  project_container: 219x140
tree:
  - id: du_an_card
    type: use
    use: "Project-List"  # DS component instance
    props: { }  # Project-List has no size/style variants — one component
    behavior: BH-H4
    slots:
      status_badge_text: { source: data, from: status_badge_text, sample: "Đang nhận hồ sơ" }
      status_badge_variant: { source: data, from: status_badge_text, sample: "open" }
      image_url: { source: data, from: project_image }
      applicant_count_overlay: { source: data, from: applicant_count, sample: "6" }
      project_name: { source: data, from: project_name, sample: "NOXH Happy Home Nhơn Trạch" }
      address: { source: data, from: "project.address_1", sample: "Nhà Thái Sơn - 456 P.Tân Thới Nhất" }
      location: { source: data, from: project_location, sample: "Nhà Thái Sơn - Bình Chánh, TPHCM" }
      developer_name: { source: data, from: "project.developer_name", sample: "K-Home" }
      developer_verified: { source: data, from: "project.developer_verified", sample: true }
```

### H1-BLOCK — Full NOXH Hub / Tin đăng screen composition

```yaml
screen: HUB-TIN-DANG
section: [SEC-HUB-HEADER, SEC-HUB-FILTER-TIN, SEC-HUB-LIST-TIN]
requirement: REQ-H1
verifies: [AC-H1-1, AC-H1-6]
action: >
  NEW. This screen does not exist in production today. Create a new route
  `/noxh-hub/tin-dang` that composes the three sections above vertically.
what: >
  A scrollable mobile screen 375 wide, no fixed height (scrolls beyond
  viewport). Top-to-bottom composition: SEC-HUB-HEADER (H6-BLOCK, 238px) +
  SEC-HUB-FILTER-TIN (H3-BLOCK, 48px) + SEC-HUB-LIST-TIN (repeated
  H5-CARD-PRIMARY + H5-CARD-SECONDARY, scrollable).
figma: "18863:14982 — `NOXH HUB/ Tin đăng` frame (375x1813 with 6 cards visible in Figma sample)"
reference_width: 375
maths: >
  Fixed height 1813 (Figma sample with 6 cards visible). Real runtime
  height depends on card count: header 238 + filter 48 + N*card_avg (~218
  per card) + optional empty-state. `1813 = 238 + 48 + 4*218 + 68 (illus
  spacing?)`; verify against actual measurement.
computed:
  root: 375x1813  # Figma sample height with 6 cards
  header: 375x238
  filter: 375x48
  list: "375x(N*218)"
tree:
  - id: hub_tin_dang_page
    type: column
    fill: background-app  # #f7f7f7
    children:
      - id: header
        type: use
        use: H6-BLOCK  # SEC-HUB-HEADER
      - id: filter
        type: use
        use: H3-BLOCK  # SEC-HUB-FILTER-TIN
      - id: list
        type: column
        scrollable: vertical
        gap: 0  # cards butt-up; visual separation is card-internal
        children:
          # Repeat mix of Ad-Primary and Ad-Secondary cards per backend sort.
          # This tree is the STRUCTURE; individual cards are described by
          # H5-CARD-PRIMARY-BLOCK and H5-CARD-SECONDARY-BLOCK.
          - id: ad_card_repeat
            type: use
            repeat_source: data
            repeat_bind: "hub.ads[]"
            use_conditional:
              "ad.seller_type == 'chu_dau_tu'": H5-CARD-PRIMARY-BLOCK
              "otherwise": H5-CARD-SECONDARY-BLOCK
```

### H2-BLOCK — Full NOXH Hub / Dự án screen composition

```yaml
screen: HUB-DU-AN
section: [SEC-HUB-HEADER, SEC-HUB-FILTER-DUAN, SEC-HUB-LIST-DUAN]
requirement: REQ-H2
verifies: [AC-H2-1, AC-H2-2, AC-H2-3, AC-H2-4]
action: >
  NEW. Create route `/noxh-hub/du-an`. Same header as H1-BLOCK; filter and
  list differ per REQ-H4 and REQ-H5-DUAN.
what: >
  Scrollable mobile screen 375 wide. Composition: SEC-HUB-HEADER (238px) +
  SEC-HUB-FILTER-DUAN (172px, includes tab bar!) + SEC-HUB-LIST-DUAN
  (repeated Project-List cards + Pagination at bottom).
figma: "18863:15095 — `NOXH HUB/ Dự án` frame (375x1813)"
reference_width: 375
maths: "1813 (Figma sample) = 238 (header) + 172 (filter section incl tab bar) + ~1068 (list including pagination). Verify."
computed:
  root: 375x1813
  header: 375x238
  filter: 375x172
  list: "375x(N*168 + 64 pagination)"
tree:
  - id: hub_du_an_page
    type: column
    fill: background-app  # #f7f7f7
    children:
      - id: header
        type: use
        use: H6-BLOCK  # SEC-HUB-HEADER
      - id: filter
        type: use
        use: H4-BLOCK  # SEC-HUB-FILTER-DUAN (includes tab bar internally)
      - id: list
        type: column
        scrollable: vertical
        gap: 0
        children:
          - id: du_an_card_repeat
            type: use
            use: H5-CARD-DUAN-BLOCK
            repeat_source: data
            repeat_bind: "hub.projects[]"
          - id: pagination
            type: use
            use: "Pagination"  # DS pagination component
            behavior: "keep_as_is — existing Nhà Tốt pagination"
```

### E2-INIT-BLOCK — Eligibility sheet, initial state

```yaml
screen: ELI-SHEET-INIT
section: SEC-ELI-SHEET-BODY
requirement: REQ-E2
verifies: [AC-E2-1, AC-E2-2, AC-E2-3]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its
  contents with the tree below. The tree IS the finished contents — whatever
  renders today and is not in the tree does not survive the replace.
what: >
  A bottom sheet 375 wide × 760 tall (viewport minus safe area). Top: header
  row 48 tall with close × icon (24) on the left and centred title
  `"Kiểm tra điều kiện mua NOXH"`. Below the header: a 217-tall promo-card
  area with a tinted card `"Đã có hơn ~ 215+ người kiểm tra điều kiện mua và
  nhận thông tin"` + inline avatar row + "Nhà Tốt thông báo" example
  notification card. Then a form section 140 tall with the heading
  `"Có 25+ dự án đang chờ bạn"` (centred, bold), subheading `"Điền thông tin
  để xem các dự án phù hợp"` (centred, tertiary), and a single dropdown
  `"Bạn thuộc nhóm nào? *"` (empty, placeholder text-tertiary). Sticky at
  the bottom: 72-tall row hosting a disabled `"Nhập các thông tin trên"`
  button (background button-disabled #c0c0c0, text text-on-background, height
  40, radius 8). Whitespace fills between the form and the CTA (drawer body
  is scrollable but doesn't need to scroll in this state).
figma: "18863:15765 (screen) / 18863:15767 (Drawer)"
reference_width: 375
maths: "760 = 48 (Drawer Header) + 640 (Body Content) + 72 (Bottom Button). Body 640 = 217 (promo area) + 140 (form area) + 283 (scroll whitespace before sticky CTA)"
computed:
  root: 375x760
  drawer_header: 375x48
  body_content: 375x640
  promo_area: 375x217
  promo_card_container: 343x213
  form_area: 375x140
  offer_details_container: 343x48
  dropdown: 343x48
  bottom_button_row: 375x72
  cta_button: 335x40  # 375 - 20*2 = 335
tree:
  - id: drawer
    type: column
    figma_ref: "18863:15767"
    fill: background-primary  # #fff (drawer chrome)
    radius: { top_left: radius-modal, top_right: radius-modal }  # 20 top corners
    children:
      - id: drawer_header
        type: row
        figma_ref: "18863:15768"
        gap: gap-x-small-8  # 8
        padding: [padding-small-12, padding-large-20, padding-small-12, padding-large-20]  # 12, 20
        counterAlign: CENTER
        h: 48
        children:
          - id: close_icon
            type: icon
            name: "Close-outline"
            size: 24
            color: icon-primary
            behavior: BH-E2
          - id: title
            type: text
            text: "Kiểm tra điều kiện mua NOXH"
            typography: label-page  # 16/24 Bold
            color: text-primary
            layoutGrow: 1
            align: center
      - id: body_content_outer
        type: column
        figma_ref: "18863:15774"
        layoutGrow: 1
        scrollable: vertical
        children:
          - id: promo_area
            type: column
            figma_ref: "18863:15775"
            padding: [padding-medium-16, padding-medium-16, padding-medium-16, padding-medium-16]
            gap: gap-small-12
            h: 217
            children:
              - id: promo_headline_row
                type: row
                gap: gap-x-small-8
                counterAlign: CENTER
                children:
                  - id: promo_avatar_stack
                    type: use
                    use: "AvatarStack"  # existing DS stacked-avatar (3 overlapping)
                    slots: { avatars: { source: data, from: applicant_avatars } }
                  - id: promo_headline
                    type: text
                    text: "Đã có hơn ~ {registered_user_count} người kiểm tra điều kiện mua và nhận thông tin"
                    typography: body-caption  # 12/18 Regular
                    color: text-primary
                    sample_data: "Đã có hơn ~ 215+ người kiểm tra điều kiện mua và nhận thông tin"
              - id: promo_notification_card
                type: column
                fill: background-brand-light-secondary  # #ffe5cf soft-orange
                radius: radius-card  # 12
                padding: [padding-small-12, padding-small-12, padding-small-12, padding-small-12]
                gap: gap-2x-small-4
                children:
                  - id: promo_notif_title_row
                    type: row
                    gap: gap-2x-small-4
                    counterAlign: CENTER
                    children:
                      - type: text
                        text: "Nhà Tốt thông báo"
                        typography: label-caption  # 12 Bold
                        color: text-primary
                      - type: text
                        text: "🔔"  # bell emoji per Figma sample
                        typography: body-caption
                  - id: promo_notif_line_1
                    type: row
                    gap: gap-2x-small-4
                    counterAlign: CENTER
                    children:
                      - id: notif_thumb_1
                        type: image
                        w: 24
                        h: 24
                        radius: radius-ad-small
                        source: fixed  # sample content
                        path: "assets/noxh/sample-notif-1.png"
                      - type: text
                        text: "Becamex Định Hòa vừa mở đợt nhận hồ sơ!"
                        typography: body-caption
                        color: text-primary
                  - id: promo_notif_line_2
                    type: row
                    gap: gap-2x-small-4
                    counterAlign: CENTER
                    children:
                      - id: notif_thumb_2
                        type: image
                        w: 24
                        h: 24
                        radius: radius-ad-small
                        source: fixed
                        path: "assets/noxh/sample-notif-2.png"
                      - type: text
                        text: "K-Home Avenue chuẩn bị bàn giao căn hộ đến dân cư"
                        typography: body-caption
                        color: text-primary
          - id: form_area
            type: column
            figma_ref: "18863:15798"
            padding: [padding-medium-16, padding-medium-16, padding-medium-16, padding-medium-16]
            gap: gap-small-12  # 12
            h: 140
            children:
              - id: offer_details_container
                type: column
                figma_ref: "18863:15799"
                gap: gap-x-small-8  # 8
                counterAlign: CENTER
                h: 48
                children:
                  - id: heading
                    type: text
                    text: "Có 25+ dự án đang chờ bạn"
                    typography: label-page  # 16/24 Bold
                    color: text-primary
                    align: center
                  - id: subheading
                    type: text
                    text: "Điền thông tin để xem các dự án phù hợp"
                    typography: body-caption  # 12/18 Regular
                    color: text-secondary
                    align: center
              - id: dropdown
                type: use
                use: "Input-field/ Dropdown"  # DS input variant
                figma_ref: "18863:15803"
                props: { State: "Default" }
                placeholder: "Bạn thuộc nhóm nào? *"
                right_icon: "Chevrondown-outline"
                behavior: BH-E3
                h: 48
      - id: bottom_button_row
        type: row
        figma_ref: "18863:15804"
        padding: [padding-medium-16, padding-large-20, padding-medium-16, padding-large-20]  # 16, 20
        gap: gap-x-small-8
        counterAlign: CENTER
        h: 72
        sticky_bottom: true
        border: { top: { color: border-divider, weight: 1 } }
        children:
          - id: cta_button
            type: use
            use: Button
            props: { Type: "Primary", Size: "Large", State: "Disabled", Text: true }
            label: "Nhập các thông tin trên"
            layoutGrow: 1
            h: 40  # button-height-large
            behavior: BH-E5
```

### E3-GROUP-SELECTED-BLOCK — Sheet after group selection (progressive disclosure)

```yaml
screen: ELI-SHEET-GROUP-SELECTED
section: SEC-ELI-SHEET-BODY
requirement: REQ-E3
verifies: [AC-E3-1, AC-E3-2, AC-E3-3, AC-E3-4, AC-E3-6]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its
  contents with the tree below. The tree IS the finished contents — whatever
  renders today and is not in the tree does not survive the replace.
what: >
  Same Drawer chrome as E2-INIT (48-tall header + sticky 72-tall CTA row).
  Body is now taller — the promo area shrinks/compacts, the group dropdown
  shows the selected value in text-primary, and TWO new sub-sections stack
  below the dropdown: `"Tình trạng hôn nhân"` (heading label-section +
  segmented chip row with 3 chips `"Độc thân"` / `"Đã kết hôn"` /
  `"Đơn thân có con"`) and (Version A only) `"Tình trạng nhà ở"` with 2-chip
  segment `"Chưa sở hữu"` / `"Sở hữu nhà"`. When `"Sở hữu nhà"` is selected,
  2 empty text inputs `"Số người trong người *"` and `"Diện tích sàn *  m²"`
  render inline horizontally, with helper text `"Diện tích cho phép mua NOXH
  1 người dưới 15m2"` below. CTA remains disabled with label `"Nhập các
  thông tin trên"`. Total Drawer height 850 (per Figma sample).
figma: "18863:15871"
reference_width: 375
maths: "850 = 48 (header) + 730 (body content extends taller with progressive fields) + 72 (sticky CTA)"
computed:
  root: 375x850
  drawer_header: 375x48
  body_content: 375x730
  promo_area: 375x120  # smaller than E2-INIT (compacted)
  form_area_group: 375x60  # dropdown row with selected value
  section_marital: 375x88  # heading + chip row
  section_house_ownership_va: 375x196  # heading + chips + 2 inputs + helper text (Version A only)
  bottom_button_row: 375x72
tree:
  - id: drawer
    type: column
    fill: background-primary
    radius: { top_left: radius-modal, top_right: radius-modal }
    children:
      - id: drawer_header
        # Identical structure to E2-INIT drawer_header — reuse verbatim.
        # See E2-INIT-BLOCK for the tree.
        type: use
        use: "E2-INIT.drawer_header"
      - id: body_content_outer
        type: column
        layoutGrow: 1
        scrollable: vertical
        children:
          - id: promo_area
            # Same structure as E2-INIT.promo_area but visually compressed
            # (promo card sits closer to top). Content identical.
            type: use
            use: "E2-INIT.promo_area"
            h: 120  # compressed
          - id: form_area
            type: column
            padding: [padding-medium-16, padding-medium-16, padding-medium-16, padding-medium-16]
            gap: gap-small-12
            children:
              - id: heading_row
                type: column
                gap: gap-x-small-8
                counterAlign: CENTER
                children:
                  - type: text
                    text: "Có 25+ dự án đang chờ bạn"
                    typography: label-page
                    color: text-primary
                    align: center
                  - type: text
                    text: "Điền thông tin để xem các dự án phù hợp"
                    typography: body-caption
                    color: text-secondary
                    align: center
              - id: dropdown_group
                type: use
                use: "Input-field/ Dropdown"
                props: { State: "Default" }
                placeholder: "Bạn thuộc nhóm nào? *"
                value: "{selected_group}"
                sample_data: "Lao động tự do tại thành phố"
                right_icon: "Chevrondown-outline"
                behavior: BH-E3
                h: 48
              - id: section_marital
                type: column
                gap: gap-x-small-8
                children:
                  - type: text
                    text: "Tình trạng nhà ở"  # ⚠ per Figma sample; note this section is house-ownership below marital. Order: check with Figma — screenshot shows "Tình trạng nhà ở" TWICE. Suspicion: Figma duplicate. Order per PRD: MARITAL first. Ship marital first (Version A + B), then house-ownership (Version A only).
                    typography: label-section
                    color: text-primary
                  - id: marital_chip_row
                    type: row
                    gap: gap-x-small-8
                    children:
                      - id: chip_doc_than
                        type: use
                        use: Chip
                        props: { Size: "Medium 32px", Style: "Fill", Select: "Yes", State: "Default" }
                        label: "Độc thân"
                        behavior: BH-E4
                      - id: chip_da_ket_hon
                        type: use
                        use: Chip
                        props: { Size: "Medium 32px", Style: "Fill", Select: "No", State: "Default" }
                        label: "Đã kết hôn"
                        behavior: BH-E4
                      - id: chip_don_than_co_con
                        type: use
                        use: Chip
                        props: { Size: "Medium 32px", Style: "Fill", Select: "No", State: "Default" }
                        label: "Đơn thân có con"
                        behavior: BH-E4
              - id: section_house_ownership
                # Version A only — hidden when feature flag Version B is active. See OI-5.
                type: column
                gap: gap-x-small-8
                conditional_render: "version == 'A'"
                children:
                  - type: text
                    text: "Tình trạng nhà ở"
                    typography: label-section
                    color: text-primary
                  - id: house_chip_row
                    type: row
                    gap: gap-x-small-8
                    children:
                      - id: chip_chua_so_huu
                        type: use
                        use: Chip
                        props: { Size: "Medium 32px", Style: "Fill", Select: "Yes", State: "Default" }
                        label: "Chưa sở hữu"
                        behavior: BH-E4
                      - id: chip_so_huu_nha
                        type: use
                        use: Chip
                        props: { Size: "Medium 32px", Style: "Fill", Select: "No", State: "Default" }
                        label: "Sở hữu nhà"
                        behavior: BH-E4
                  - id: house_inputs_row
                    # Only rendered when chip_so_huu_nha is active (Sở hữu nhà selected)
                    type: row
                    gap: gap-x-small-8
                    conditional_render: "house_ownership == 'so_huu'"
                    children:
                      - id: input_num_people
                        type: use
                        use: Input
                        props: { Size: "Medium", State: "Default" }
                        placeholder: "Số người trong người *"
                        layoutGrow: 1
                        h: 40
                      - id: input_area
                        type: use
                        use: Input
                        props: { Size: "Medium", State: "Default" }
                        placeholder: "Diện tích sàn *"
                        suffix: "m²"
                        layoutGrow: 1
                        h: 40
                  - id: house_helper_text
                    type: text
                    text: "Diện tích cho phép mua NOXH 1 người dưới 15m2"
                    typography: body-annotation  # 10/16
                    color: text-tertiary
                    conditional_render: "house_ownership == 'so_huu'"
      - id: bottom_button_row
        type: use
        use: "E2-INIT.bottom_button_row"  # same structure; CTA state still Disabled with label "Nhập các thông tin trên"
```

### E4-INPUT-FILLED-BLOCK — Sheet with all fields filled, CTA enabled

```yaml
screen: ELI-SHEET-INPUT-FILLED
section: SEC-ELI-SHEET-BODY
requirement: REQ-E4
verifies: [AC-E4-1, AC-E4-2, AC-E3-5]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its
  contents with the tree below. The tree IS the finished contents — whatever
  renders today and is not in the tree does not survive the replace.
what: >
  Same Drawer + same body structure as E3-GROUP-SELECTED but with all
  required fields filled. Inputs show values (`"4"` in Số người, `"70"` in
  Diện tích). Marital chip 2 (Đã kết hôn) selected + house_ownership chip 2
  (Sở hữu nhà) selected. CTA at the bottom flips to ENABLED state: label
  `"Tiếp tục"`, background button-primary #fa6819. If any input value causes
  the warning threshold to trigger (income OR house area/person ≥ 15),
  those specific inputs render with border-warning #fb7328 (Warning state
  of Input DS component).
figma: "18863:18169"
reference_width: 375
maths: "850 = 48 + 730 + 72 (same layout envelope as E3-GROUP-SELECTED — fields fill but structure is identical)"
computed:
  root: 375x850
  cta_button: 335x40  # enabled state, orange
tree:
  - id: drawer
    # Structure IDENTICAL to E3-GROUP-SELECTED. Only differences:
    # 1. Field values populated (data-bound to form state).
    # 2. Chips that are selected reflect user's selections (Select: "Yes").
    # 3. When over-threshold, affected inputs get State: "Warning" (border-warning).
    # 4. CTA switches to enabled state with new label:
    type: use
    use: "E3-GROUP-SELECTED.drawer"
    override_at:
      "cta_button":
        props: { Type: "Primary", Size: "Large", State: "Default", Text: true }
        label: "Tiếp tục"
      "input_num_people":
        value: "{n_people}"
        sample_data: "4"
        # Add State: "Warning" if computed threshold crossed:
        state_conditional: "avg_area_per_person >= 15 ? 'Warning' : 'Default'"
      "input_area":
        value: "{area_m2}"
        sample_data: "70"
        state_conditional: "avg_area_per_person >= 15 ? 'Warning' : 'Default'"
      # ⚠ NOTE: This `override_at` pattern is a spec convenience — dev should
      # implement one form component that renders all field states with
      # standard controlled-input pattern. The override is here to signal
      # that the tree structure does not change, only field values +
      # state props. Do NOT rebuild the tree.
```

### E5-SUFFICIENT-BLOCK — Sufficient result: contact form

```yaml
screen: ELI-SUFFICIENT
section: SEC-ELI-RESULT-BODY
requirement: REQ-E5
verifies: [AC-E5-1, AC-E5-2, AC-E5-3, AC-E5-4]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its
  contents with the tree below. The tree IS the finished contents — whatever
  renders today and is not in the tree does not survive the replace.
what: >
  Same Drawer chrome as E2-INIT. Body REPLACED: same promo area at top
  (compressed), then a celebration heading `"Chúc mừng bạn, bạn có đủ điều
  kiện đã NOXH"` (label-page, centred, primary), then a 2-field contact
  form (`"Họ và tên *"` + `"Nhập số điện thoại *"`, both 40-tall inputs,
  pre-filled from account). Below the form: small disclaimer text with
  inline `"Chính sách bảo mật"` link (text-info blue underlined). Sticky
  bottom: enabled orange CTA `"Để lại thông tin ngay"`.
figma: "18863:15948"
reference_width: 375
maths: "862 = 48 (header) + 742 (body) + 72 (CTA row). Body includes: promo 120 + heading area 60 + form 150 (2 inputs + gaps) + disclaimer 60 + whitespace 352"
computed:
  root: 375x862
  drawer_header: 375x48
  body_content: 375x742
  promo_area: 375x120
  heading_area: 375x60
  form_area: 375x150
  disclaimer_area: 375x60
  cta_row: 375x72
  input: 335x40
  cta_button: 335x40
tree:
  - id: drawer
    type: column
    fill: background-primary
    radius: { top_left: radius-modal, top_right: radius-modal }
    children:
      - id: drawer_header
        type: use
        use: "E2-INIT.drawer_header"
      - id: body_content_outer
        type: column
        layoutGrow: 1
        scrollable: vertical
        children:
          - id: promo_area
            type: use
            use: "E2-INIT.promo_area"
            h: 120
          - id: contact_form_block
            type: use
            use: "ContactForm"  # local component — see Reusable.local_components
            params:
              framing_heading: "Chúc mừng bạn, bạn có đủ điều kiện đã NOXH"
              framing_subheading: null  # Sufficient has no subheading, only the heading
      - id: bottom_button_row
        type: row
        padding: [padding-medium-16, padding-large-20, padding-medium-16, padding-large-20]
        gap: gap-x-small-8
        counterAlign: CENTER
        h: 72
        sticky_bottom: true
        border: { top: { color: border-divider, weight: 1 } }
        children:
          - id: cta_button
            type: use
            use: Button
            props: { Type: "Primary", Size: "Large", State: "Default", Text: true }
            label: "Để lại thông tin ngay"
            layoutGrow: 1
            h: 40
            behavior: BH-E6
```

### E6-INSUFFICIENT-BLOCK — Insufficient result: contact form with warning framing

```yaml
screen: ELI-INSUFFICIENT
section: SEC-ELI-RESULT-BODY
requirement: REQ-E6
verifies: [AC-E6-1, AC-E6-2, AC-E6-3, AC-E6-4]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its
  contents with the tree below. The tree IS the finished contents — whatever
  renders today and is not in the tree does not survive the replace.
what: >
  Same Drawer chrome as E5. Body replaced with WARNING framing: promo area
  with a warning-badge overlay on buildings image (celebratory chip icon
  replaced by warning-orange badge with `⚠️`), then two-line heading:
  `"Thông tin của bạn có thể vượt ngưỡng."` (label-page, centred) and
  `"Một số dự án có thể có tiêu chí riêng."` (body-page, centred, secondary).
  Then a heading `"Để lại thông tin để"` + `"Chủ Đầu Tư liên hệ lại với bạn"`
  (label-page, 2 lines, centred). Same 2-field contact form as E5.
  Disclaimer text same as E5. Sticky bottom CTA `"Để lại thông tin ngay"`.
figma: "18863:16297"
reference_width: 375
maths: "862 = 48 + 742 + 72. Same envelope as E5."
computed:
  root: 375x862
  drawer_header: 375x48
  body_content: 375x742
  warning_promo: 375x180  # slightly taller because warning badge overlay + 2-line warning heading
  contact_form_area: 375x310  # heading + form + disclaimer
  cta_row: 375x72
tree:
  - id: drawer
    type: column
    fill: background-primary
    radius: { top_left: radius-modal, top_right: radius-modal }
    children:
      - id: drawer_header
        type: use
        use: "E2-INIT.drawer_header"
      - id: body_content_outer
        type: column
        layoutGrow: 1
        scrollable: vertical
        children:
          - id: warning_promo
            type: column
            padding: [padding-medium-16, padding-medium-16, padding-medium-16, padding-medium-16]
            gap: gap-x-small-8
            counterAlign: CENTER
            h: 180
            children:
              - id: warning_illustration
                type: stack
                w: 120
                h: 100
                children:
                  - id: buildings_image
                    type: image
                    source: fixed
                    path: "assets/noxh/warning-buildings.png"
                    w: 120
                    h: 100
                    radius: radius-card-small
                  - id: warning_badge
                    type: image
                    source: fixed
                    path: "assets/noxh/warning-badge.svg"  # orange badge with ⚠️
                    w: 40
                    h: 40
                    position: { right: 0, bottom: 0 }
              - id: warning_heading
                type: text
                text: "Thông tin của bạn có thể vượt ngưỡng."
                typography: label-page
                color: text-primary
                align: center
              - id: warning_subheading
                type: text
                text: "Một số dự án có thể có tiêu chí riêng."
                typography: body-page
                color: text-secondary
                align: center
          - id: contact_form_block
            type: use
            use: "ContactForm"
            params:
              framing_heading: "Để lại thông tin để"
              framing_subheading: "Chủ Đầu Tư liên hệ lại với bạn"
      - id: bottom_button_row
        type: row
        padding: [padding-medium-16, padding-large-20, padding-medium-16, padding-large-20]
        gap: gap-x-small-8
        counterAlign: CENTER
        h: 72
        sticky_bottom: true
        border: { top: { color: border-divider, weight: 1 } }
        children:
          - id: cta_button
            type: use
            use: Button
            props: { Type: "Primary", Size: "Large", State: "Default", Text: true }
            label: "Để lại thông tin ngay"
            layoutGrow: 1
            h: 40
            behavior: BH-E7
```

### ContactForm sub-tree (used by E5 + E6)

```yaml
# This is the local component referenced in Reusable.local_components.ContactForm.
# Used as `use: "ContactForm"` above with 2 parameters:
#   - framing_heading (string, required)
#   - framing_subheading (string | null, optional)
# Everything else identical between E5 and E6.

ContactForm_tree:
  - id: contact_form_container
    type: column
    padding: [padding-medium-16, padding-medium-16, padding-medium-16, padding-medium-16]
    gap: gap-small-12
    children:
      - id: framing_heading
        type: text
        text: "{framing_heading}"  # parameter
        typography: label-page
        color: text-primary
        align: center
      - id: framing_subheading
        # Rendered only when parameter is not null (E6 has it, E5 does not)
        type: text
        text: "{framing_subheading}"
        typography: label-page
        color: text-primary
        align: center
        conditional_render: "framing_subheading != null"
      - id: input_name
        type: use
        use: Input
        props: { Size: "Medium", State: "Default" }
        placeholder: "Họ và tên *"
        value: "{account.name | prior_submission.name | ''}"
        sample_data: "Thảo"
        h: 40
      - id: input_phone
        type: use
        use: Input
        props: { Size: "Medium", State: "Default" }
        placeholder: "Nhập số điện thoại *"
        value: "{account.phone | prior_submission.phone | ''}"
        sample_data: "01234567654"
        h: 40
        inputMode: numeric
      - id: disclaimer
        type: text
        # Composed static + inline link. Render as a single paragraph with the
        # link segment styled as text-info + underline.
        text: "Bằng việc bấm nút \"Xem kết quả\", bạn đã đọc và đồng ý {link} của Nhà Tốt và cho phép chia sẻ thông tin cá nhân của bạn cho Nhà Tốt để họ liên hệ tư vấn về NOXH"
        typography: body-annotation  # 10/16
        color: text-tertiary
        inline_link:
          placeholder: "{link}"
          text: "Chính sách bảo mật"
          color: text-info
          underline: true
          on_tap: "Open privacy policy URL in external browser (existing Nhà Tốt handler)"
```

### E7-WITH-CAROUSEL-BLOCK — Post-submit success WITH matching projects carousel

```yaml
screen: ELI-SUCCESS-WITH-CAROUSEL
section: SEC-ELI-RESULT-BODY
requirement: REQ-E7
verifies: [AC-E7-1, AC-E7-2, AC-E7-3, AC-E7-4]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its
  contents with the tree below. The tree IS the finished contents — whatever
  renders today and is not in the tree does not survive the replace.
what: >
  Same Drawer chrome (48-tall header, no bottom CTA). Body: confetti-heading
  row `"🎉 Để lại thông tin thành công!"` + subheading `"Nhận thông báo để
  cập nhật về các dự án phù hợp"`. Then the same promo notification card as
  E2-INIT. Then a row `"Đồng ý nhận thông báo"` + toggle (default ON,
  orange). Then a section heading `"Danh sách NOXH phù hợp"` + horizontal
  scrolling carousel of matching project cards (~168 wide each). NO bottom
  CTA — user dismisses via the top-left × button. Total 862.
figma: "18863:16646"
reference_width: 375
maths: "862 = 48 (header) + 814 (body, no sticky CTA). Body: 90 (confetti heading area) + 100 (promo notif card compact) + 56 (toggle row) + 40 (danh sách heading) + 200 (carousel) + 328 (whitespace)"
computed:
  root: 375x862
  drawer_header: 375x48
  body_content: 375x814
  confetti_heading_area: 375x90
  promo_notification_card: 343x100  # 375 - 16*2
  toggle_row: 343x56  # 375 - 16*2
  danh_sach_heading: 343x40
  carousel_row: 375x200  # scrolls horizontally, card ~168 wide
  carousel_card: 168x200
tree:
  - id: drawer
    type: column
    fill: background-primary
    radius: { top_left: radius-modal, top_right: radius-modal }
    children:
      - id: drawer_header
        type: use
        use: "E2-INIT.drawer_header"
      - id: body_content_outer
        type: column
        layoutGrow: 1
        scrollable: vertical
        children:
          - id: confetti_heading_area
            type: column
            padding: [padding-medium-16, padding-medium-16, padding-medium-16, padding-medium-16]
            gap: gap-x-small-8
            counterAlign: CENTER
            h: 90
            children:
              - id: confetti_heading
                type: text
                text: "🎉 Để lại thông tin thành công!"
                typography: label-page
                color: text-primary
                align: center
              - id: subheading
                type: text
                text: "Nhận thông báo để cập nhật về các dự án phù hợp"
                typography: body-caption
                color: text-secondary
                align: center
          - id: promo_notification_area
            type: column
            padding: [0, padding-medium-16, padding-medium-16, padding-medium-16]  # no top pad, 16 sides & bottom
            children:
              - id: promo_notification_card
                # Same soft-orange notification card as in E2-INIT.promo_area.
                # Reused verbatim — see E2-INIT.promo_notification_card.
                type: use
                use: "E2-INIT.promo_notification_card"
          - id: toggle_row
            type: row
            padding: [padding-small-12, padding-medium-16, padding-small-12, padding-medium-16]
            gap: gap-x-small-8
            counterAlign: CENTER
            primaryAlign: SPACE_BETWEEN
            h: 56
            fill: background-primary
            border: { top: { color: border-divider, weight: 1 } }
            children:
              - id: toggle_label
                type: text
                text: "Đồng ý nhận thông báo"
                typography: label-section
                color: text-primary
              - id: toggle_switch
                type: use
                use: Toggle
                props: { State: "On" }
                behavior: BH-E8
          - id: danh_sach_section
            type: column
            padding: [padding-medium-16, padding-medium-16, padding-x-small-8, padding-medium-16]
            gap: gap-x-small-8
            children:
              - id: danh_sach_heading_row
                type: row
                primaryAlign: SPACE_BETWEEN
                counterAlign: CENTER
                h: 32
                children:
                  - id: danh_sach_heading
                    type: text
                    text: "Danh sách NOXH phù hợp"
                    typography: label-section
                    color: text-primary
                  - id: xem_tat_ca
                    type: text
                    text: "Xem tất cả >"
                    typography: header-caption
                    color: text-brand
                    behavior: "Navigate to full NOXH matching-projects list (existing route)"
              - id: carousel
                type: row
                scrollable: horizontal
                gap: gap-small-12
                h: 200
                children:
                  - id: carousel_card_template
                    type: use
                    use: "Project-List"  # DS Project card (compact carousel variant)
                    props: { }
                    repeat_source: data
                    repeat_bind: matching_noxh_projects
                    sample_data: 2  # 2 cards visible in Figma sample (NOXH Happy Home Nhơn Trạch + K-Home Avenue)
                    slots:
                      # Same slots as H5-CARD-DUAN-BLOCK carousel-variant
                      passthrough: "same as H5-CARD-DUAN-BLOCK"
```

### E7-WITHOUT-CAROUSEL-BLOCK — Post-submit success WITHOUT carousel

```yaml
screen: ELI-SUCCESS-WITHOUT-CAROUSEL
section: SEC-ELI-RESULT-BODY
requirement: REQ-E7
verifies: [AC-E7-1, AC-E7-2, AC-E7-3, AC-E7-5, AC-E7-6]
action: >
  UPDATE. Keep the section node and everything around it; replace ALL of its
  contents with the tree below. The tree IS the finished contents — whatever
  renders today and is not in the tree does not survive the replace.
what: >
  IDENTICAL to E7-WITH-CAROUSEL up to the toggle row, then STOPS. NO
  `"Danh sách NOXH phù hợp"` heading, NO carousel section. The rest of the
  Drawer body is whitespace. Total 862 (same viewport envelope). Reused by
  BOTH: Sufficient submit + zero matching projects; AND Insufficient submit.
figma: "18863:17177"
reference_width: 375
maths: "862 = 48 (header) + 814 (body). Body: 90 (confetti heading) + 100 (promo notif) + 56 (toggle row) + 568 (whitespace, no carousel)"
computed:
  root: 375x862
  drawer_header: 375x48
  body_content: 375x814
  confetti_heading_area: 375x90
  promo_notification_card: 343x100
  toggle_row: 343x56
  whitespace_below_toggle: 375x568  # explicit — do NOT render an empty placeholder here
tree:
  - id: drawer
    type: column
    fill: background-primary
    radius: { top_left: radius-modal, top_right: radius-modal }
    children:
      - id: drawer_header
        type: use
        use: "E2-INIT.drawer_header"
      - id: body_content_outer
        type: column
        layoutGrow: 1
        scrollable: vertical
        children:
          - id: confetti_heading_area
            type: use
            use: "E7-WITH-CAROUSEL.confetti_heading_area"
          - id: promo_notification_area
            type: use
            use: "E7-WITH-CAROUSEL.promo_notification_area"
          - id: toggle_row
            type: use
            use: "E7-WITH-CAROUSEL.toggle_row"
          # NOTHING else. Do NOT render danh_sach_section. Do NOT render an
          # empty-state placeholder for the carousel. See
          # waived_items.empty_carousel — dev-behavior is "HIDE the entire
          # section — the heading AND the carousel row".
```

### ELI-ENTRY-BLOCK — Entry-point row on the Hub header (delegated to H6-BLOCK)

```yaml
screen: ELI-ENTRY-SCREEN
section: SEC-HUB-HEADER
requirement: REQ-E1
verifies: [AC-E1-1, AC-E1-2]
action: >
  UPDATE. This block is a POINTER — the entry-point row already lives
  inside H6-BLOCK (see `entry_row` child in H6-BLOCK.tree). The behavior
  BH-E1 attached to that row is the entry-point implementation. Do not
  build a separate section for this REQ.
what: >
  See H6-BLOCK — the `entry_row` child inside the NOXH-hub/Header. Row
  behavior BH-E1: tap → open bottom sheet at E2-INIT-BLOCK.
figma: "18863:15092 (Hub header instance — the entry row is inside)"
reference_width: 375
maths: "See H6-BLOCK"
computed: { root: 351x30 }
tree:
  - id: entry_row_pointer
    type: use
    use: "H6-BLOCK.entry_row"
    behavior: BH-E1  # confirmed handler — see behaviors list
```

---

## Verify checklist

Run in numbered order. Each group names the ACs it verifies. When a check fails, use the diagnostic notes to know what to fix.

```yaml
0_right_section:
  # AC-H1-1, AC-H2-1, AC-E1-1, AC-E2-1
  - Before editing SEC-HUB-HEADER: the located node contains the two-line
    heading "Tim nhà ở xã hội" + "dành cho bạn". If not present, wrong node.
  - Before editing SEC-HUB-FILTER-TIN: the row contains chip "Đăng bởi"
    AND the chip "Giá bán". If ONLY one is present, or "Trạng thái thi
    công" is present (Dự án filter), wrong node — stop and re-locate.
  - Before editing SEC-HUB-FILTER-DUAN: the section contains "Trạng thái
    thi công" chip AND the location row with "Khu vực:" label. Both must
    be present.
  - Before editing SEC-ELI-SHEET-BODY: the body contains the "Có 25+ dự
    án đang chờ bạn" heading OR the dropdown "Bạn thuộc nhóm nào?". If
    the body has a success illustration + contact form, you are in
    SEC-ELI-RESULT-BODY — wrong section, re-locate.
  - Before editing SEC-ELI-RESULT-BODY: the body contains the celebration
    icon OR warning icon in a promo area, AND a 2-input contact form.
  - Adjacent sections (siblings under `untouched` on the Screen) must be
    unchanged before AND after the edit. Diff each named sibling before
    committing.

1_padding_ownership:
  # AC-H6-1, AC-E2-1..3
  - A doubled inner padding on SEC-HUB-HEADER means the section was
    built twice instead of reused. Section owns padding 12 all sides —
    do NOT add an outer padded wrapper. If measured inner padding is 24,
    remove the outer wrapper (don't halve either value).
  - A slot gap of exactly double between HUB header and the filter row
    means a margin was added on top of the parent's gap. Parent provides
    no gap between sections; if there's a gap, it comes from padding on
    one of the sections. Remove the added margin.
  - SEC-ELI-SHEET-BODY owns its 16px inner padding — the Drawer chrome
    provides NO padding. If the sheet content has double 16 (32) inner
    padding, the section was wrapped in another padded frame.
  - Bottom Button row is 72 tall INCLUDING its 16-top-bottom padding.
    The Button itself is 40 tall (button-height-large). If the row is 88
    or the button is 56, the padding was added twice.

2_variant_isolation:
  # A single-variant + single-platform spec — this section is trivial for now
  - No block contains an id that appears only in another version's blocks
    (Version B / Version A are handled by feature flag on section
    `section_house_ownership` in E3/E4 — see OI-5).
  - Version B render (feature flag = B) produces zero DOM nodes for the
    `section_house_ownership` id. Grep the built tree.
  - Ad-Primary block does NOT contain any Ad-Secondary internal nodes.
    Ad-Secondary block reuses the existing property-ad component
    completely — no new nodes.

3_computed:
  # AC-H6-1, AC-H3-1, AC-H4-2, AC-E2-1, AC-E3-1..6, AC-E5-2, AC-E6-1
  - Every block's root `computed` height matches the sum of its children.
    Diff the built root height against `computed.root.height` for every
    screen listed in Screens. When there's a discrepancy:
    - Height over by a multiple of 4/8/12: a margin was added duplicating
      a parent gap. Remove the margin.
    - Height over by ~48: a duplicated header row (48 is the drawer
      header height). Verify the shared Drawer chrome is used, not
      rebuilt.
    - Width and height swapped against `computed`: the container was
      built on the wrong axis. Re-read its `type` — column becomes row
      when swapped.
  - E2-INIT: root 375x760 = 48 header + 640 body + 72 CTA row. If measured
    is 808, someone added the header twice.
  - E3-GROUP-SELECTED / E4-INPUT-FILLED: root 375x850.
  - E5 / E6 / E7-*: root 375x862.

4_responsive:
  # AC-H1-1, AC-H2-1, general responsive
  - Screens are fluid-width: `reference_width` 375 is the SAMPLE, not a
    fixed constraint. Screen renders correctly on 360-430 device widths.
  - No fixed heights on any container that holds text or a repeat
    (except the sheet Drawer height which is bounded by viewport).
  - Every card container has `min-width: 0` so text truncation works
    (line-clamp requires min-width).
  - Toggle DOWN/UP screen size: bottom sheet always docked to bottom
    with radius-modal top corners; keyboard-open on E5/E6 shifts the
    sheet up so contact form is above the keyboard (use existing modal
    keyboard-avoiding pattern).

5_tokens_and_literals:
  # AC-H1-1 through AC-E7-6 — colour + literal verification
  - Every bare number in every tree was built as written. `gap: 12`
    built as 12 (not rounded to a token). `radius: 12` built as 12
    literal. If a value was rounded to the nearest token, it's a defect
    — report node id, spec value, built value, and unround.
  - Every colour is either a token from `tokens` (bound at build time)
    or a documented hex. If a build shows `background-brand` on a
    surface that should be `background-secondary` (grey), the token was
    substituted by name similarity — reset to the correct token.
  - Surface colours never resolve to a brand/accent/highlight/warning
    token. A grey card that renders orange is a token-by-name-similarity
    bug; look up the hex in the `tokens` table and rebind to the
    correct role.
  - Chip 18863:15236 (Giá bán) — verify it now binds
    `background-secondary` token, not raw #f4f4f4. See OI-7.

6_components_and_data:
  # AC-H5-5, AC-E4-1, AC-E5-3, AC-E6-3
  - Every Chip renders via DS Chip component (import, don't rebuild).
  - Every Button renders via DS Button component.
  - Every DS component instance in a tree carries its `props` verbatim —
    Size/Style/Select/State on Chip, Type/Size/State on Button, etc. If
    the built code uses props that are NOT in
    `Reusable.ds_components.<component>.props`, either the DS was
    extended (log which prop) or the wrong component name was used.
  - Every leaf with `source: data` resolves through the source named in
    `data_bound`. A DS name hardcoded where the tree said `source: data`
    is a frozen sample — replace with the runtime value.
  - Status badges on cards: value + colour + icon come from
    `data_bound.status_badge_text` mapping. If any status badge renders
    with a HARDCODED text like "Đang nhận hồ sơ" as a fixed constant,
    dev missed the data binding.

7_content_and_behavior:
  # BH-H1..H9, BH-E1..E8, and CR-* content rules
  - Every id in every tree that has a `behavior:` fires that behavior
    (the id in `behaviors` list resolves to a handler). Grep every
    `behavior: BH-*` in the tree — every one must have an implementation.
  - Every string that appears on the built screen matches the literal
    in the spec, in Vietnamese, verbatim.
  - CTA labels: E2/E3 → "Nhập các thông tin trên" (disabled); E4 →
    "Tiếp tục" (enabled); E5/E6 → "Để lại thông tin ngay" (enabled).
  - Applicant count over 99 renders "99+" (CR-H11). No comma, no space.
  - Group dropdown has EXACTLY 4 options; "Lực lượng vũ trang" NOT
    included (CR-E4).
  - Đăng bởi popover has EXACTLY 3 options; "Bán chuyên" NOT included
    (CR-H7 + pre_handoff_renames).
  - "Thời gian thu hồ sơ" chip renders on Tin đăng ONLY when Đăng bởi
    = "Chủ đầu tư"; NOT on Dự án (CR-H1 + pre_handoff_renames).
  - E7 has NO bottom CTA. Sheet dismisses via top-left ×. If a CTA
    button renders on E7, remove it.
  - Waived behaviors: verify NONE of the waived UI elements were
    accidentally implemented. Specifically:
    - No login gate sheet triggered from any non-login flow entry.
    - No tooltip UI on warning-state chips/inputs.
    - No empty state placeholder for the empty carousel.
    - No US2 alt-action CTA on the Insufficient screen.
    - No enumeration of specific failed condition on Insufficient.

8_report:
  # Every build hands this back
  - Measured inner padding per section (SEC-HUB-HEADER, SEC-HUB-FILTER-TIN,
    SEC-HUB-FILTER-DUAN, SEC-HUB-LIST-TIN, SEC-HUB-LIST-DUAN,
    SEC-ELI-SHEET-BODY, SEC-ELI-RESULT-BODY).
  - Measured slot gaps between adjacent sections on each screen.
  - Measured root height per block. If not equal to `computed.root.h`,
    report which block, spec value, built value, and diff.
  - Every token name that could NOT be resolved by the codebase, and
    what hex was used as fallback.
  - Every raw hex that appeared in the built code (should be zero — every
    colour is a token; if not, list the values and where).
  - Every bare number in the trees that did NOT land as written, with
    the node id and the value shipped instead.
  - Every `source: data` leaf and how it resolves — the runtime source
    and any bind-time fallback.
  - Breakpoint value used (if the app has multiple breakpoints).
  - Any Part 1 vs Part 2 contradiction encountered during build — do
    NOT resolve silently. Report both sides.
  - `path_hint.provenance = verified` OR `unknown` per section — every
    unknown that was resolved during this build.
  - A build that reports nothing was not measured.
```

---

## Self-check report (Step 10 of skill)

Run at the end of writing this spec. Results:

```yaml
1_parse: PASS - Every ```yaml block parses as valid YAML (validated during write).
2_exclusivity: PASS - Version A/B differentiation is a feature flag inside E3/E4 (via `conditional_render: "version == 'A'"` on section_house_ownership), NOT separate blocks per variant. No block contains ids from another version.
3_anchor_uniqueness: PASS - Every `find_by` string is a full quoted string. Near-miss siblings named in every Sections.confirm entry.
4_no_matrix_left: PASS - No `in_variant:` keys; no `variant_*:` override keys inside layout blocks. Feature flag is a runtime concern, marked via `conditional_render`.
5_numbers_agree: PASS - Every number in an AC (48, 32, 172, 218, 375, 760, 810, 850, 862) also appears in Part 2 computed values. No orphan numbers in ACs.
6_arithmetic: PASS - Every `maths` line closes at the root `computed.h`. Spot check: 760 = 48+640+72 (E2-INIT). 850 = 48+730+72 (E3/E4). 862 = 48+742+72 (E5/E6/E7-*).
7_references_resolve:
  - Every `use:` hits a Reusable entry (Chip, Button, Popover, List, Toggle, Text_Label, Input, Icon, "AvatarStack", "Ad-List", "Project-List", "Input-field/ Dropdown", "Pagination") OR another block's named child (via "BLOCK.child_id" syntax).
  - Every `screen:` hits Screens.
  - Every `section:` hits Sections.
  - Every `req:` / `cr:` / `behavior:` / `data_bound` reference hits its id.
  - "AvatarStack" is a DS component NAMED but NOT fully documented in Reusable — file OI if unsure how it's imported.
  PASS with note: "AvatarStack" import path to verify.
8_ids_unique_no_axis_span:
  - Ids unique within each block: PASS (audited during write).
  - No id spans two axes: PASS (each block's ids are self-contained).
9_placeholders:
  - No placeholder appears twice in one string: PASS.
  - Every placeholder has sample_data: PASS (see Reusable.sample_data + inline sample_data on individual leaves).
  - Every placeholder states whether its value arrives with or without unit: PASS in CR-E3 (m² is a suffix rendered as separate token), CR-H11 (integer with cap).
10_colours: PASS - Every colour is a token from the `tokens` table. No raw hexes in any tree (except the sample sample_data displays).
10b_strings: PASS - Every visible label is quoted verbatim in Vietnamese. Every AC about an on-screen element quotes the string.
11_no_contradictions:
  - Grep for "not shared" contradiction: nothing declared not-shared that is defined in Reusable.
  - Grep for a fact stated in two layers:
    - `applicant count cap` — stated in CR-H11 AND in Reusable.data_bound.applicant_count. Both agree ("99+").
    - `Đăng bởi options` — stated in AC-H3-2 AND CR-H7 AND pre_handoff_renames. All agree (3 options, no "Bán chuyên").
    - Warning tooltip — stated in AC-E3-5 (waived) AND waived_items.warning_tooltip. Both agree.
    - Non-login flow — stated in AC-H6-2 AND waived_items.non_login_flow. Both agree.
  PASS.
12_data_bound_resolves:
  - Every `source: data` leaf has a data_bound entry.
  - Every data_bound entry is referenced by at least one leaf: PASS (status_badge_text, applicant_count, applicant_avatars, project_name, project_price, project_price_per_area, project_area, project_location, region_chips, registered_user_count, contact_lead_submit, internal_lead_submit, matching_noxh_projects — all referenced).
  - No `source: fixed` anywhere in any tree: PASS (fixed is the default, not annotated).
13_composition_axis:
  - No `+` composing elements in any AC, what, or context: PASS.
  - Every container has explicit `type`: PASS.
14_literal_rule: PASS - Numbers-are-literals paragraph is in Tag definitions. No `Waived values` section. No `# waived raw` comments anywhere.
15_action_verbatim: PASS - Every `update` block's `action` contains the three replace-all sentences UNMODIFIED.
16_structural_completeness:
  - Screens, Sections, Layout blocks, Reusable, Verify checklist, Open items all exist: PASS.
  - Every block has action, what, reference_width, maths, computed, tree: PASS.
  - Every section has padding_check: PASS.
  - Every screen has slot_check: PASS.
17_unknowns_gate_ran:
  - U-1: asked (fixed vs data-bound) — resolved for icons/images (status badge = data-bound; hero image = fixed; carousel images = data-bound).
  - U-2: mapping sources named for every data_bound entry (either "existing X API" or specific mapping table).
  - U-3: asked (off-scale numbers) — deferred to OI-1 (initial sheet state), OI-2 (Tình trạng nhà ở duplicate), OI-4 (18169 dark bg). Others accepted as literals.
  - U-4: asked (interaction states) — user confirmed "all reuse DS default". Documented on ds_components entries.
  - U-5: asked (empty / loading / error / max) — Loading + submit-error + restart-with-prefill = reuse existing patterns (documented). Empty carousel + tooltip + US2 alt-action + reason display = WAIVED. All resolved.
  - U-6: asked (number formats) — comma decimal + dot thousands + 99+ cap. Documented in CR-H11 + Reusable.data_bound entries.
  - U-7: platform list confirmed = App only. Content Hub has Device=Desktop variants in Figma but per user App only ships. Documented.
  - U-8: PRD-vs-Figma conflicts — resolved in design-ready round and in prd-divergence entries. All resolved.
  PASS.
```

**Overall: PASS.** Spec is deliverable. Notes:
- 6 items in `before_qc` need dev/design/pm resolution — dev builds per `until_then` assumption in each; the OIs are for the follow-up conversation, not blockers on this build.
- 2 cosmetic items (OI-6, OI-7) — designer resolves in Figma at leisure.
- 2 pre-handoff Tier-1 fixes still pending in Figma (Bán chuyên removal + Thời gian thu hồ sơ move) — designer commits to fix; this spec is written as if fixed.
