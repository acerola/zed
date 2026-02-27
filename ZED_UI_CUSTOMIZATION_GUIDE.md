# Zed UI Customization Guide

A comprehensive map of every entry point in the Zed codebase you can edit to customize the UI.

---

## Table of Contents

- [1. Theme & Colors](#1-theme--colors)
- [2. UI Component Library](#2-ui-component-library)
- [3. Workspace Layout](#3-workspace-layout)
- [4. Editor Appearance](#4-editor-appearance)
- [5. UI Settings](#5-ui-settings)
- [6. Icons](#6-icons)
- [7. GPUI Primitives](#7-gpui-primitives)
- [8. Quick Reference](#8-quick-reference-i-want-to-change-x)

---

## 1. Theme & Colors

The theme system controls all colors across the entire UI. Themes are loaded as JSON and deserialized into Rust structs.

### Key Files

| What | File | Key Struct / Item |
|------|------|-------------------|
| 100+ UI color tokens (borders, surfaces, text, tabs, scrollbars, etc.) | `crates/theme/src/styles/colors.rs` | `ThemeColors` |
| Status colors (error, warning, created, modified, deleted, conflict) | `crates/theme/src/styles/status.rs` | `StatusColors` |
| Syntax highlighting colors | `crates/theme/src/styles/syntax.rs` | `SyntaxTheme` |
| Accent colors | `crates/theme/src/styles/accents.rs` | `AccentColors` |
| Player/collaboration colors | `crates/theme/src/styles/players.rs` | `PlayerColors` |
| OS-level system colors | `crates/theme/src/styles/system.rs` | `SystemColors` |
| Theme loading & switching | `crates/theme/src/registry.rs` | `ThemeRegistry` |
| Core theme struct & active theme access | `crates/theme/src/theme.rs` | `Theme`, `GlobalTheme`, `ActiveTheme` trait |
| Theme JSON serialization schema | `crates/theme/src/schema.rs` | `ThemeContent`, `ThemeFamilyContent` |
| Icon theme definitions | `crates/theme/src/icon_theme.rs` | `IconTheme` |

### How Themes Are Applied

1. `GlobalTheme::reload_theme()` reloads the theme based on user settings.
2. Components access colors via `cx.theme().colors()`, `cx.theme().status()`, etc.
3. Settings changes trigger `cx.refresh_windows()` to repaint everything.

### Color Format

All colors use `Hsla` (Hue, Saturation, Lightness, Alpha).

---

## 2. UI Component Library

All reusable UI widgets live in `crates/ui/src/components/`. Modify these to change how UI elements look and behave across the entire application.

### Buttons

| Component | File |
|-----------|------|
| `Button` | `crates/ui/src/components/button/button.rs` |
| `IconButton` | `crates/ui/src/components/button/icon_button.rs` |
| `ButtonLike` (base for all styled buttons) | `crates/ui/src/components/button/button_like.rs` |
| `ToggleButton` | `crates/ui/src/components/button/toggle_button.rs` |

### Text & Labels

| Component | File |
|-----------|------|
| `Label` | `crates/ui/src/components/label/label.rs` |
| `HighlightedLabel` | `crates/ui/src/components/label/highlighted_label.rs` |

### Lists

| Component | File |
|-----------|------|
| `List` | `crates/ui/src/components/list/list.rs` |
| `ListItem` | `crates/ui/src/components/list/list_item.rs` |
| `ListHeader` | `crates/ui/src/components/list/list_header.rs` |

### Menus & Popovers

| Component | File |
|-----------|------|
| `ContextMenu` | `crates/ui/src/components/context_menu.rs` |
| `DropdownMenu` | `crates/ui/src/components/dropdown_menu.rs` |
| `Popover` | `crates/ui/src/components/popover.rs` |
| `PopoverMenu` | `crates/ui/src/components/popover_menu.rs` |

### Other Components

| Component | File |
|-----------|------|
| `Tab` | `crates/ui/src/components/tab.rs` |
| `Modal` | `crates/ui/src/components/modal.rs` |
| `Tooltip` | `crates/ui/src/components/tooltip.rs` |
| `Scrollbar` | `crates/ui/src/components/scrollbar.rs` |
| `Notification` | `crates/ui/src/components/notification.rs` |
| `Banner` | `crates/ui/src/components/banner.rs` |
| `Callout` | `crates/ui/src/components/callout.rs` |
| `Icon` | `crates/ui/src/components/icon.rs` |
| `Avatar` | `crates/ui/src/components/avatar.rs` |
| `Progress` | `crates/ui/src/components/progress.rs` |
| `Radio` | `crates/ui/src/components/radio.rs` |
| `Toggle` | `crates/ui/src/components/toggle.rs` |
| `Disclosure` | `crates/ui/src/components/disclosure.rs` |

### Component Pattern

All components implement the `RenderOnce` trait with `#[derive(IntoElement)]` and use a fluent builder pattern with Tailwind-style methods (`.bg()`, `.text_color()`, `.border()`, `.rounded()`, `.p()`, `.m()`, etc.).

---

## 3. Workspace Layout

Controls the overall window structure — panes, docks, toolbar, and status bar.

### Key Files

| What | File | Key Struct / Trait |
|------|------|--------------------|
| Main window layout (panes, docks, modals) | `crates/workspace/src/workspace.rs` | `Workspace` |
| Tab containers / editor panes | `crates/workspace/src/pane.rs` | `Pane` |
| Pane splitting (horizontal/vertical) | `crates/workspace/src/pane_group.rs` | `PaneGroup`, `Member` enum |
| Side/bottom panel docks | `crates/workspace/src/dock.rs` | `Dock`, `Panel` trait, `DockPosition` enum |
| Top toolbar | `crates/workspace/src/toolbar.rs` | `Toolbar`, `ToolbarItemView` trait |
| Bottom status bar | `crates/workspace/src/status_bar.rs` | `StatusBar`, `StatusItemView` trait |
| Modal dialog layer | `crates/workspace/src/modal_layer.rs` | — |
| Toast notifications | `crates/workspace/src/toast_layer.rs` | — |
| Notification system | `crates/workspace/src/notifications.rs` | — |

### Panel Registration

Panels implement the `Panel` trait:

```rust
pub trait Panel: Focusable + EventEmitter<PanelEvent> + Render {
    fn persistent_name() -> &'static str;
    fn position(&self, window: &Window, cx: &App) -> DockPosition;
    fn toggle_icon(&self, cx: &App) -> IconName;
}
```

Panels dock to `DockPosition::Left`, `DockPosition::Right`, or `DockPosition::Bottom`.

---

## 4. Editor Appearance

Controls everything you see inside the code editor.

### Key Files

| What | File | Key Item |
|------|------|----------|
| Full editor rendering (gutter, cursors, selections, line highlights, minimap, scrollbar) | `crates/editor/src/element.rs` | Main `paint()` / layout logic |
| Editor visual settings (cursor shape, line numbers, scrollbar, gutter, minimap) | `crates/editor/src/editor_settings.rs` | `EditorSettings` |
| Code folding transforms | `crates/editor/src/display_map/fold_map.rs` | — |
| Line wrapping transforms | `crates/editor/src/display_map/wrap_map.rs` | — |
| Virtual blocks (inlay hints, file headers) | `crates/editor/src/display_map/block_map.rs` | — |
| Layout dimension calculations | `crates/editor/src/display_map/dimensions.rs` | — |

### Visual Elements in `element.rs`

- **Gutter**: line numbers, breakpoint indicators, fold controls, code action icons
- **Cursors**: block, bar, underline shapes via `CursorShape` enum
- **Selections**: highlighted ranges with theme-driven colors
- **Line highlights**: current line, search matches, diagnostic highlights
- **Minimap**: scaled-down code overview
- **Scrollbar**: with optional git diff and diagnostic indicators
- **Breadcrumbs**: file path and symbol navigation

---

## 5. UI Settings

User-facing settings that control appearance. Add fields here to expose new customization options.

### Theme Settings

**File:** `crates/settings_content/src/theme.rs` — `ThemeSettingsContent`

| Setting | Type | Description |
|---------|------|-------------|
| `ui_font_size` | `FontSize` | Font size for UI elements |
| `ui_font_family` | `FontFamilyName` | Font family for UI elements |
| `buffer_font_size` | `FontSize` | Font size in the editor |
| `buffer_font_family` | `FontFamilyName` | Font family in the editor |
| `buffer_line_height` | `BufferLineHeight` | Line height in the editor |
| `ui_density` | `UiDensity` | Compact / Default / Comfortable |
| `theme` | `ThemeSelection` | Active color theme |
| `icon_theme` | `IconThemeSelection` | Active icon theme |
| `experimental_theme_overrides` | `ThemeStyleContent` | Global color overrides |
| `theme_overrides` | `HashMap<String, ThemeStyleContent>` | Per-theme color overrides |

### Editor Settings

**File:** `crates/editor/src/editor_settings.rs` — `EditorSettings`

| Setting | Type | Description |
|---------|------|-------------|
| `cursor_shape` | `CursorShape` | Block, Bar, or Underline |
| `current_line_highlight` | `CurrentLineHighlight` | How the current line is highlighted |
| `selection_highlight` | `bool` | Whether selections are highlighted |
| `scrollbar` | `Scrollbar` | Scrollbar visibility, git diff, diagnostics |
| `minimap` | `Minimap` | Minimap visibility and behavior |
| `gutter` | `Gutter` | Line numbers, breakpoints, fold controls |
| `toolbar` | `Toolbar` | Editor toolbar visibility |
| `relative_line_numbers` | `RelativeLineNumbers` | Relative line number mode |

### Settings UI

| What | File |
|------|------|
| Font picker | `crates/settings_ui/src/components/font_picker.rs` |
| Theme picker | `crates/settings_ui/src/components/theme_picker.rs` |
| Icon theme picker | `crates/settings_ui/src/components/icon_theme_picker.rs` |
| Settings infrastructure | `crates/settings/src/settings.rs` |

---

## 6. Icons

### Icon Definitions

| What | File | Key Item |
|------|------|----------|
| 300+ named icons (`Ai`, `ArrowDown`, `Bell`, `Check`, `ChevronRight`, etc.) | `crates/icons/src/icons.rs` | `IconName` enum |
| File/folder icons by extension, stem, or name | `crates/file_icons/src/file_icons.rs` | `FileIcons` |
| Icon theme structure | `crates/theme/src/icon_theme.rs` | `IconTheme` |

### Icon Theme Fields

The `IconTheme` struct contains mappings for:

- `file_icons` — icons by file extension
- `file_stems` — icons by file stem (e.g., `Makefile`)
- `file_suffixes` — icons by file suffix
- `directory_icons` — icons for directories
- `named_directory_icons` — icons for specific directory names

---

## 7. GPUI Primitives

The lowest-level rendering layer. Modify these to change fundamental element behavior.

### Core Elements

| Element | File | Description |
|---------|------|-------------|
| `Div` | `crates/gpui/src/elements/div.rs` | Flexbox container (the primary layout element) |
| `StyledText` / `InteractiveText` | `crates/gpui/src/elements/text.rs` | Text rendering with style runs |
| SVG rendering | `crates/gpui/src/elements/svg.rs` | Vector icon rendering |
| Image rendering | `crates/gpui/src/elements/img.rs` | Raster image support |
| Canvas | `crates/gpui/src/elements/canvas.rs` | Custom drawing surface |
| Virtualized list | `crates/gpui/src/elements/list.rs` | Efficient long list rendering |
| Uniform list | `crates/gpui/src/elements/uniform_list.rs` | List with equal-height items |

### Style & Color Utilities

| What | File |
|------|------|
| Color parsing and manipulation | `crates/gpui/src/colors.rs` |
| Interactive element traits | `crates/gpui/src/interactive.rs` |
| Core element trait | `crates/gpui/src/element.rs` |

### Styling API

Elements are styled using Tailwind-like methods:

```rust
div()
    .bg(cx.theme().colors().surface_background)
    .border_1()
    .border_color(cx.theme().colors().border)
    .rounded_md()
    .p_2()
    .text_color(cx.theme().colors().text)
    .child("Hello")
```

---

## 8. Quick Reference: "I want to change X"

| Goal | Where to Edit |
|------|---------------|
| Change a color across the whole UI | `crates/theme/src/styles/colors.rs` — modify `ThemeColors` fields |
| Create a new theme | `crates/theme/src/schema.rs` — follow `ThemeContent` / `ThemeFamilyContent` schema |
| Restyle a button, list, tab, or other widget | `crates/ui/src/components/<component>.rs` |
| Change the main window skeleton | `crates/workspace/src/workspace.rs` — modify `Workspace::render()` |
| Move or resize a panel / dock | `crates/workspace/src/dock.rs` |
| Change the status bar | `crates/workspace/src/status_bar.rs` |
| Change the toolbar | `crates/workspace/src/toolbar.rs` |
| Change editor gutter / cursors / scrollbar | `crates/editor/src/element.rs` |
| Change editor visual settings | `crates/editor/src/editor_settings.rs` |
| Add a new user-facing UI setting | `crates/settings_content/src/theme.rs` — add field to `ThemeSettingsContent` |
| Add a new icon | `crates/icons/src/icons.rs` — add variant to `IconName` |
| Change file/folder icons | `crates/file_icons/src/file_icons.rs` |
| Change how text is rendered | `crates/gpui/src/elements/text.rs` |
| Change flexbox layout behavior | `crates/gpui/src/elements/div.rs` |
| Add a new panel to the sidebar | Implement `Panel` trait from `crates/workspace/src/dock.rs` |
| Add a new status bar item | Implement `StatusItemView` trait from `crates/workspace/src/status_bar.rs` |
