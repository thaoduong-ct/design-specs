# handoff-noxh-buyer-actions-ad-detail.md

**Feature:** NOXH Buyer Actions — Primary Ad Detail entry points (US1 Tư vấn hồ sơ + US3 Quan tâm)
**PRD:** `carousell/ct-product-planning/specs/vertical/pty/noxh-buyer-actions-prd.md`
**Figma:** `UvLAgVP7Em1fwytMbuKHuv` node `15222:37328` (section: **Ad Detail**)
**Platforms:** iOS app, Android app (msite out of scope)
**Vertical:** PTY (Nhà Tốt) — NOXH (Nhà ở Xã hội) Primary listings
**Reference width:** 375
**Design mode:** greenfield (bottom sheets + toast are new; bottom action bar extends the existing sticky lead bar)

## How to use

Read PRD + this file together. The PRD carries intent and business rules; this file carries what to build and where it goes.

- **Part 1** (markdown tables) — ACs, content rules, behaviors. Each row is a sentence with an id. Reference from the layout tree by id.
- **Part 2** (YAML) — Reusable, Screens, Sections, Layout blocks, Verify checklist. Geometry lives here.
- **Chain:** an AC sends you to Layout blocks → `screen:` names the page → `section:` sends you to `Sections` to locate the node → `action:` says what to do → `what:` describes the finished frame → the tree **is** the finished contents.

Vietnamese labels are the codebase handle. Grep the exact strings quoted below to find the nodes. Static strings appear as-is (`"Tư vấn hồ sơ"`); dynamic strings appear as placeholders (`"{project_name}"`) with `sample_data` beside them in Reusable.

The design is one visual system for both iOS and Android — same layout, tokens, and typography (Reddit Sans, per the DS). Per-platform notes call out the framework primitive to reach for.

## Tag definitions

Defaults (omitted everywhere else):
- `align` on a text container defaults to **left** (natural language reading direction).
- Text `color` defaults to token `text-primary`.
- A leaf with no `source:` is **fixed**. A leaf with `source: data` binds to the value in `Reusable.data_bound.<id>`.
- Container `type` is **never** defaulted — every container writes `row`, `column`, or `stack` explicitly.
- `ty:` is shorthand for `typography` (references a token from `Reusable.typography`).
- `cr:` and `bh:` cross-reference a Part-1 content rule or behavior id.

**Literal rule (stated once):**
> Any numeric value in this spec is a literal. Implement it exactly as written. A design token is used only where the spec writes the token's name. A bare number is the value to build, not an approximation of a token, and nothing in this file converts one to the other. Rounding a number to a nearby token is a defect — report it rather than performing it.

Tag column (Part 1) is present only where a REQ mixes NEW with PARITY behavior. Nothing here is PARITY — the whole feature is NEW — so the Tag column is omitted.

## Open items

```yaml
blocking: []
before_qc:
  OI-1:
    issue: "PRD §Open Decisions ADR-H: which product surface manages the 'additional eligibility conditions' flag on a NOXH project (seller dashboard or admin?). Does NOT block the Primary Ad Detail build."
    owner: pm
    until_then: "The Ad Detail bottom action bar reads `project.status ∈ {đang_nhận_hồ_sơ, sắp_nhận_hồ_sơ}` to decide enabled/disabled; the eligibility flag does not gate the Tư vấn hồ sơ button on this surface."
  OI-2:
    issue: "PRD does not name the notification-toggle backend field on the success sheet. The design shows a toggle labelled 'Đồng ý nhận thông báo' that follows the project when ON and unfollows when OFF."
    owner: dev + pm
    until_then: "Wire the toggle to the same follow endpoint as the 'Quan tâm' bell (US3, ADR-D — new project_id-scoped follow endpoint). Toggle ON at open == follow, toggle OFF == unfollow."
  OI-3:
    issue: "Flow 3 (Đã kiểm tra điều kiện) shortcut confirmed with designer: tapping Tư vấn hồ sơ auto-submits using the info from US4's contact form and opens the success sheet without showing the form modal. PRD §US1 does not state this."
    owner: pm
    until_then: "Add the shortcut to PRD §US1 as an AC (see AC-04-shortcut below). Backend must expose 'has valid recent contact submission for this account' as a boolean on the ad-detail bootstrap payload so the client can pick the path (`shortcut_active` in Reusable.data_bound)."
cosmetic:
  OI-4:
    issue: "iOS uses Reddit Sans (per DS tokens) but the platform default is San Francisco. Confirm the DS ships a Reddit Sans typeface bundle for iOS."
    owner: design + dev
    until_then: "Use the Reddit Sans family names verbatim (`Reddit Sans` regular/medium/semibold/bold) as the DS defines them. If the bundle is missing, fall back to the platform system font at the same size/weight and report."
  OI-5:
    issue: "Success sheet toggle subtitle in the design reads `\"Để luôn cập nhập thông tin mới nhất từ dự án này\"`. Standard Vietnamese would be `cập nhật` (a diacritic on `ập` vs `ập`). We spec what the UI renders — the string is used as-is — but confirm this is intentional."
    owner: design
    until_then: "Build with `\"Để luôn cập nhập thông tin mới nhất từ dự án này\"` verbatim; if the designer confirms it should be `cập nhật`, the fix is a single copy edit in one string constant."
  OI-6:
    issue: "The positive banner card in the form sheet (`\"Để lại thông tin liên hệ\"` + orange helper line) is a RASTER IMAGE in Figma (image ref `623b2bc2119ad3934799758a85bfdc5f0795847b`), not native text. Baked-in copy is not localisable and cannot be A/B tested."
    owner: design + dev
    until_then: "Rebuild the banner as a native text container over a coloured background using the tokens named in AC-02c. The raster asset in Figma is a design placeholder; do NOT ship it."
  OI-7:
    issue: "The form sheet's footer uses the DS `Bottom Button` component in the `\"2 button\"` variant, but the design only renders one button (`\"Gửi thông tin\"`). PRD does not describe a secondary button."
    owner: design
    until_then: "Ship only the primary `\"Gửi thông tin\"` CTA — either by switching the instance to the DS `\"1 button\"` variant if it exists, or by hiding the secondary slot with a boolean property. Report on first build which path the DS supports."
  OI-8:
    issue: "The Flow 3 host Ad Detail screen (`15222:38669`) has a `\"Kiểm tra ngay điều kiện cần mua của dự án →\"` announcer strip (52pt tall, fill `#f2f6fc` — a new `info-subtle` token) above the bottom action bar. This is the PRD §US4 entry point on Primary Ad Detail. The design shows it ONLY on the Flow 3 host screen, not on Flow 1 or Flow 2 hosts. It is unclear whether visibility is gated on `has_checked_eligibility` or shown always."
    owner: design + pm
    until_then: "US4 entry point is out of scope for THIS spec (Buyer Actions covers US1 + US3 only) — do not build the announcer here. When US4's spec is written, use this token/height/copy as the starting point. The Ad Detail bottom action bar in this spec ignores the announcer's presence — its geometry (375×90) is authoritative for the bar itself; a host that also renders the US4 announcer will stack it above the bar."
```

---

## PART 1 — the requirement

### REQ-01 — Primary Ad Detail bottom action bar (Quan tâm + Tư vấn hồ sơ)

**Context** On the Primary Ad Detail page for a NOXH listing (a `listing_type = pty_primary` ad belonging to a NOXH `project`), a sticky bar pinned to the bottom of the viewport carries two controls: a small `"Quan tâm"` toggle on the left and a primary `"Tư vấn hồ sơ"` button on the right. Nothing above the bar in the ad detail content changes.
**Render when** The ad has `project.type = "noxh"` AND `project.status ∈ {"đang_nhận_hồ_sơ", "sắp_nhận_hồ_sơ"}`. When `project.status = "đóng_hồ_sơ"` the bar renders REQ-01 Block 3 (registered/disabled).
**Control** On a non-NOXH primary ad the existing generic sticky lead bar renders instead (unchanged).

#### Acceptance criteria

| ID | Criterion |
|---|---|
| **AC-01a** | Both controls sit on the same sticky bar pinned to the bottom of the ad detail viewport, above the OS home indicator; the bar has a 1px `border-thin` top border and a `background-primary` fill. Bar height is 90pt (56pt row + 34pt home-indicator strip). |
| **AC-01b** | Default state (project not followed, not yet registered) renders `bottom_action_bar_default`: the bell button on the left carries the icon and the label `"Quan tâm"` on a single line, 137×40 tonal-neutral pill; the orange button on the right carries the document icon and the label `"Tư vấn hồ sơ"`, 202×40 primary pill; the orange button is wider than the bell button. Row math: `16 + 137 + 4 + 202 + 16 = 375`. |
| **AC-01c** | Followed state (Quan tâm active) renders `bottom_action_bar_followed`: the bell shows a small green check-badge in the bottom-right and the `"Quan tâm"` text label is dropped from the button; only the icon remains, in a 56×40 icon-only pill. The orange `"Tư vấn hồ sơ"` button grows to 283×40 to fill the freed horizontal space. Row math: `16 + 56 + 4 + 283 + 16 = 375`. |
| **AC-01d** | Registered state renders `bottom_action_bar_registered`: the orange button is replaced by a disabled tonal button (`button-tonal-neutral` fill, `text-tertiary` label, no icon) reading `"Đã đăng ký"`; the bell control remains on the left in whichever state (default 137×40 or followed 56×40) it was in. |
| **AC-01e** | Tapping the bell toggles the follow state for `project.project_id` via the new project-scoped follow endpoint (ADR-D, PRD §Open Decisions). On follow: bell animates to the followed variant and a snackbar renders (see REQ-04). On unfollow: bell reverts and the unfollow snackbar renders. |
| **AC-01f** | Tapping the orange `"Tư vấn hồ sơ"` opens the `form_tu_van` bottom sheet (REQ-02) UNLESS the shortcut in AC-04-shortcut applies. |
| **AC-01g** | While the follow endpoint is in flight the bell shows its next-state visual optimistically. On error the bell reverts and an inline error toast renders (`text-error` text on a light surface, one line, distinct from the dark DS `.Snackbar`). |
| **AC-01h** | The bar remains fixed on the bottom edge across all scroll positions of the ad detail. The scrollable content above it must gain a bottom `contentInset` equal to the bar height (90) so the last piece of ad content is not obscured. |
| **AC-01i** | The bar is single-line on every viewport in scope (mobile 320–430pt wide). Labels never wrap; the orange button truncates last if the viewport is exceptionally narrow. |
| **AC-01j** | Bell button uses corner-radius 8 (`radius-card-small`); orange button uses corner-radius 8. |

#### Content rules

| ID | Element | Rule |
|---|---|---|
| **CR-01a** | Bell label (default state) | Static string `"Quan tâm"`. |
| **CR-01b** | Orange button label (active states) | Static string `"Tư vấn hồ sơ"`. |
| **CR-01c** | Disabled button label (registered state) | Static string `"Đã đăng ký"`. |
| **CR-01d** | Bell icon (default) | DS `Bell-line` icon at 20pt in token `icon-primary`. |
| **CR-01e** | Bell icon (followed) | DS `Bell-line` icon at 20pt in `icon-primary` PLUS a badge overlay in the bottom-right corner: 8pt filled circle in `text-success` (`#12a154`) with a 1pt `border-blank` outline. |
| **CR-01f** | Orange button icon | DS `Document-fill` (or equivalent contact icon) at 20pt in `icon-blank`. |

#### Behaviors

| ID | Element | On tap |
|---|---|---|
| **BH-01a** | Bell button (default → followed) | Optimistic UI: switch to followed variant, fire follow API, render REQ-04 follow snackbar on success, revert + error toast on failure. |
| **BH-01b** | Bell button (followed → default) | Optimistic UI: switch to default variant, fire unfollow API, render REQ-04 unfollow snackbar on success, revert + error toast on failure. |
| **BH-01c** | Orange `"Tư vấn hồ sơ"` button | If AC-04-shortcut applies → auto-submit + open success sheet (REQ-03). Otherwise → open `form_tu_van` bottom sheet (REQ-02). |
| **BH-01d** | Disabled `"Đã đăng ký"` button | No tap handler; button has `pointer-events: none`. |

### REQ-02 — Tư vấn hồ sơ form bottom sheet

**Context** A bottom-sheet modal that collects the buyer's contact info (Họ và tên, Số điện thoại), shown on top of the ad detail with a dim overlay. Header reads `"Tư vấn hồ sơ"`. Bottom-pinned primary CTA reads `"Gửi thông tin"`.
**Render when** Buyer taps the orange `"Tư vấn hồ sơ"` button on REQ-01 AND AC-04-shortcut does not apply.
**Control** Nothing on the ad detail below the sheet changes; the sheet only overlays.

#### Acceptance criteria

| ID | Criterion |
|---|---|
| **AC-02a** | The sheet has a `background-primary` fill, a 1px `border-thin` hairline border below the header, and top corners rounded to `radius-card` (12). Bottom corners are square (they meet the viewport edge). Sheet height 438. |
| **AC-02b** | Header row (375×48) contains a leading X close icon (24×24) and a centered title `"Tư vấn hồ sơ"` (typography `label-page`, 16/24 Bold, `text-primary`); padding `[12,20,12,20]`, gap 8. |
| **AC-02c** | A positive banner card (343×78, `radius-card` 12) sits under the header carrying a bold line `"Để lại thông tin liên hệ"` and an orange helper line `"Giúp chúng tôi có thể tư vấn cho bạn kỹ hơn!"` (`button-primary` orange, matching the DS positive-highlight illustration). In the Figma file this banner is a raster image asset with the text baked in — the build MUST recreate the banner as a text container over a coloured gradient background, using the DS tokens named in this file, so the copy can be localized and updated without a designer round-trip (see OI-6). |
| **AC-02d** | Two text inputs (343×48 each) stack vertically with 8pt gap: `"Họ và tên *"` (pre-filled with the account's `full_name` if logged in, with a clear-icon inside the field), `"Nhập số điện thoại *"` (pre-filled with the account's `phone` if logged in, phone keypad). Both are required — the CTA is enabled only when both fields have non-empty text. |
| **AC-02e** | Below the inputs, a consent row (343×72, 4pt gap): a leading shield-check icon (20×20 in `text-success` `#12a154`) and a body-caption paragraph (319×72) reading `"Bằng việc gửi thông tin, bạn đồng ý với "` + tappable `"Chính sách bảo mật của Nhà Tốt"` (in `text-link`) + `" và cho phép Nhà Tốt thu thập, Nhà Tốt xin cam kết thông tin sẽ được bảo mật và chỉ phục vụ cho việc liên hệ tư vấn dịch vụ theo nhu cầu của người mua"`. |
| **AC-02f** | Bottom-pinned CTA row is a DS `Bottom Button` instance rendered in its `"2 button"` variant, resolved to a single visible primary `"Gửi thông tin"` — full-width minus 16pt outer margin (343×40), `button-primary` fill (`#fa6819`), `text-blank` label, radius 8. The footer container itself is 375×72 (holds 16pt vertical padding + the button). The `"2 button"` variant's second slot ("Đóng") is not rendered on this screen — hide/omit it (see OI-7). |
| **AC-02g** | On CTA tap: submit `{full_name, phone, project_id, ad_id, source: "ad_detail"}` to the developer-lead endpoint. On success: dismiss form and open the success sheet (REQ-03). On failure: keep the form open, keep field values, show an inline `text-error` line above the CTA with the retryable message. |
| **AC-02h** | On close-X tap OR overlay tap OR back gesture: dismiss the form; state is discarded (no draft persistence). |
| **AC-02i** | Sheet height adapts to content up to `85vh` cap; internal scroll appears when content exceeds the cap (never scroll under the pinned CTA). |
| **AC-02j** | Overlay behind the sheet uses token `background-overlay` (`#22222280`) full-viewport. |
| **AC-02k** | If the buyer has previously submitted contact info for THIS same project, the CTA remains active but on submit the response is 409-registered — surface the error inline and update the bell + button to the registered state (AC-01d) after dismiss. |

#### Content rules

| ID | Element | Rule |
|---|---|---|
| **CR-02a** | Positive banner primary line | Static: `"Để lại thông tin liên hệ"`. |
| **CR-02b** | Positive banner helper line | Static: `"Giúp chúng tôi có thể tư vấn cho bạn kỹ hơn!"`. |
| **CR-02c** | Field 1 label + placeholder | Label `"Họ và tên"` with `*` in `text-error`; placeholder same text; pre-fill from `account.full_name` if logged in. |
| **CR-02d** | Field 2 label + placeholder | Label `"Nhập số điện thoại"` with `*` in `text-error`; placeholder same text; pre-fill from `account.phone` if logged in. Keyboard `phonePad` (iOS) / `TYPE_CLASS_PHONE` (Android). |
| **CR-02e** | Consent copy | Static paragraph verbatim per AC-02e. Only `"Chính sách bảo mật của Nhà Tốt"` is a tappable link. |
| **CR-02f** | CTA label | Static: `"Gửi thông tin"`. |
| **CR-02g** | Header title | Static: `"Tư vấn hồ sơ"`. |

#### Behaviors

| ID | Element | On tap |
|---|---|---|
| **BH-02a** | X close icon / overlay / back gesture | Dismiss sheet, discard state. |
| **BH-02b** | `"Chính sách bảo mật của Nhà Tốt"` link | Open the existing Nhà Tốt privacy policy screen (grep `chinh-sach-bao-mat` route). |
| **BH-02c** | `"Gửi thông tin"` CTA | See AC-02g. |

### REQ-03 — Thành công success bottom sheet

**Context** A bottom-sheet modal replaces the form after a successful `Tư vấn hồ sơ` submission (from REQ-02) OR is shown directly after the AC-04-shortcut fires. Header reads `"Thành công"`. Body confirms the submission, offers a notification toggle for THIS project (variant 3a only), and shows a horizontal carousel of `Danh sách NOXH phù hợp` — other active NOXH projects in the same area as the registered project, excluding the registered project itself.
**Render when** REQ-02 submit returns 2xx, OR the AC-04-shortcut fires (skipping the form).
**Control** Nothing on the ad detail changes until the sheet is dismissed.

#### Acceptance criteria

| ID | Criterion |
|---|---|
| **AC-03a** | Same sheet chrome as REQ-02 (fill, radius, header, X). Header title is `"Thành công"`. Sheet height 671 for default variant, 593 for no-toggle variant. |
| **AC-03b** | Under the header a 375×207 peach-gradient hero band (fading from a warm peach at the top down to `background-primary`) carries an emoji `🎉` + static line `"Tư vấn viên sẽ liên hệ lại bạn sớm nhất."` centered near the top, and a 96×96 clapping-hands illustration (asset `success_clapping_hands`) centered near the bottom. |
| **AC-03c** | **Variant 3a (default)** — under the hero, a notification toggle card renders: title `"Đồng ý nhận thông báo"` + subtitle `"Để luôn cập nhập thông tin mới nhất từ dự án này"` + a toggle switch aligned right; toggle default state is ON; the card fills to viewport width minus 16pt outer margin (343×66), `background-app` fill (`#f7f7f7`), `radius-card` (12), inner padding `[12,16,12,16]`, 4pt gap between the inner text column and the toggle. |
| **AC-03d** | **Variant 3b (no toggle)** — the notification toggle card AND its adjacent 12pt vertical gap AND the hairline divider directly below it are OMITTED from the vertical stack; every other node is unchanged. Sheet height shrinks by exactly 78pt (66 toggle + 12 gap). This variant is rendered whenever the buyer has ALREADY opted into notifications for this project prior to the sheet opening (e.g. via US4 area subscription or a prior US3 follow that included this project). |
| **AC-03e** | Under the toggle (or hero, for 3b) a section row (343×32, 12pt gap): title `"Danh sách NOXH phù hợp"` (typography `label-page`, `text-primary`) on the left in a 223-wide slot, and a 108×32 outlined pill `"Xem tất cả →"` (typography `label-annotation`, `text-primary`, `button-blank` fill, `border-regular` outline, radius 8) on the right. |
| **AC-03f** | Below the section row: a horizontal-scroll carousel (648×252) of `noxh_project_card` items (each 153×252), first card left-aligned to the 16pt outer margin, subsequent cards separated by 12pt gap; the right edge of the last visible card bleeds off the viewport, indicating scrollability. |
| **AC-03g** | Each `noxh_project_card` shows: hero thumbnail (aspect ratio ~4:3, top corners `radius-card` 12), a status badge overlay at top-left (green fill for `"Đang nhận hồ sơ"`, orange fill for `"Sắp nhận hồ sơ dd/mm"`), a photo-count chip at bottom-right (dark scrim `background-overlay`, camera icon + count), title (`label-page` `text-primary`, clamp-2), price range (`display-annotation` in `button-primary` orange), address row with pin icon + address (`body-caption` in `text-secondary`, clamp-1). |
| **AC-03h** | Toggle behavior: OFF → unfollow this project via ADR-D endpoint; ON → follow this project. Optimistic UI, revert on error. |
| **AC-03i** | `"Xem tất cả →"` button navigates to the NOXH Hub, `Dự án` tab, scrolled to the top of the projects list — same route the Hub banner uses. |
| **AC-03j** | Each `noxh_project_card` tap navigates to that project's detail page (`/project/{project_id}`). |
| **AC-03k** | Data-source: `GET /noxh/projects/similar?area_id={registered_project.area_id}&exclude={registered_project.project_id}&status_in=đang_nhận_hồ_sơ,sắp_nhận_hồ_sơ`; carousel renders when response has ≥1 result and is omitted when the list is empty (empty-list case falls back to just showing the hero + toggle + `"Xem tất cả →"` linking to Hub). |
| **AC-03l** | On sheet dismiss (X or overlay or gesture) — return to the ad detail with the bell in followed state and the button in `"Đã đăng ký"` (AC-01d). |

#### Content rules

| ID | Element | Rule |
|---|---|---|
| **CR-03a** | Hero success line | Static: `"🎉 Tư vấn viên sẽ liên hệ lại bạn sớm nhất."` (emoji is part of the literal string, not an icon leaf). |
| **CR-03b** | Toggle card title | Static: `"Đồng ý nhận thông báo"`. |
| **CR-03c** | Toggle card subtitle | Static: `"Để luôn cập nhập thông tin mới nhất từ dự án này"` (see OI-5 about `cập nhập`). |
| **CR-03d** | Section header title | Static: `"Danh sách NOXH phù hợp"`. |
| **CR-03e** | View-all pill label | Static: `"Xem tất cả →"` (arrow is part of the literal string). |
| **CR-03f** | Card status badge — accepting | Static: `"Đang nhận hồ sơ"`, badge fill `text-success` (`#12a154`), `text-blank` label. |
| **CR-03g** | Card status badge — upcoming | Placeholder: `"Sắp nhận hồ sơ {dd/mm}"`; `{dd/mm}` is `project.application_open_at` formatted as day/month, no leading zero on day. Badge fill `button-primary` (`#fa6819`), `text-blank` label. |
| **CR-03h** | Card title | Placeholder: `"{project_name}"`; clamp-2. sample: `"NOXH Happy Home Nhơn Trạch"`. |
| **CR-03i** | Card price range | Placeholder: `"Từ {min_price_vnd_short} - {max_price_vnd_short}"`. Vietnamese short form: `890 triệu`, `1,5 tỷ`, decimals with a comma, unit inline. sample: `"Từ 890 triệu - 1,5 tỷ"`. |
| **CR-03j** | Card address | Placeholder: `"{project.address_short}"`; clamp-1. sample: `"Đường số 10 Nguyễn..."`. |
| **CR-03k** | Card photo-count chip | Placeholder: `"{photo_count}"` as bare integer + camera icon; on a dark scrim (`background-overlay`). sample: `"6"`. |

#### Behaviors

| ID | Element | On tap |
|---|---|---|
| **BH-03a** | X / overlay / back gesture | Dismiss; on dismiss update the ad detail bar to registered state (AC-01d) and followed bell state (AC-01c) if toggle was ON. |
| **BH-03b** | Notification toggle | See AC-03h. |
| **BH-03c** | `"Xem tất cả →"` pill | Navigate to NOXH Hub / `Dự án` tab. |
| **BH-03d** | `noxh_project_card` | Navigate to `/project/{project_id}`. |

### REQ-04 — Follow / unfollow snackbar

**Context** A single-line dark snackbar appears at the bottom of the Ad Detail viewport (above the sticky bottom action bar) when the user taps the Quan tâm bell to follow or unfollow.
**Render when** Follow API returns 2xx (follow snackbar) or unfollow API returns 2xx (unfollow snackbar).
**Control** No snackbar on API error — instead the bell reverts + an inline error toast at the same position renders per AC-01g.

#### Acceptance criteria

| ID | Criterion |
|---|---|
| **AC-04a** | Snackbar sits above the sticky bottom action bar (REQ-01) — never overlapping it — with a 19pt clear gap between snackbar bottom and bar top (snackbar bottom-edge y ≈ 703, bar top-edge y = 722 on the 812pt reference frame). |
| **AC-04b** | Snackbar renders as the DS `.Snackbar` component instance (single-line variant): dark surface (`background-inverted`, `#222222`), `text-blank` label, `radius-card` (12), horizontal padding 16pt, vertical padding 12pt, width 343 (viewport 375 minus 16pt outer margin on each side), height 44. |
| **AC-04c** | Auto-dismiss after 2000ms; user can also swipe down to dismiss. |
| **AC-04d** | On follow success → snackbar text: `"Thêm dự án nhận thông báo thành công"`. On unfollow success → snackbar text: `"Đã bỏ theo dõi dự án"`. |
| **AC-04e** | Only one snackbar can be visible at a time; a new one replaces the previous with no stacking. |

#### Content rules

| ID | Element | Rule |
|---|---|---|
| **CR-04a** | Follow snackbar text | Static: `"Thêm dự án nhận thông báo thành công"`. |
| **CR-04b** | Unfollow snackbar text | Static: `"Đã bỏ theo dõi dự án"`. |

### AC-04-shortcut — Flow 3 auto-submit path

| ID | Criterion |
|---|---|
| **AC-04-shortcut** | Shortcut applies when: buyer is logged in AND `account.has_recent_valid_contact_submission == true` (per OI-3 backend signal) AND `project.registration_status_for_this_account == "not_registered"`. When it applies: tapping the orange `"Tư vấn hồ sơ"` button (REQ-01) skips the form modal, fires the developer-lead submission with the account's stored `full_name`/`phone`, and opens the success sheet (REQ-03) directly on success. On failure the buyer is dropped into the form modal (REQ-02) with fields pre-filled from the account. |

---

## PART 2 — layout

```yaml
Reusable:
  tokens:
    - { name: background-primary,        hex: "#ffffff",   role: "sheet surface, page background, blank button fill" }
    - { name: background-secondary,      hex: "#f4f4f4",   role: "tonal button fill (Quan tâm default state, Đã đăng ký disabled state)" }
    - { name: background-app,            hex: "#f7f7f7",   role: "ad detail page background AND notification-toggle-card fill on the success sheet" }
    - { name: background-inverted,       hex: "#222222",   role: "snackbar surface" }
    - { name: background-overlay,        hex: "#22222280", role: "modal dim overlay behind bottom sheet, photo-count chip scrim" }
    - { name: background-success-light,  hex: "#ecf9f1",   role: "positive banner card fill inside form sheet (fallback if the raster is replaced by a solid fill instead of a gradient)" }
    - { name: text-primary,              hex: "#222222",   role: "primary text; icon-primary shares the hex" }
    - { name: text-secondary,            hex: "#595959",   role: "secondary text (toggle subtitle, address)" }
    - { name: text-tertiary,             hex: "#8c8c8c",   role: "muted labels, `Đã đăng ký` disabled label" }
    - { name: text-blank,                hex: "#ffffff",   role: "text on dark surfaces (snackbar, orange CTA)" }
    - { name: text-link,                 hex: "#306bd9",   role: "link text (consent copy)" }
    - { name: text-success,              hex: "#12a154",   role: "check icon in consent, followed-bell badge, `Đang nhận hồ sơ` badge fill" }
    - { name: text-error,                hex: "#f0325e",   role: "asterisk on required inputs, inline form error" }
    - { name: button-primary,            hex: "#fa6819",   role: "orange CTA fill (`Tư vấn hồ sơ`, `Gửi thông tin`, `Sắp nhận hồ sơ` badge, positive-banner helper text)" }
    - { name: button-blank,              hex: "#ffffff",   role: "`Xem tất cả` pill fill" }
    - { name: button-tonal-neutral,      hex: "#f4f4f4",   role: "`Quan tâm` bell (default) fill, disabled `Đã đăng ký` button fill" }
    - { name: icon-primary,              hex: "#222222",   role: "bell icon, X close icon" }
    - { name: icon-blank,                hex: "#ffffff",   role: "document icon on orange CTA, camera icon on photo-count chip" }
    - { name: icon-tertiary,             hex: "#8c8c8c",   role: "location pin icon on card address" }
    - { name: border-thin,               hex: "#e8e8e8",   role: "hairline dividers (below sheet header, top border of sticky bar)" }
    - { name: border-regular,            hex: "#dddddd",   role: "input field border, `Xem tất cả` pill outline" }
    - { name: border-divider,            hex: "#f4f4f4",   role: "hairline divider between the toggle card and the section header on success sheet" }
    - { name: border-blank,              hex: "#ffffff",   role: "outline on the green check badge over the followed-bell icon" }
    - { name: "info-subtle (new)",       hex: "#f2f6fc",   role: "US4 announcer strip fill on the Flow 3 host — OUT OF SCOPE (see OI-8)" }

  typography:
    - { name: header-page,        family: "Reddit Sans", weight: SemiBold, size: 20, line_height: 28 }
    - { name: header-caption,     family: "Reddit Sans", weight: SemiBold, size: 14, line_height: 20 }
    - { name: label-page,         family: "Reddit Sans", weight: Bold,     size: 16, line_height: 24 }
    - { name: label-section,      family: "Reddit Sans", weight: Bold,     size: 14, line_height: 20 }
    - { name: label-annotation,   family: "Reddit Sans", weight: Bold,     size: 10, line_height: 16 }
    - { name: body-section,       family: "Reddit Sans", weight: Regular,  size: 14, line_height: 20 }
    - { name: body-caption,       family: "Reddit Sans", weight: Regular,  size: 12, line_height: 18 }
    - { name: display-caption,    family: "Reddit Sans", weight: Bold,     size: 20, line_height: 28 }
    - { name: display-annotation, family: "Reddit Sans", weight: Bold,     size: 18, line_height: 26 }
    - { name: tagline-caption,    family: "Reddit Sans", weight: Medium,   size: 14, line_height: 20 }
    - { name: tagline-annotation, family: "Reddit Sans", weight: Medium,   size: 12, line_height: 18 }

  spacing:
    gap-min-2: 2
    gap-2x-small-4: 4
    gap-2x-small-6: 6
    gap-x-small-8: 8
    gap-small-12: 12
    gap-medium-16: 16
    padding-min-2: 2
    padding-2x-small-4: 4
    padding-x-small-8: 8
    padding-small-12: 12
    padding-medium-16: 16

  radius:
    radius-ad-small: 4
    radius-ad: 6
    radius-card-small: 8
    radius-card: 12
    radius-pill: 999

  effects:
    shadow-floating: "drop-shadow(0 4px 16px #2222221F)"

  icons:
    - Bell-line
    - Document-fill
    - X
    - Shield-check
    - Camera
    - Pin

  assets:
    - name: success_clapping_hands
      kind: raster
      size: 96x96
      source: "existing Nhà Tốt success illustration; if not present, ship as png+webp asset at 1x/2x/3x"
      note: "same illustration used on the eligibility-check (US4) result screen, per PRD parity"

  ds_components:
    Snackbar:
      variant: single-line
      props: { Device: App, "Action button": No, Icon: No, "Horizontal / Vertical": No }
      resolved_style: { fill: background-inverted, radius: 12, padding_x: 16, padding_y: 12, text_style: body-section, text_color: text-blank, w: 343, h: 44 }
      note: "the design uses the DS `.Snackbar` instance at 343×44 on 15222:37981"
    ButtonPrimary_L_40:
      variant: "Primary/Active/L-40"
      props: { size: large, tone: primary, "Icon left": true }
      resolved_style: { fill: button-primary, radius: 8, height: 40, padding_x: 16, text_style: label-section, text_color: text-blank, icon_size: 20, icon_gap: 8 }
    ButtonPrimary_S_32:
      variant: "Primary/Outline/S-32"
      props: { size: small }
      resolved_style: { fill: button-blank, border: "1px border-regular", radius: 8, height: 32, padding_x: 12, padding_y: 6, text_style: label-annotation, text_color: text-primary }
      note: "`Xem tất cả →` pill on the success sheet uses this outlined-blank variant"
    ButtonTonalNeutral_L_40:
      variant: "Tonal/Neutral/L-40"
      props: { size: large, tone: tonal-neutral }
      resolved_style: { fill: button-tonal-neutral, radius: 8, height: 40, padding_x: 12, text_style: label-section, text_color: text-primary, icon_size: 20, icon_gap: 4 }
      note: "used for the `Quan tâm` bell (default state, 137×40) and the disabled `Đã đăng ký` button (202×40)"
    ButtonTonalNeutral_IconOnly_40:
      variant: "Tonal/Neutral/IconOnly-40"
      props: { size: large, tone: tonal-neutral, "Icon only": true }
      resolved_style: { fill: button-tonal-neutral, radius: 8, w: 56, h: 40, padding: 8, icon_size: 20 }
      note: "`Quan tâm` bell followed state, 56×40; the followed variant adds an 8pt filled circle badge (`text-success` fill, `border-blank` 1pt outline) absolute-positioned in the bottom-right corner"
    Switch:
      variant: default
      props: { size: small, On: Yes }
      resolved_style: { on_fill: button-primary, off_fill: border-regular, thumb: background-primary, w: 36, h: 20 }
    InputField:
      variant: "Input-field/Input/Default/Out Focus"
      props: { size: large, "Fill text": bool, "Requirement": true, "Clear icon": bool }
      resolved_style: { fill: background-primary, border: "1px border-regular", radius: 8, height: 48, padding_x: 12, padding_y: 8, label_style: body-caption, label_color: text-secondary, value_style: body-section, value_color: text-primary, asterisk_color: text-error }
      note: "input height is 48pt (NOT 56pt); clear icon is present on field 1 (fill_text=true) and absent on field 2 (fill_text=false)"
    DrawerHeader:
      variant: "Drawer Header/Default"
      props: { "Left icon": X, "Right slot": empty }
      resolved_style: { fill: background-primary, height: 48, padding: "[12,20,12,20]", gap: 8, title_style: label-page, title_color: text-primary, x_icon_size: 24, right_spacer_size: 24 }
    BottomButton_2:
      variant: "2 button"
      props: { primary_label: string, secondary_label: string, "Show secondary": bool }
      resolved_style: { height: 72, container_padding: "16 top + 16 bottom", primary: ButtonPrimary_L_40 full-width, secondary: ButtonBlank_L_40 full-width, gap: 8 }
      note: "on the form sheet, `Show secondary=false` — see OI-7"

  data_bound:
    account_full_name:
      binds: value
      source: "account.full_name from the authenticated session; empty string when logged out"
      sample_data: "Thảo"
    account_phone:
      binds: value
      source: "account.phone from the authenticated session; empty string when logged out"
      sample_data: "0915343932"
    project_id:
      binds: navigation target
      source: "project.project_id on the current Ad Detail"
      sample_data: "phong-5-tot-nd121"
    similar_project_list:
      binds: repeat over the carousel item
      source: "GET /noxh/projects/similar?area_id={registered_project.area_id}&exclude={registered_project.project_id}&status_in=đang_nhận_hồ_sơ,sắp_nhận_hồ_sơ"
      sample_data: "2 cards visible: NOXH Happy Home Nhơn Trạch (Đang nhận hồ sơ, Từ 890 triệu - 1,5 tỷ) and K-Home Avenue (Sắp nhận hồ sơ 15/7, Từ 890 triệu - 1,5 tỷ)"
    project_card_status_badge:
      binds: fill + text
      source: "project.status ∈ {đang_nhận_hồ_sơ, sắp_nhận_hồ_sơ}"
      mapping: "đang_nhận_hồ_sơ → CR-03f (green fill, static text `Đang nhận hồ sơ`); sắp_nhận_hồ_sơ → CR-03g (orange fill, `Sắp nhận hồ sơ {dd/mm}` with `{dd/mm}` from project.application_open_at)"
      sample_data: "K-Home Avenue → `Sắp nhận hồ sơ 15/7`"
    project_card_price_range:
      binds: text
      source: "project.min_price and project.max_price, formatted with the existing Vietnamese short-currency helper (grep for `formatVndShort` or `formatToVND` in the repo — do not invent a new one)"
      mapping: "reuses the existing formatter"
      sample_data: "Từ 890 triệu - 1,5 tỷ"
    project_card_address_short:
      binds: text
      source: "project.address_short, single-line, clamp-1"
      sample_data: "Đường số 10 Nguyễn..."
    quan_tam_state:
      binds: bell button visual + button-adjacent label
      source: "follow_state.projects[project.project_id] via ADR-D endpoint; boolean followed | not_followed"
      mapping: "not_followed → CR-01d bell + `Quan tâm` label · followed → CR-01e bell (with green badge) + no label"
      sample_data: "not_followed on Flow 1; followed on Flow 2 after tap; followed on Flow 3 entry"
    registered_state:
      binds: right-side button
      source: "lead_state.projects[project.project_id] via developer-lead endpoint; boolean registered | not_registered"
      mapping: "not_registered → orange `Tư vấn hồ sơ` (CR-01b); registered → tonal `Đã đăng ký` (CR-01c)"
      sample_data: "not_registered on all three flow entries; registered after REQ-03 success"
    shortcut_active:
      binds: which handler fires when the orange button is tapped
      source: "account.has_recent_valid_contact_submission — a server-side boolean returned on the ad-detail bootstrap payload (see OI-3)"
      mapping: "false → open REQ-02 form modal; true → skip form, auto-submit, open REQ-03 success sheet directly"
      sample_data: "false on Flow 1 and Flow 2; true on Flow 3"
    toggle_variant_gate:
      binds: whether REQ-03 renders variant 3a or 3b
      source: "account.has_active_notification_subscription_for_project(project.project_id) at the moment the success sheet is opened"
      mapping: "false → variant 3a (toggle card visible, default ON); true → variant 3b (toggle card omitted)"
      sample_data: "false on Flow 1 and Flow 2; true on Flow 3 (user already opted in via US4)"

  local_components:
    noxh_project_card:
      w: 153
      h: 252
      radius: 12
      fill: background-primary
      border: "1px border-thin"
      note: "used only in success_sheet_default and success_sheet_no_toggle; not shared with the NOXH Hub card in this wave (per Q3 answer)"
      tree:
        - id: card_hero
          type: stack
          w: 153
          h: 114
          radius: "12 top-left, 12 top-right, 0 bottom-left, 0 bottom-right"
          children:
            - { id: card_thumbnail, type: image, source: data, name: "{project_thumbnail_url}", w: 153, h: 114 }
            - { id: card_status_badge, type: text, source: data, text: "{project_card_status_badge}", position: "absolute, top 8, left 8", fill: "text-success (green) OR button-primary (orange) — per data mapping", text_color: text-blank, ty: tagline-annotation, radius: 6, padding: "[4,8,4,8]", cr: [CR-03f, CR-03g] }
            - id: card_photo_chip
              type: row
              position: "absolute, bottom 8, right 8"
              fill: background-overlay
              radius: 6
              padding: "[2,6,2,6]"
              gap: 4
              children:
                - { id: card_photo_icon, type: icon, name: Camera, size: 12, color: icon-blank }
                - { id: card_photo_count, type: text, text: "{photo_count}", ty: tagline-annotation, color: text-blank, source: data, cr: CR-03k }
        - id: card_body
          type: column
          w: 153
          h: 138
          gap: 4
          padding: "[8,8,8,8]"
          children:
            - { id: card_title, type: text, text: "{project_name}", ty: label-page, color: text-primary, clamp: 2, source: data, cr: CR-03h }
            - { id: card_price, type: text, text: "Từ {min_price_vnd_short} - {max_price_vnd_short}", ty: display-annotation, color: button-primary, source: data, cr: CR-03i }
            - id: card_address_row
              type: row
              gap: 4
              align_items: center
              children:
                - { id: card_address_icon, type: icon, name: Pin, size: 12, color: icon-tertiary }
                - { id: card_address, type: text, text: "{address_short}", ty: body-caption, color: text-secondary, clamp: 1, source: data, cr: CR-03j }

Screens:
  primary_ad_detail_mobile:
    renders: [bottom_action_bar_default, bottom_action_bar_followed, bottom_action_bar_registered, snackbar_toast]
    place: |
      viewport (screen, 375 wide, dynamic height, background-app)
        header (fixed, existing app header — untouched)
        gallery_and_content_scroll (existing, contentInset.bottom = 90 to clear the sticky bar)
        ★ bottom_action_bar (sticky, pinned to safe-area-bottom)
        snackbar (transient, sits ABOVE bottom_action_bar with 19pt clear gap when render fires)
    parent_provides: [the safe-area inset at the bottom is applied by OS chrome, not by this bar; the scroll view provides its own contentInset]
    untouched: [header, gallery, ad info card, project info section, price insight, map/address block, similar ads, and every other adview cell that already ships on Primary Ad Detail]
    page_background: background-app
    slot_check: { bar_height: 90, bar_top_border: "1px border-thin" }

  form_tu_van_sheet:
    renders: [form_sheet]
    place: |
      viewport (375 × device height)
        overlay (full-viewport, background-overlay #22222280, tap dismisses)
        ★ form_sheet (docked at bottom, height 438, top-corners radius-card 12, fill background-primary)
    parent_provides: [the overlay + tap-to-dismiss chrome is the DS `BottomSheet` container — do NOT add a second scrim inside `form_sheet`]
    untouched: [the underlying Ad Detail — the sheet is a modal overlay only]
    page_background: background-primary
    slot_check: { sheet_height: 438, sheet_radius_top: 12, overlay: background-overlay }

  success_tu_van_sheet:
    renders: [success_sheet_default, success_sheet_no_toggle]
    place: |
      viewport (375 × device height)
        overlay (full-viewport, background-overlay)
        ★ success_sheet (docked at bottom, height 671 for default OR 593 for no-toggle, top-corners radius-card 12, fill background-primary)
    parent_provides: [DS `BottomSheet` overlay + swipe-down-to-dismiss + tap-outside-to-dismiss]
    untouched: [the underlying Ad Detail — the sheet overlays it]
    page_background: background-primary
    slot_check: { sheet_height_default: 671, sheet_height_no_toggle: 593, sheet_radius_top: 12 }

Sections:
  bottom_action_bar:
    acted_on_by: [bottom_action_bar_default, bottom_action_bar_followed, bottom_action_bar_registered]
    figma:
      primary_ad_detail_mobile:
        default:  { file: UvLAgVP7Em1fwytMbuKHuv, node: "15222:37507", frame: "Ad Detail (15222:37331) > bottom-button" }
        followed: { file: UvLAgVP7Em1fwytMbuKHuv, node: "15222:37982", frame: "Ad Detail/quan tam du an (15222:37805) > bottom-button" }
    path_hint:
      ios:     { path: "Cells/PTY/", provenance: documented, source: "sitemap-adview.md — PTY cells convention (AVPTYPriceInfoViewCell etc.)", verify_on_first_build: true }
      android: { path: "app/.../sd/features/pty/addetail/", provenance: documented, source: "sitemap-adview.md — PTY viewholder convention (PtyAdDetailEnquiryViewHolder)", verify_on_first_build: true }
    find_by: >
      A sticky container pinned to the bottom of a Primary Ad Detail page whose two children are, in order, the button-group row `[Quan tâm, Tư vấn hồ sơ]` and the OS home-indicator strip. Grep the string `"Tư vấn hồ sơ"` OR the string `"Quan tâm"` together with the file/module that already owns the Primary Ad Detail's other sticky footer (the existing generic sticky lead bar for non-NOXH primary ads).
    confirm: >
      The bar renders ONLY when the ad is a NOXH primary listing (`ad.listing_type == pty_primary` AND `ad.project.type == noxh`). If the same file also renders the generic FB/chat lead bar for non-NOXH ads — that is the near miss. Do NOT edit the generic bar; add a branch that returns this NOXH-specific bar for NOXH ads. If you are looking at a bar that has an FB Messenger button, you are in the wrong file.
    current: >
      Nothing renders here today for NOXH ads — the sticky bar for NOXH primaries is what this spec introduces. Non-NOXH primary ads keep the existing generic sticky lead bar (untouched).
    owns:
      this_section_provides: [the sticky container surface (background-primary fill, 1px border-thin top border), the horizontal button row, the home-indicator strip]
      therefore: >
        The container IS the sticky footer surface — do NOT nest a second card inside. The scroll view above must gain a bottom contentInset of 90pt so the last ad content is not obscured; this is set on the scroll view, NOT via padding on the sticky bar itself.
    padding_check: { inner_x: 16, inner_y: 8, surfaces: 1 }

  snackbar_toast:
    acted_on_by: [snackbar_toast]
    figma:
      primary_ad_detail_mobile:
        follow: { file: UvLAgVP7Em1fwytMbuKHuv, node: "15222:37981", frame: "Ad Detail/quan tam du an (15222:37805) > .Snackbar" }
    path_hint:
      ios:     { path: "unknown — new call site of the existing DS `.Snackbar` UIView bundled with the platform DS", provenance: unknown }
      android: { path: "unknown — new call site of the existing DS `.Snackbar` Composable bundled with the platform DS", provenance: unknown }
    find_by: >
      A single-line dark toast bearing exactly the string `"Thêm dự án nhận thông báo thành công"` (follow success) or `"Đã bỏ theo dõi dự án"` (unfollow success). Grep the DS `.Snackbar` component; if the DS ships a `Toast` or `SnackbarHost`, use that.
    confirm: >
      The snackbar surfaces from tapping the `Quan tâm` bell in the bottom action bar; it does not appear on any other Ad Detail interaction. If you find the same DS `.Snackbar` used elsewhere (e.g. save-ad confirmation), that is a different call site with different copy — do not merge.
    current: >
      Nothing renders here today for NOXH follow/unfollow — this is a new call site of the DS `.Snackbar` component with new copy strings.
    owns:
      this_section_provides: [the transient snackbar surface — dark fill, rounded 12, 343×44, text-blank single-line label]
      therefore: >
        Do NOT stack multiple snackbars. A new toast replaces the previous one immediately. The snackbar's y-position is anchored 19pt above the sticky bottom action bar; when the bar's height changes (e.g. the OI-8 announcer variant on the Flow 3 host), the snackbar stays 19pt above the bar's top edge.
    padding_check: { inner_x: 16, inner_y: 12, surfaces: 1 }

  form_sheet:
    acted_on_by: [form_sheet]
    figma:
      form_tu_van_sheet:
        default: { file: UvLAgVP7Em1fwytMbuKHuv, node: "15222:37512", frame: "Ad Detail/form tu van (15222:37510) > <sheet>" }
    path_hint:
      ios:     { path: "unknown — new bottom-sheet view controller, likely under `PTY/NoxhBuyerActions/`", provenance: unknown }
      android: { path: "unknown — new bottom-sheet fragment, likely under `feature-noxh-buyer-actions/` (create module if missing)", provenance: unknown }
    find_by: >
      A modal bottom sheet whose header title is exactly the string `"Tư vấn hồ sơ"` and whose bottom CTA is exactly the string `"Gửi thông tin"`. Grep either literal.
    confirm: >
      The sheet is presented on tap of the orange `"Tư vấn hồ sơ"` button in the bottom_action_bar (BH-01c). If a sheet with the same header title `"Tư vấn hồ sơ"` renders from another entry point (e.g. NOXH Hub or Project Detail), those are separate call sites but should share this same component — verify on first build and factor out a shared component if the same shape appears in ≥2 call sites.
    current: >
      Nothing renders here today. New component.
    owns:
      this_section_provides: [the sheet surface (background-primary, radius-card top corners), the header row, the scrollable body (positive banner + inputs + consent), the docked footer with the primary CTA]
      therefore: >
        The DS `BottomSheet` container provides the overlay and dismiss chrome — do NOT add a second scrim inside this section. Sheet height is content-driven up to 85vh cap; body scrolls if content exceeds cap; the footer stays pinned outside the scroll.
    padding_check: { inner_x: 16, inner_y: 16, surfaces: 1 }

  success_sheet:
    acted_on_by: [success_sheet_default, success_sheet_no_toggle]
    figma:
      success_tu_van_sheet:
        default:   { file: UvLAgVP7Em1fwytMbuKHuv, node: "15222:37531", frame: "Ad Detail/de lai tu van thanh cong (15222:37529) > <sheet>" }
        no_toggle: { file: UvLAgVP7Em1fwytMbuKHuv, node: "15222:38868", frame: "Ad Detail/de lai tu van thanh cong (Flow 3, 15222:38669) > <sheet>" }
    path_hint:
      ios:     { path: "unknown — new bottom-sheet view controller, same module as form_sheet", provenance: unknown }
      android: { path: "unknown — new bottom-sheet fragment, same module as form_sheet", provenance: unknown }
    find_by: >
      A modal bottom sheet whose header title is exactly the string `"Thành công"` and whose section header carries the string `"Danh sách NOXH phù hợp"`. Grep either literal.
    confirm: >
      The sheet is presented after a successful REQ-02 form submission OR after the AC-04-shortcut fires. Its `noxh_project_card` items are visually similar to the ones on the NOXH Hub `Dự án` tab — this spec builds a NEW card component per Q3 answer, so do NOT try to reuse a Hub card component before confirming with the designer/PM that the two are equivalent (they should converge later, but not in this wave).
    current: >
      Nothing renders here today. New component.
    owns:
      this_section_provides: [the sheet surface, header row, illustration block, notification toggle card (default variant only), section header row, horizontal carousel of project cards]
      therefore: >
        The DS `BottomSheet` container provides the overlay and dismiss chrome — do NOT add a second scrim. Sheet height 671 for default variant, 593 for no-toggle variant; both are content-driven and cap at 85vh (add scroll if the device is shorter than the content).
    padding_check: { inner_x: 16, inner_y: 16, surfaces: 1 }

Layout_blocks:
  # Chain: an AC sends you here → `screen:` names the page → `section:` sends you to `Sections`
  # to locate the node → `action:` says what to do → `what:` describes the finished frame →
  # the tree IS the finished contents.

  # ── Block 1 ──────────────────────────────────────────────────────────
  bottom_action_bar_default:
    screen: primary_ad_detail_mobile
    section: bottom_action_bar
    requirement: REQ-01
    verifies: [AC-01a, AC-01b, AC-01e, AC-01f, AC-01g, AC-01h, AC-01i, AC-01j]
    action: >
      NEW. Create the sticky bottom action bar for NOXH primary Ad Detail. Build the exact tree
      below into the located sticky-footer slot; do not modify any other adview cell above it.
      The tree IS the finished contents — whatever renders today and is not in the tree does not
      survive the build.
    what: >
      A 375-wide sticky footer at the bottom of the ad detail viewport. Row layout, gap 4, side padding 16 and vertical padding 8, on a white surface with a 1px hairline top border. The row holds two buttons: on the left, a 137×40 tonal-neutral pill carrying a bell icon and the Vietnamese label `"Quan tâm"` (icon left, gap 4); on the right, a 202×40 orange primary button carrying a document icon and the label `"Tư vấn hồ sơ"` (icon left, gap 8). Below the row sits a 375×34 home-indicator strip (transparent — the OS renders the actual home indicator on top of it).
    figma: "15222:37507"
    reference_width: 375
    maths: "vertical: 8 + 40 + 8 + 34 = 90 (pad-top 8 + button 40 + pad-bottom 8 + home indicator 34); horizontal row: 16 + 137 + 4 + 202 + 16 = 375 (pad-left 16 + quan_tam 137 + gap 4 + tu_van 202 + pad-right 16)"
    computed: { bottom_action_bar: 375x90, button_group: 375x56, button_row: 343x40, home_indicator: 375x34 }
    tree:
      - id: bottom_action_bar
        type: column
        w: 375
        h: 90
        fill: background-primary
        border_top: "1px border-thin"
        children:
          - id: button_group
            type: row
            w: 375
            h: 56
            gap: 4
            padding: "[8,16,8,16]"
            align_items: center
            children:
              - { id: quan_tam_button, type: use, use: ButtonTonalNeutral_L_40, params: { label: "Quan tâm", icon_left: Bell-line, w: 137, h: 40, radius: 8 }, cr: CR-01a, bh: BH-01a }
              - { id: tu_van_button, type: use, use: ButtonPrimary_L_40, params: { label: "Tư vấn hồ sơ", icon_left: Document-fill, w: 202, h: 40, radius: 8 }, cr: CR-01b, bh: BH-01c }
          - { id: home_indicator, type: stack, w: 375, h: 34, fill: background-primary }

  # ── Block 2 ──────────────────────────────────────────────────────────
  bottom_action_bar_followed:
    screen: primary_ad_detail_mobile
    section: bottom_action_bar
    requirement: REQ-01
    verifies: [AC-01a, AC-01c, AC-01e, AC-01f, AC-01g, AC-01h, AC-01j]
    action: >
      NEW. Same section node as Block 1; render this variant when `data_bound.quan_tam_state ==
      followed`. Build the exact tree below into the located sticky-footer slot; the tree IS the
      finished contents.
    what: >
      A 375-wide sticky footer, same chrome as Block 1. Row layout, gap 4, side padding 16 and vertical padding 8. The bell button shrinks to a 56×40 icon-only pill (tonal-neutral fill, radius 8) and gains a small green check-badge overlay in the bottom-right corner (8pt filled circle in text-success `#12a154`, 1pt border-blank outline). The `"Tư vấn hồ sơ"` orange button grows to 283×40 filling the freed horizontal space.
    figma: "15222:37982"
    reference_width: 375
    maths: "vertical: 8 + 40 + 8 + 34 = 90; horizontal row: 16 + 56 + 4 + 283 + 16 = 375"
    computed: { bottom_action_bar: 375x90, button_group: 375x56, button_row: 343x40, home_indicator: 375x34 }
    tree:
      - id: bottom_action_bar
        type: column
        w: 375
        h: 90
        fill: background-primary
        border_top: "1px border-thin"
        children:
          - id: button_group
            type: row
            w: 375
            h: 56
            gap: 4
            padding: "[8,16,8,16]"
            align_items: center
            children:
              - id: quan_tam_button
                type: stack
                w: 56
                h: 40
                children:
                  - { id: quan_tam_icon_only, type: use, use: ButtonTonalNeutral_IconOnly_40, params: { icon: Bell-line, w: 56, h: 40, radius: 8 } }
                  - { id: quan_tam_followed_badge, type: circle, w: 8, h: 8, fill: text-success, border: "1px border-blank", position: "absolute", offset: "right 4, bottom 4" }
                bh: BH-01b
              - { id: tu_van_button, type: use, use: ButtonPrimary_L_40, params: { label: "Tư vấn hồ sơ", icon_left: Document-fill, w: 283, h: 40, radius: 8 }, cr: CR-01b, bh: BH-01c }
          - { id: home_indicator, type: stack, w: 375, h: 34, fill: background-primary }

  # ── Block 3 ──────────────────────────────────────────────────────────
  bottom_action_bar_registered:
    screen: primary_ad_detail_mobile
    section: bottom_action_bar
    requirement: REQ-01
    verifies: [AC-01a, AC-01d, AC-01i, AC-01j]
    action: >
      NEW. Same section node as Blocks 1 and 2; render this variant when
      `data_bound.registered_state == registered`. Build the exact tree below into the located
      sticky-footer slot; the tree IS the finished contents.
    what: >
      A 375-wide sticky footer, same chrome as Blocks 1 and 2. Row layout, gap 4, side padding 16 and vertical padding 8. The right-side button changes from orange primary to disabled tonal (background-secondary fill, text-tertiary label, no icon) and reads `"Đã đăng ký"`. The bell button on the left keeps whichever visual state (`data_bound.quan_tam_state`) it was in — this block can nest either Block 1's or Block 2's bell subtree on the left; the difference from those blocks is only the right button. Not drawn in Figma; inferred from PRD §US1 "the button is disabled for that project".
    figma: "n/a — not drawn in this design; PRD-derived"
    reference_width: 375
    maths: "vertical: 8 + 40 + 8 + 34 = 90; horizontal row: 16 + (137 OR 56) + 4 + (202 OR 283) + 16 = 375 (per bell state; the row math from Block 1 or Block 2 holds unchanged)"
    computed: { bottom_action_bar: 375x90, button_group: 375x56, button_row: 343x40, home_indicator: 375x34 }
    tree:
      - id: bottom_action_bar
        type: column
        w: 375
        h: 90
        fill: background-primary
        border_top: "1px border-thin"
        children:
          - id: button_group
            type: row
            w: 375
            h: 56
            gap: 4
            padding: "[8,16,8,16]"
            align_items: center
            children:
              - { id: quan_tam_button, type: ref, ref_block: "bottom_action_bar_default.quan_tam_button OR bottom_action_bar_followed.quan_tam_button", axis_differs_from: none, note: "renders whichever bell subtree matches data_bound.quan_tam_state" }
              - { id: dang_ky_button_disabled, type: use, use: ButtonTonalNeutral_L_40, params: { label: "Đã đăng ký", text_color: text-tertiary, icon_left: none, disabled: true, w: 202, h: 40, radius: 8 }, cr: CR-01c, bh: BH-01d }
          - { id: home_indicator, type: stack, w: 375, h: 34, fill: background-primary }

  # ── Block 4 ──────────────────────────────────────────────────────────
  form_sheet:
    screen: form_tu_van_sheet
    section: form_sheet
    requirement: REQ-02
    verifies: [AC-02a, AC-02b, AC-02c, AC-02d, AC-02e, AC-02f, AC-02g, AC-02h, AC-02i, AC-02j, AC-02k]
    action: >
      NEW. Build the entire form bottom sheet into the located sheet slot. The tree IS the
      finished contents — whatever renders today and is not in the tree does not survive the
      build. The DS `BottomSheet` container provides the overlay and dismiss chrome; do not add
      a second scrim inside this tree.
    what: >
      A 375-wide bottom sheet 438pt tall, top corners rounded 12, white fill. At the top a 48pt drawer header with an X close icon on the left and a centered title `"Tư vấn hồ sơ"` (label-page 16/24 Bold). Below the header a scrollable body 318pt tall with 16pt inner padding on all four sides and 16pt vertical gaps: (1) a 343×78 positive-banner card rebuilt as native text over a coloured background (see OI-6) reading a bold line `"Để lại thông tin liên hệ"` above an orange helper line `"Giúp chúng tôi có thể tư vấn cho bạn kỹ hơn!"`, radius 12; (2) a 343×104 input group holding two 343×48 outlined inputs stacked with 8pt gap — the first labelled `"Họ và tên *"` with the clear-icon slot, the second labelled `"Nhập số điện thoại *"` (phone keyboard); (3) a 343×72 consent row, shield-check icon left (20×20 in text-success), a body-caption paragraph right (319×72) containing the consent copy verbatim with the `"Chính sách bảo mật của Nhà Tốt"` fragment as a tappable link. At the bottom of the sheet a 375×72 docked footer holding a single full-width `"Gửi thông tin"` orange CTA (40pt tall, radius 8) — DS `Bottom Button` `"2 button"` variant with the secondary slot hidden (OI-7).
    figma: "15222:37512"
    reference_width: 375
    maths: "sheet vertical: 48 + 318 + 72 = 438 (header + body + footer); body vertical: 16 + 78 + 16 + 104 + 16 + 72 + 16 = 318 (pad + banner + gap + inputs + gap + consent + pad); input_group vertical: 48 + 8 + 48 = 104; consent horizontal: 20 + 4 + 319 = 343"
    computed: { form_sheet: 375x438, drawer_header: 375x48, body: 375x318, positive_banner: 343x78, input_group: 343x104, input_1_ho_ten: 343x48, input_2_phone: 343x48, consent_row: 343x72, consent_text: 319x72, footer: 375x72, gui_thong_tin_cta: 343x40 }
    tree:
      - id: form_sheet
        type: column
        w: 375
        h: 438
        fill: background-primary
        radius: "12 top-left, 12 top-right, 0 bottom-left, 0 bottom-right"
        children:
          - id: drawer_header
            type: row
            w: 375
            h: 48
            gap: 8
            padding: "[12,20,12,20]"
            align_items: center
            border_bottom: "1px border-thin"
            children:
              - { id: sheet_close_icon, type: icon, name: X, size: 24, color: icon-primary, bh: BH-02a }
              - { id: sheet_title, type: text, text: "Tư vấn hồ sơ", ty: label-page, w: 271, align: center, cr: CR-02g }
              - { id: sheet_header_right_spacer, type: stack, w: 24, h: 24 }
          - id: body
            type: column
            w: 375
            h: 318
            gap: 16
            padding: "[16,16,16,16]"
            children:
              - id: positive_banner
                type: column
                w: 343
                h: 78
                radius: 12
                fill: "linear-gradient(peach — see OI-6 for asset rebuild)"
                padding: "[16,16,16,16]"
                gap: 2
                children:
                  - { id: banner_primary_line, type: text, text: "Để lại thông tin liên hệ", ty: label-section, color: text-primary, cr: CR-02a }
                  - { id: banner_helper_line, type: text, text: "Giúp chúng tôi có thể tư vấn cho bạn kỹ hơn!", ty: label-section, color: button-primary, cr: CR-02b }
              - id: input_group
                type: column
                w: 343
                h: 104
                gap: 8
                children:
                  - { id: input_1_ho_ten, type: use, use: InputField, params: { label: "Họ và tên", required: true, fill_text: true, clear_icon: true, w: 343, h: 48, value_source: data, value: "{account_full_name}" }, cr: CR-02c }
                  - { id: input_2_phone, type: use, use: InputField, params: { label: "Nhập số điện thoại", required: true, fill_text: false, clear_icon: false, keyboard: phone, w: 343, h: 48, value_source: data, value: "{account_phone}" }, cr: CR-02d }
              - id: consent_row
                type: row
                w: 343
                h: 72
                gap: 4
                align_items: start
                children:
                  - { id: consent_icon, type: icon, name: Shield-check, size: 20, color: text-success }
                  - { id: consent_text, type: text, w: 319, h: 72, ty: body-caption, color: text-secondary, text: "Bằng việc gửi thông tin, bạn đồng ý với Chính sách bảo mật của Nhà Tốt và cho phép Nhà Tốt thu thập, Nhà Tốt xin cam kết thông tin sẽ được bảo mật và chỉ phục vụ cho việc liên hệ tư vấn dịch vụ theo nhu cầu của người mua", link_range: "Chính sách bảo mật của Nhà Tốt", link_color: text-link, cr: CR-02e, bh: BH-02b }
          - id: footer
            type: column
            w: 375
            h: 72
            padding: "[16,16,16,16]"
            children:
              - { id: gui_thong_tin_cta, type: use, use: ButtonPrimary_L_40, params: { label: "Gửi thông tin", icon_left: none, w: 343, h: 40, radius: 8 }, cr: CR-02f, bh: BH-02c }

  # ── Block 5 ──────────────────────────────────────────────────────────
  success_sheet_default:
    screen: success_tu_van_sheet
    section: success_sheet
    requirement: REQ-03
    verifies: [AC-03a, AC-03b, AC-03c, AC-03e, AC-03f, AC-03g, AC-03h, AC-03i, AC-03j, AC-03k, AC-03l]
    action: >
      NEW. Build the entire success bottom sheet (default variant, with notification toggle
      card) into the located sheet slot. The tree IS the finished contents. The DS `BottomSheet`
      container provides the overlay and dismiss chrome — do not add a second scrim inside.
    what: >
      A 375-wide bottom sheet 671pt tall, top corners rounded 12, white fill. At the top a 48pt drawer header with an X close icon on the left and a centered title `"Thành công"`. Below it a 375×207 hero band with a peach gradient background (fading down from a warm peach at the top), a success line `"🎉 Tư vấn viên sẽ liên hệ lại bạn sớm nhất."` centered near the top, and a 96×96 clapping-hands illustration centered near the bottom. Below the hero a 375×416 list section with 16pt inner padding and 12pt vertical gaps, holding in order: a 343×66 notification-toggle card (background-app fill, radius 12, 16pt inner horizontal padding and 12pt vertical, 4pt horizontal gap between an inner column and the toggle switch — the column stacks the title `"Đồng ý nhận thông báo"` above the subtitle `"Để luôn cập nhập thông tin mới nhất từ dự án này"`; the toggle on the right is the DS Switch at 36×20, defaulting to ON); a 343-wide hairline divider; a 343×32 section header row containing the label-page title `"Danh sách NOXH phù hợp"` on the left and a 108×32 outlined-pill `"Xem tất cả →"` button on the right; and a horizontal-scroll carousel of NOXH project cards (each card 153×252 radius 12, first card left-aligned to the sheet's 16pt inner padding, subsequent cards separated by 12pt gap; the last visible card's right edge bleeds off the viewport).
    figma: "15222:37531"
    reference_width: 375
    maths: "sheet vertical: 48 + 623 = 671 (header + body); body vertical: 207 + 416 = 623 (hero + list); list vertical: 16 + 66 + 12 + 0 + 12 + 32 + 12 + 252 + 16 = 418 (Figma reports 416 — 2pt rounding on the divider gap; treat 16-based values as authoritative); toggle-row horizontal: 16 + 271 + 4 + 36 + 16 = 343; header-row horizontal: 223 + 12 + 108 = 343"
    computed: { success_sheet: 375x671, drawer_header: 375x48, body_content: 375x623, hero_band: 375x207, list_section: 375x416, toggle_card: 343x66, toggle_inner_col: 271x42, toggle_switch: 36x20, divider: 343x1, danh_sach_header_row: 343x32, danh_sach_title: 223x24, xem_tat_ca_pill: 108x32, carousel: 648x252, project_card: 153x252 }
    tree:
      - id: success_sheet
        type: column
        w: 375
        h: 671
        fill: background-primary
        radius: "12 top-left, 12 top-right, 0 bottom-left, 0 bottom-right"
        children:
          - id: drawer_header
            type: row
            w: 375
            h: 48
            gap: 8
            padding: "[12,20,12,20]"
            align_items: center
            border_bottom: "1px border-thin"
            children:
              - { id: sheet_close_icon, type: icon, name: X, size: 24, color: icon-primary, bh: BH-03a }
              - { id: sheet_title, type: text, text: "Thành công", ty: label-page, w: 271, align: center }
              - { id: sheet_header_right_spacer, type: stack, w: 24, h: 24 }
          - id: body_content
            type: column
            w: 375
            h: 623
            children:
              - id: hero_band
                type: stack
                w: 375
                h: 207
                fill: "linear-gradient(top peach → bottom background-primary)"
                children:
                  - { id: hero_success_line, type: text, text: "🎉 Tư vấn viên sẽ liên hệ lại bạn sớm nhất.", ty: body-section, color: text-primary, align: center, position: "absolute, top 44, left 0, right 0", cr: CR-03a }
                  - { id: hero_illustration, type: image, name: success_clapping_hands, w: 96, h: 96, position: "absolute, top 96, left 140" }
              - id: list_section
                type: column
                w: 375
                h: 416
                gap: 12
                padding: "[16,16,16,16]"
                children:
                  - id: toggle_card
                    type: row
                    w: 343
                    h: 66
                    gap: 4
                    padding: "[12,16,12,16]"
                    radius: 12
                    fill: background-app
                    align_items: center
                    children:
                      - id: toggle_inner_col
                        type: column
                        w: 271
                        h: 42
                        gap: 0
                        children:
                          - { id: toggle_title, type: text, text: "Đồng ý nhận thông báo", ty: label-page, color: text-primary, cr: CR-03b }
                          - { id: toggle_subtitle, type: text, text: "Để luôn cập nhập thông tin mới nhất từ dự án này", ty: body-caption, color: text-secondary, cr: CR-03c }
                      - { id: toggle_switch, type: use, use: Switch, params: { on: true, w: 36, h: 20 }, bh: BH-03b }
                  - { id: divider, type: stack, w: 343, h: 1, fill: border-divider }
                  - id: danh_sach_header_row
                    type: row
                    w: 343
                    h: 32
                    gap: 12
                    align_items: center
                    justify: "space-between"
                    children:
                      - { id: danh_sach_title, type: text, text: "Danh sách NOXH phù hợp", ty: label-page, color: text-primary, w: 223, cr: CR-03d }
                      - { id: xem_tat_ca_pill, type: use, use: ButtonPrimary_S_32, params: { label: "Xem tất cả →", w: 108, h: 32, radius: 8 }, cr: CR-03e, bh: BH-03c }
                  - id: carousel
                    type: row
                    w: 375
                    h: 252
                    gap: 12
                    padding: "[0,16,0,16]"
                    overflow_x: scroll
                    align_items: start
                    children:
                      - id: project_card_repeat
                        type: use
                        use: noxh_project_card
                        source: data
                        binds: "{similar_project_list}"
                        params_per_item: { project_name: "{project_name}", price_range: "{price_range}", address_short: "{address_short}", status_badge: "{project_card_status_badge}", photo_count: "{photo_count}", w: 153, h: 252 }
                        cr: [CR-03f, CR-03g, CR-03h, CR-03i, CR-03j, CR-03k]
                        bh: BH-03d

  # ── Block 6 ──────────────────────────────────────────────────────────
  success_sheet_no_toggle:
    screen: success_tu_van_sheet
    section: success_sheet
    requirement: REQ-03
    verifies: [AC-03d, AC-03e, AC-03f, AC-03g, AC-03i, AC-03j, AC-03k, AC-03l]
    action: >
      NEW. Render this variant when `data_bound.toggle_variant_gate == true` (buyer has already
      opted into notifications for this project). Same section node as Block 5; the tree IS the
      finished contents — the notification toggle card AND its adjacent 12pt gap and the hairline
      divider below it are OMITTED. Every other node is identical to Block 5.
    what: >
      A 375-wide bottom sheet 593pt tall (78pt shorter than Block 5 — toggle_card 66 + gap 12 = 78 pt removed), top corners rounded 12, white fill. Header, hero band and section header row are unchanged from Block 5; the list section shrinks from 416 to 338 with the toggle card + divider + surrounding gaps removed, and the section header + carousel shift up 78pt to immediately follow the hero band.
    figma: "15222:38868"
    reference_width: 375
    maths: "sheet vertical: 48 + 545 = 593 (header + body); body vertical: 207 + 338 = 545; list vertical: 16 + 32 + 12 + 252 + 16 = 328 (pad + header + gap + carousel + pad; deep-read reports 338 — 10pt extra unexplained in Figma, validate on first build; 78pt shift confirmed: list header y_rel moves from 361 to 283)"
    computed: { success_sheet: 375x593, drawer_header: 375x48, body_content: 375x545, hero_band: 375x207, list_section: 375x338, danh_sach_header_row: 343x32, carousel: 648x252, project_card: 153x252 }
    tree:
      - id: success_sheet
        type: column
        w: 375
        h: 593
        fill: background-primary
        radius: "12 top-left, 12 top-right, 0 bottom-left, 0 bottom-right"
        children:
          - { id: drawer_header, type: ref, ref_block: "success_sheet_default.drawer_header" }
          - id: body_content
            type: column
            w: 375
            h: 545
            children:
              - { id: hero_band, type: ref, ref_block: "success_sheet_default.hero_band" }
              - id: list_section
                type: column
                w: 375
                h: 338
                gap: 12
                padding: "[16,16,16,16]"
                children:
                  - { id: danh_sach_header_row, type: ref, ref_block: "success_sheet_default.list_section.danh_sach_header_row" }
                  - { id: carousel, type: ref, ref_block: "success_sheet_default.list_section.carousel" }

  # ── Block 7 (snackbar) ───────────────────────────────────────────────
  # Six-block cap: 6 geometry blocks + 1 snackbar block = 7 total; the two
  # snackbar variants (follow / unfollow) were collapsed into one block with
  # a `per_variant_fields` table because they differ ONLY by their text
  # literal. Reason for the extra block over the six cap: the snackbar is a
  # transient overlay that lives on a different screen slot (viewport
  # anchor, not sheet content) and its position math (19pt above the bar)
  # is load-bearing for AC-04a — folding it into another block would
  # confuse the locator.
  snackbar_toast:
    screen: primary_ad_detail_mobile
    section: snackbar_toast
    requirement: REQ-04
    verifies: [AC-04a, AC-04b, AC-04c, AC-04d, AC-04e]
    action: >
      NEW. Render a DS `.Snackbar` instance positioned 19pt above the sticky bottom action bar.
      Auto-dismiss after 2000ms. Copy varies by variant (see per_variant_fields).
    what: >
      A 343×44 dark rounded rectangle (radius 12, fill background-inverted #222222) with white single-line body-section text, horizontally 16-pad, vertically 12-pad, sitting 19pt above the sticky bottom action bar.
    figma: "15222:37981 (follow variant); unfollow variant not drawn — follows AC-04d per designer answer"
    reference_width: 375
    maths: "vertical: 44 (single-line 20 line-height + 12 pad top + 12 pad bottom); horizontal: 343 (viewport 375 minus 16 outer margin ×2); vertical position: viewport 812 - 90 (bar) - 19 (gap) - 44 (snackbar) = 659 from viewport top"
    computed: { snackbar: 343x44 }
    per_variant_fields:
      follow:   { text: "Thêm dự án nhận thông báo thành công", cr: CR-04a }
      unfollow: { text: "Đã bỏ theo dõi dự án",                  cr: CR-04b }
    tree:
      - { id: snackbar, type: use, use: Snackbar, params: { text: "{per_variant.text}", w: 343, h: 44 }, position: "absolute, left 16, top (viewport_height - 153)", cr: "{per_variant.cr}" }

Verify_checklist:
  0_right_section:
    - "Before editing: the located sticky-footer file is the NOXH-primary branch, NOT the generic sticky lead bar for non-NOXH primary ads (near-miss)."
    - "Before editing: the located bottom-sheet file for `form_sheet` and `success_sheet` is NEW — no such sheet exists today for NOXH. If a bottom sheet with title `\"Tư vấn hồ sơ\"` OR `\"Thành công\"` renders anywhere in the app today, you are looking at a leaked build; stop and report."
    - "The generic sticky lead bar is untouched. Diff the file it lives in."
    - AC: [AC-01a]
  1_padding_ownership:
    - "The sticky bottom action bar owns its own surface (bg + top border). If you see a card nested inside it with its own fill and padding, the surface was built twice — collapse it."
    - "The success sheet's list section has 16pt inner padding on all four sides and 12pt vertical gaps between children. A doubled 16 on either edge means a child added its own margin instead of inheriting the parent's gap."
    - "The scroll view above the sticky bar has bottom contentInset 90 — if the last piece of ad content is obscured by the bar, the inset is missing; do not shrink the bar."
    - AC: [AC-01h, AC-02f, AC-03e]
  2_variant_isolation:
    - "quan_tam_state variants (default/followed) MUST NOT share a mutable button ref — the followed variant is 56pt wide, the default is 137pt wide. A build that toggles the same instance risks the wrong width being persisted."
    - "registered_state (Block 3) is orthogonal to quan_tam_state — a project can be `followed AND registered`. The bell subtree keeps its state under Block 3."
    - "toggle_variant_gate (Block 5 vs Block 6) MUST NOT render both toggle_card AND its absence — an animation between the two is out of scope, use a full re-render."
    - AC: [AC-01b, AC-01c, AC-01d, AC-03c, AC-03d]
  3_computed:
    - "bottom_action_bar height = 90 (56 + 34). If measured height is 124, the DS ButtonGroup added extra top padding — verify the pad token is `[8,16,8,16]`."
    - "form_sheet height = 438 (48 + 318 + 72). If the sheet peeks at a different y, the body's inner padding or gaps were doubled — re-check body maths (16 + 78 + 16 + 104 + 16 + 72 + 16 = 318)."
    - "success_sheet (default) height = 671; (no-toggle) height = 593. Difference is exactly 78pt (toggle 66 + gap 12); if the difference is 66 or 90 the toggle's adjacent gap is misapplied."
    - "carousel width = 648 (extends past the 343 inner viewport); measured width should be greater than viewport-minus-padding. A carousel that measures at 327 means it was mistakenly clipped to the parent's max-width."
    - AC: [AC-01a, AC-02a, AC-03a, AC-03f]
  4_responsive:
    - "The sticky bar is single-line at all viewport widths ≥ 320. On viewport 320 the row math is 8 + 137 + 4 + 202 + 8 = 359 which fits; on wider viewports the row grows proportionally (buttons flex, gap and padding stay literal)."
    - "The bottom sheets adapt height to content up to 85vh; on shorter devices the body scrolls and the footer stays pinned."
    - "The carousel scrolls horizontally on every viewport; cards keep 153×252 literal."
    - AC: [AC-01i, AC-02i]
  5_tokens_and_literals:
    - "Every bare number in the tree was built as written. A value that landed on a token instead is a defect — report the node, the spec value and the value built."
    - "Toggle card fill MUST be `background-app` `#f7f7f7`, NOT `background-secondary` `#f4f4f4`. These two greys are similar but not the same and the design uses the lighter one."
    - "Snackbar radius MUST be 12 (`radius-card`), NOT 8 (`radius-card-small`). Verify against `15222:37981`."
    - "The `Quan tâm` default bell fill MUST be `button-tonal-neutral` `#f4f4f4`, NOT `background-app`. Similar hex, different token."
    - AC: [AC-01j, AC-02j, AC-04b]
  6_components_and_data:
    - "Every data-bound leaf resolves at runtime through its named source. A hardcoded name where the tree says the data drives it is a frozen sample."
    - "similar_project_list carousel repeat: if the response has 0 items, the carousel node is omitted entirely; the section still renders `Danh sách NOXH phù hợp` header + `Xem tất cả →` pill (AC-03k)."
    - "The bell state (`quan_tam_state`) and the register state (`registered_state`) come from TWO different endpoints — do not co-mutate them."
    - "shortcut_active is a server-provided boolean, not a client-side computation. Do NOT infer it from account presence alone."
    - AC: [AC-01e, AC-02d, AC-03h, AC-03k]
  7_content_and_behavior:
    - "Every Vietnamese literal in this file is a real string the UI renders; grep the codebase and confirm exact character-for-character match — including the possible typo `cập nhập` (see OI-5)."
    - "The consent copy link range covers ONLY `\"Chính sách bảo mật của Nhà Tốt\"` — not the surrounding words. If tapping any other part of the paragraph opens the privacy policy, the link range is too wide."
    - "The snackbar auto-dismiss timer is 2000ms. A shorter timer means the user cannot read it in Vietnamese; a longer one blocks the sticky bar too long."
    - "On follow API failure, the bell reverts AND an inline error toast (NOT the DS `.Snackbar`) renders — the visual distinction between success (dark snackbar) and error (red inline toast) is load-bearing."
    - AC: [AC-01a-01d, AC-02c-02g, AC-03a-03d, AC-04d]
  8_report:
    - "Report measured padding + slot gaps on every block."
    - "Report every token name that did not resolve and what replaced it."
    - "Report the new `info-subtle` `#f2f6fc` token discovered on the Flow 3 announcer (OI-8) — is it a real DS token, an alias, or a raw hex?"
    - "Report every bare number that did NOT land as written (see 5_tokens_and_literals)."
    - "Report every data-bound leaf and the endpoint/field that resolved it."
    - "Report the confirmation from the designer on OI-5 (`cập nhập` vs `cập nhật`), OI-6 (raster banner rebuild), and OI-7 (form footer secondary button)."
    - "Report the final `path_hint` for the four sections after the first build — replace `documented` with `verified` and update the sitemap."
```
