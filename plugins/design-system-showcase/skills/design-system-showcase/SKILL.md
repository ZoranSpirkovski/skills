---
name: design-system-showcase
description: Use when creating or modifying frontend components, building UI, or when a project needs a living component library page for visual inspection and developer reference
---

# Design System Showcase

## Overview

A living component library page where every UI component is rendered with all its variants, sizes, and states. Developers use it to visually inspect components without navigating to the pages that use them. Designers use it to verify implementation matches intent.

This skill enforces one rule: **if you touch a component, the showcase page reflects the change before the work is done.**

## The Iron Law

```
NO COMPONENT WORK IS COMPLETE WITHOUT A SHOWCASE PAGE ENTRY.
```

This is not optional. This is not "nice to have." This is the definition of done for any component work. A component that exists in the codebase but not on the showcase page is invisible to the team.

**When the user says to skip it:** Surface the rule, explain the cost (10–15 minutes now vs. reverse-engineering later), and recommend compliance. If the user explicitly overrides after hearing the trade-off, comply — but log the component as a known gap so the next audit catches it. Never silently skip.

## Mode Detection

The skill operates in three modes. Auto-detect which one applies:

| Condition | Mode |
|-----------|------|
| No showcase page found in the project | **Bootstrap** |
| Showcase page exists + you are doing component work | **Update** |
| Explicit request to find missing components | **Audit** |

---

## Mode 1: Bootstrap

**Trigger:** No showcase page exists in the project.

### Steps

1. **Detect the framework.** Look for these markers:

   | Framework | Detection signals |
   |-----------|-------------------|
   | Rails | `Gemfile` with `rails`, `app/views/`, `config/routes.rb` |
   | React / Next.js | `package.json` with `react`, `next.config.*` |
   | Vue / Nuxt | `package.json` with `vue`, `nuxt.config.*` |
   | Svelte / SvelteKit | `package.json` with `svelte`, `svelte.config.*` |
   | Astro | `package.json` with `astro`, `astro.config.*` |
   | Plain HTML | None of the above |

2. **Generate the showcase page** using the framework template from `references/framework-templates.md`. The page MUST follow the layout defined in `references/page-structure.md`.

3. **Set up routing** appropriate to the framework (Rails route, Next.js page, etc.).

4. **Seed the Foundations section** with:
   - Color palette (all named colors with swatches)
   - Typography scale (heading levels, body, mono)
   - Spacing scale (visual blocks at each step)
   - Shadows (elevated cards at each level)
   - Border radii (shapes at each token)

5. **Pull from design system file if present.** If `.interface-design/system.md` exists, extract tokens from it for the Foundations section. If not, scan CSS/Tailwind config for token definitions.

6. **Add existing components.** Scan for component files and create a section for each one found:
   - Rails: `app/components/**/*.rb` (ViewComponent)
   - React: `src/components/**/*.tsx`, `app/components/**/*.tsx`
   - Vue: `src/components/**/*.vue`
   - Svelte: `src/lib/components/**/*.svelte`

### Bootstrap Checklist

- [ ] Framework detected and logged
- [ ] Page file created at correct location
- [ ] Route/path configured
- [ ] Foundations section populated
- [ ] All existing components scanned and added
- [ ] Sidebar navigation working with anchor links
- [ ] Page loads and renders without errors

---

## Mode 2: Component Update

**Trigger:** Showcase page exists AND you are creating, modifying, or deleting a component.

### Before claiming completion of any component work:

1. **Check** if the component has a section on the showcase page.
2. **If no section exists**, create one with:
   - Component name as section heading with anchor ID
   - Brief description of what it does
   - All variants rendered side by side
   - All sizes rendered side by side
   - All interactive states (default, hover, active, disabled, loading, error)
   - Dark/light mode variants if applicable
3. **If section exists**, update it to reflect changes:
   - New variants → add to variant grid
   - Changed API/props → update rendered examples
   - Visual changes → examples already reflect them (re-render)
   - Removed variants → remove from showcase
4. **Update the sidebar** — add new entries to the correct category, remove deleted ones.

### Hard Gate

```
STOP. Before marking this component work as complete:
- [ ] Showcase page section exists for this component
- [ ] All variants are rendered
- [ ] All states are shown
- [ ] Sidebar navigation updated
- [ ] Page still loads without errors

If ANY box is unchecked, the work is NOT complete.
```

### Which Category?

Place each component in the correct section:

| Category | Components |
|----------|-----------|
| **Foundations** | Colors, typography, spacing, shadows, radii, icons |
| **Actions** | Buttons, links, icon buttons, button groups, dropdowns |
| **Forms** | Inputs, selects, checkboxes, radios, toggles, date pickers, file uploads |
| **Data Display** | Tables, lists, cards, badges, tags, avatars, stats, descriptions |
| **Navigation** | Tabs, breadcrumbs, pagination, sidebars, menus, steppers |
| **Feedback** | Alerts, toasts, progress bars, spinners, empty states, skeletons |
| **Overlays** | Modals, dialogs, drawers, tooltips, popovers, command palettes |
| **Layout** | Containers, grids, stacks, dividers, page headers, section headers |

---

## Mode 3: Audit

**Trigger:** Explicit request to audit the showcase page for missing components.

### Steps

1. **Scan for component files** in the project:
   - Rails ViewComponents: `app/components/**/*.rb`
   - React components: `src/components/**/*.tsx`, `app/components/**/*.tsx`
   - Vue components: `src/components/**/*.vue`, `components/**/*.vue`
   - Svelte components: `src/lib/components/**/*.svelte`
   - Generic: any `components/` directory

2. **Parse the showcase page** — extract all component section headings/IDs.

3. **Compare.** For each component file:
   - Does a corresponding section exist on the showcase page?
   - Is the section name consistent with the component name?

4. **Report gaps:**

   ```
   SHOWCASE AUDIT RESULTS
   ========================
   Components found: 24
   On showcase page:  18
   Missing:            6

   MISSING COMPONENTS:
   - ButtonGroup (app/components/button_group_component.rb)
   - DatePicker (app/components/date_picker_component.rb)
   - EmptyState (app/components/empty_state_component.rb)
   - Skeleton (app/components/skeleton_component.rb)
   - CommandPalette (app/components/command_palette_component.rb)
   - Stepper (app/components/stepper_component.rb)
   ```

5. **Offer to generate entries** for each missing component. For each one:
   - Read the component source to understand variants and API
   - Generate a showcase section with all variants and states
   - Add to the correct category
   - Update sidebar navigation

---

## Integration with interface-design

This skill works independently but integrates with the `interface-design` skill when present:

| Condition | Behavior |
|-----------|----------|
| `.interface-design/system.md` exists | Foundations section pulls tokens from it |
| `interface-design` skill is active | Defer to its token definitions — don't invent new ones |
| Neither present | Scan CSS/Tailwind config for tokens, or prompt the user |

When both skills are active:
- `interface-design` owns the design *decisions* (tokens, patterns, rules)
- `design-system-showcase` owns the *page* (rendering, layout, completeness)

---

## Red Flags — Common Rationalizations

When you catch yourself thinking any of these, STOP:

| Rationalization | Reality |
|----------------|---------|
| "I'll add it to the showcase page later" | No you won't. Do it now. |
| "This component is too simple for the showcase" | Simple components still need visual verification. |
| "The showcase page is for designers, not me" | It's for anyone who touches UI. That includes you. |
| "I just changed the padding, the showcase doesn't need updating" | The showcase renders the component. Padding changes show automatically if you re-render. Verify it. |
| "There's no showcase page in this project" | Then you're in Bootstrap mode. Create one. |
| "This is a one-off component, not reusable" | If it's in the components directory, it goes on the showcase. |
| "I'll batch-update the showcase when I'm done with all components" | Each component gets added as it's completed. No batching. |
| "The component isn't ready for the showcase yet" | If it's ready to use in the app, it's ready for the showcase. |
| "Updating the showcase slows me down" | Discovering broken components in production slows everyone down more. |
| "The user explicitly told me to skip it" | Surface the rule and the cost. Comply only after the user confirms with full context. |
| "It's just documentation, not real functionality" | The showcase is a verification gate, not documentation. It catches visual regressions. |
| "I'll note it as a TODO in the commit message" | TODOs in commit messages are where work goes to die. Do it now. |
| "The responsibility shifts to the user if they override" | Your job is to make the cost visible. Silent compliance helps no one. |
