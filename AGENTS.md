# Windows Contrast Theme Color Compliance

## Scope

Replace hardcoded color values with system color references to support Windows 11 contrast themes.

**Only change colors. Do not add/remove elements, icons, borders, or modify layout.**

---

## Step 1 — Detect Tech Stack

Before making any changes, identify the tech stack by inspecting the project files.

| If you find | Tech stack |
|-------------|-----------|
| `.css` / `.scss` / `.less` / CSS-in-JS | Web / CEF / Electron / WebView2 |
| `.xaml` + WinUI namespace | WinUI 3 / UWP |
| `.xaml` + WPF namespace | WPF |
| `.cs` + `System.Windows.Forms` | WinForms |

Apply the implementation method for the detected stack. If multiple stacks coexist, apply each method to its respective files.

---

## Step 2 — Color Semantics

The 8 semantic slots are universal across all tech stacks. Map every color in the project to one of these slots.

| Slot | Use for |
|------|---------|
| **Window Background** | All backgrounds — page, container, card, panel, modal, sidebar |
| **Window Text** | All non-interactive foreground — text, headings, icons, borders |
| **Button Face** | Interactive element backgrounds — button, input, select, toggle |
| **Button Text** | Interactive element foreground — button text, input text |
| **Highlight** | Hover / selected / active / pressed — background |
| **Highlight Text** | Hover / selected / active / pressed — foreground |
| **Hot Light** | Hyperlinks only |
| **Gray Text** | Disabled state only. Do not use for secondary text. |

> Secondary text (captions, hints, non-disabled) → **Window Text**, not **Gray Text**.  
> **Button Face** equals **Window Background** in all themes. Interactive element borders must use **Button Text** to remain visible.

### SVG Color Rule

SVG `fill` and `stroke` follow the parent container's semantic slot — not a fixed slot.

| SVG location | Slot to use |
|--------------|-------------|
| Inside interactive elements (button, input, toggle) | **Button Text** |
| Inside text / content areas (body, heading, card) | **Window Text** |
| Inside hover / selected / active state | **Highlight Text** |
| Inside disabled elements | **Gray Text** |

Do not assign a fixed slot to all SVGs. Determine the slot from where the SVG lives.

---

## Step 3 — Theme Reference Values

Default hex values per theme. Use for testing and verification only — never hardcode these.

| Slot | Aquatic | Desert | Dusk | Night Sky |
|------|---------|--------|------|-----------|
| Window Background | `#202020` | `#fffaef` | `#2d3236` | `#000000` |
| Window Text | `#ffffff` | `#3d3d3d` | `#ffffff` | `#ffffff` |
| Button Face | `#202020` | `#fffaef` | `#2d3236` | `#000000` |
| Button Text | `#ffffff` | `#202020` | `#b6f6f0` | `#ffee32` |
| Highlight | `#8ee3f0` | `#903909` | `#a1bfde` | `#d6b4fd` |
| Highlight Text | `#263b50` | `#fff5e3` | `#212d3b` | `#2b2b2b` |
| Hot Light | `#75e9fc` | `#1c5e75` | `#70ebde` | `#8080ff` |
| Gray Text | `#a6a6a6` | `#676767` | `#a6a6a6` | `#a6a6a6` |

---

## Step 4 — Implementation by Tech Stack

### Web / CEF / Electron / WebView2 (CSS)

Use CSS system color keywords inside `@media (forced-colors: active)`.

| Slot | CSS Keyword |
|------|-------------|
| Window Background | `Canvas` |
| Window Text | `CanvasText` |
| Button Face | `ButtonFace` |
| Button Text | `ButtonText` |
| Highlight | `Highlight` |
| Highlight Text | `HighlightText` |
| Hot Light | `LinkText` |
| Gray Text | `GrayText` |

```css
/* ✅ DO */
@media (forced-colors: active) {
  .component {
    color: CanvasText;
    background-color: Canvas;
    border-color: ButtonText;
  }
}

/* ❌ DON'T — hardcoded hex values break user-customized themes */
color: #ffffff;
background-color: #1f1f1f;
```

**Border colors:**

| Border type | Keyword |
|-------------|---------|
| Structural borders | `CanvasText` |
| Interactive element borders | `ButtonText` |
| Focus ring | `Highlight` |

**Inline SVG:**

SVG follows the parent container's semantic slot (see Step 2 — SVG Color Rule).

```css
@media (forced-colors: active) {
  /* Inside a button / interactive element → ButtonText */
  button svg, .interactive svg { fill: ButtonText; }

  /* Inside body text / content area → CanvasText */
  p svg, .content svg { fill: CanvasText; }

  /* Inside hover / selected / active state → HighlightText */
  :hover svg, [aria-selected="true"] svg { fill: HighlightText; }
}
```

**Images — dark themes only (Aquatic / Dusk / Night Sky):**

```css
@media (forced-colors: active) and (prefers-color-scheme: dark) {
  img { opacity: 0.8; }
}
```

Skip: externally linked SVG (`<img src="*.svg">`), Canvas element, Video, iframe.

---

### WinUI 3 / UWP (XAML)

Add a `HighContrast` entry to `ResourceDictionary.ThemeDictionaries`. Map each brush to the corresponding `SystemColor` resource.

| Slot | XAML Resource |
|------|--------------|
| Window Background | `{ThemeResource SystemColorWindowColor}` |
| Window Text | `{ThemeResource SystemColorWindowTextColor}` |
| Button Face | `{ThemeResource SystemColorButtonFaceColor}` |
| Button Text | `{ThemeResource SystemColorButtonTextColor}` |
| Highlight | `{ThemeResource SystemColorHighlightColor}` |
| Highlight Text | `{ThemeResource SystemColorHighlightTextColor}` |
| Hot Light | `{ThemeResource SystemColorHotlightColor}` |
| Gray Text | `{ThemeResource SystemColorGrayTextColor}` |

```xaml
<ResourceDictionary.ThemeDictionaries>
    <ResourceDictionary x:Key="Default">
        <SolidColorBrush x:Key="PageBackgroundBrush" Color="#1F1F1F" />
    </ResourceDictionary>
    <ResourceDictionary x:Key="HighContrast">
        <SolidColorBrush x:Key="PageBackgroundBrush"
            Color="{ThemeResource SystemColorWindowColor}" />
    </ResourceDictionary>
</ResourceDictionary.ThemeDictionaries>
```

---

### WPF (XAML + C#)

Use `{DynamicResource}` with `SystemColors` keys so values update automatically when the theme changes.

| Slot | WPF Resource Key |
|------|-----------------|
| Window Background | `{DynamicResource {x:Static SystemColors.WindowBrushKey}}` |
| Window Text | `{DynamicResource {x:Static SystemColors.WindowTextBrushKey}}` |
| Button Face | `{DynamicResource {x:Static SystemColors.ControlBrushKey}}` |
| Button Text | `{DynamicResource {x:Static SystemColors.ControlTextBrushKey}}` |
| Highlight | `{DynamicResource {x:Static SystemColors.HighlightBrushKey}}` |
| Highlight Text | `{DynamicResource {x:Static SystemColors.HighlightTextBrushKey}}` |
| Hot Light | `{DynamicResource {x:Static SystemColors.HotTrackBrushKey}}` |
| Gray Text | `{DynamicResource {x:Static SystemColors.GrayTextBrushKey}}` |

```xaml
<!-- ✅ DO -->
<TextBlock Foreground="{DynamicResource {x:Static SystemColors.WindowTextBrushKey}}" />

<!-- ❌ DON'T -->
<TextBlock Foreground="#FFFFFF" />
```

---

### WinForms (C#)

Use `SystemColors` properties directly.

| Slot | SystemColors Property |
|------|----------------------|
| Window Background | `SystemColors.Window` |
| Window Text | `SystemColors.WindowText` |
| Button Face | `SystemColors.Control` |
| Button Text | `SystemColors.ControlText` |
| Highlight | `SystemColors.Highlight` |
| Highlight Text | `SystemColors.HighlightText` |
| Hot Light | `SystemColors.HotTrack` |
| Gray Text | `SystemColors.GrayText` |

```csharp
// ✅ DO
this.BackColor = SystemColors.Window;
label1.ForeColor = SystemColors.WindowText;
button1.BackColor = SystemColors.Control;
button1.ForeColor = SystemColors.ControlText;

// ❌ DON'T
this.BackColor = Color.FromArgb(31, 31, 31);
```

---

## Checklist

- [ ] Tech stack identified
- [ ] All hardcoded color values replaced with system color references
- [ ] `Gray Text` slot used only for disabled state
- [ ] `Hot Light` slot used only for hyperlinks
- [ ] Interactive element borders use `Button Text` slot
- [ ] Focus rings use `Highlight` slot
- [ ] SVG `fill` / `stroke` uses the slot matching its parent container, not a fixed slot
- [ ] (CSS only) `<img>` elements have `opacity: 0.8` for dark themes (Aquatic / Dusk / Night Sky), not Desert
- [ ] Tested in all four themes: Aquatic / Desert / Dusk / Night Sky

---

## Testing

Enable via: **Settings → Accessibility → Contrast themes**  
Toggle shortcut: `Left Alt + Left Shift + PrtScn`

Test all four themes. Do not test Aquatic only.
