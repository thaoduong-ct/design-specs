# design-specs

Nơi lưu trữ và version-control các design spec (tokens, components, patterns) dùng cho hand-off giữa design và engineering.

## Cấu trúc

```
design-specs/
├── README.md
└── design-spec/
    ├── tokens.md       # Design tokens: color, typography, spacing, radius, shadow…
    ├── components.md   # Component spec: variants, states, props, a11y
    └── patterns.md     # UX patterns: flows, layouts, interaction rules
```

## Cách dùng

- Mỗi khi có thiết kế mới hoặc thay đổi, cập nhật file spec tương ứng.
- Commit theo scope: `tokens:`, `components:`, `patterns:`.
- Reference bằng đường dẫn tương đối, ví dụ `design-spec/tokens.md#color-primary`.

## Trạng thái

Bản khởi tạo — các file trong `design-spec/` là scaffold trống, chờ điền nội dung.
