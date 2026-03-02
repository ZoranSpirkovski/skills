# Showcase Page Structure

Required layout and organization for every showcase page, regardless of framework.

## Page Layout

The showcase page uses a two-column layout:

```
┌──────────────────────────────────────────────────────┐
│  Design System                              [search] │
├────────────┬─────────────────────────────────────────┤
│            │                                         │
│  Sidebar   │  Main Content                           │
│  (sticky)  │  (scrollable)                           │
│            │                                         │
│  ┌───────┐ │  ┌─────────────────────────────────┐   │
│  │Found- │ │  │ Foundations                      │   │
│  │ations │ │  │ ┌───────┐ ┌───────┐ ┌───────┐  │   │
│  │ Colors│ │  │ │Colors │ │Typo   │ │Space  │  │   │
│  │ Typo  │ │  │ └───────┘ └───────┘ └───────┘  │   │
│  │ Space │ │  └─────────────────────────────────┘   │
│  ├───────┤ │                                         │
│  │Action │ │  ┌─────────────────────────────────┐   │
│  │ Button│ │  │ Actions                          │   │
│  │ Link  │ │  │ ┌───────────────────────────┐   │   │
│  ├───────┤ │  │ │ Button                    │   │   │
│  │Forms  │ │  │ │ Variants / Sizes / States │   │   │
│  │ Input │ │  │ └───────────────────────────┘   │   │
│  │ Select│ │  └─────────────────────────────────┘   │
│  └───────┘ │                                         │
│            │                                         │
└────────────┴─────────────────────────────────────────┘
```

### Sidebar

- **Position:** Sticky, full viewport height, scrollable independently
- **Width:** Fixed (240–280px on desktop)
- **Sections:** Collapsible groups matching the category taxonomy
- **Items:** Anchor links to component sections in the main area
- **Search/filter:** Text input at top that filters sidebar items and scrolls to match
- **Active state:** Highlight the currently visible section (scroll-spy)
- **Mobile:** Collapses to a hamburger menu or top sheet

### Main Content Area

- **Scrollable:** Independent scroll from sidebar
- **Organization:** Sections appear in category order (Foundations first, Layout last)
- **Spacing:** Generous spacing between sections for clear visual separation

## Section Categories (in order)

1. **Foundations** — Colors, typography, spacing, shadows, border radii, icons
2. **Actions** — Buttons, links, icon buttons, button groups, dropdowns
3. **Forms** — Inputs, selects, checkboxes, radios, toggles, date pickers, file uploads
4. **Data Display** — Tables, lists, cards, badges, tags, avatars, stats
5. **Navigation** — Tabs, breadcrumbs, pagination, sidebars, menus, steppers
6. **Feedback** — Alerts, toasts, progress bars, spinners, empty states, skeletons
7. **Overlays** — Modals, dialogs, drawers, tooltips, popovers, command palettes
8. **Layout** — Containers, grids, stacks, dividers, page headers, section headers

## Per-Component Section Structure

Every component section follows this template:

```
┌─────────────────────────────────────────┐
│ Component Name                    [src] │
│ Brief description of what it does       │
├─────────────────────────────────────────┤
│                                         │
│ Variants                                │
│ ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│ │ Primary │ │Secondary│ │ Outline │   │
│ └─────────┘ └─────────┘ └─────────┘   │
│                                         │
│ Sizes                                   │
│ ┌───┐ ┌─────┐ ┌───────┐ ┌─────────┐   │
│ │ S │ │  M  │ │   L   │ │   XL    │   │
│ └───┘ └─────┘ └───────┘ └─────────┘   │
│                                         │
│ States                                  │
│ ┌────────┐ ┌────────┐ ┌────────┐       │
│ │Default │ │Disabled│ │Loading │       │
│ └────────┘ └────────┘ └────────┘       │
│                                         │
└─────────────────────────────────────────┘
```

### Required elements

| Element | Purpose |
|---------|---------|
| **Heading** | Component name as `h2` or `h3` with a slug `id` for anchor linking |
| **Source link** | Optional link to the component source file |
| **Description** | One sentence explaining what the component does and when to use it |
| **Variant grid** | Every visual variant rendered side by side for comparison |
| **Size scale** | Every size option rendered side by side, smallest to largest |
| **State examples** | Default, hover (note), active, focused, disabled, loading, error — whatever states the component supports |
| **Dark mode** | If the project supports dark mode, show both light and dark |

### Rendering rules

- Render **real components**, not screenshots or mocks
- Each example should be **self-contained** — no external dependencies to understand it
- Use **neutral/generic content** in examples (e.g., "Button" not "Submit Order")
- Group related examples with **subtle separators** (borders or spacing, not heavy dividers)
- Add **labels** above or below each example indicating the variant/size/state name

## Foundations Section Structure

The Foundations section is special — it shows design tokens rather than components.

### Colors

Display each named color as a swatch with:
- Color swatch (minimum 48x48px)
- Token name (e.g., `gray-500`)
- Hex value
- Group by palette family (grays, primary, secondary, semantic)

### Typography

Show each level of the type scale:
- The actual rendered text at that size/weight
- Token name, font family, size, weight, line-height

### Spacing

Visual blocks demonstrating each spacing token:
- Colored blocks at each spacing value
- Token name and pixel/rem value

### Shadows

Cards or boxes elevated at each shadow level:
- Visual example of the shadow
- Token name

### Border Radii

Shapes demonstrating each radius token:
- Box with the applied radius
- Token name and value

## Responsive Behavior

| Breakpoint | Layout |
|------------|--------|
| Desktop (1024px+) | Sidebar visible, two-column layout |
| Tablet (768–1023px) | Sidebar collapsible, defaults to hidden |
| Mobile (<768px) | Sidebar as overlay/drawer, single column |

The main content area should be responsive within its constraints — variant grids wrap to multiple rows on narrow screens.
