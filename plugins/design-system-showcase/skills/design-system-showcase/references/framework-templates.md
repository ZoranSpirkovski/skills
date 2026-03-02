# Framework Templates

Bootstrap templates for creating the showcase page in each supported framework. Each template includes routing setup, page skeleton, an example component section, and sidebar navigation.

---

## Rails (ViewComponent + ERB)

### Detection

- `Gemfile` contains `rails`
- `app/views/` directory exists
- `config/routes.rb` exists

### Routing

```ruby
# config/routes.rb
get "design-system", to: "design_system#show"
```

```ruby
# app/controllers/design_system_controller.rb
class DesignSystemController < ApplicationController
  layout "application" # or a minimal layout without app chrome
end
```

### Page location

```
app/views/design_system/show.html.erb
```

### Skeleton

```erb
<div class="flex h-screen">
  <!-- Sidebar -->
  <nav class="w-64 shrink-0 border-r overflow-y-auto sticky top-0 h-screen p-4"
       data-controller="design-system-sidebar">
    <h1 class="text-lg font-bold mb-4">Design System</h1>

    <input type="text"
           placeholder="Filter..."
           class="w-full mb-4 px-3 py-2 border rounded"
           data-design-system-sidebar-target="filter"
           data-action="input->design-system-sidebar#filter">

    <!-- Category: Foundations -->
    <div class="mb-3">
      <button class="font-semibold text-sm w-full text-left"
              data-action="click->design-system-sidebar#toggle">
        Foundations
      </button>
      <ul class="ml-2 mt-1 space-y-1" data-design-system-sidebar-target="section">
        <li><a href="#colors" class="text-sm hover:underline">Colors</a></li>
        <li><a href="#typography" class="text-sm hover:underline">Typography</a></li>
        <li><a href="#spacing" class="text-sm hover:underline">Spacing</a></li>
      </ul>
    </div>

    <!-- Repeat for each category -->
  </nav>

  <!-- Main content -->
  <main class="flex-1 overflow-y-auto p-8">
    <section id="foundations" class="mb-16">
      <h2 class="text-2xl font-bold mb-8">Foundations</h2>

      <div id="colors" class="mb-12">
        <h3 class="text-xl font-semibold mb-4">Colors</h3>
        <!-- Color swatches rendered here -->
      </div>
    </section>

    <!-- Component sections follow -->
  </main>
</div>
```

### Stimulus controller for sidebar

```javascript
// app/javascript/controllers/design_system_sidebar_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["filter", "section"]

  filter() {
    const query = this.filterTarget.value.toLowerCase()
    this.element.querySelectorAll("a").forEach(link => {
      const match = link.textContent.toLowerCase().includes(query)
      link.closest("li").style.display = match ? "" : "none"
    })
  }

  toggle(event) {
    const section = event.target.nextElementSibling
    section.classList.toggle("hidden")
  }
}
```

### Example component section (Rails ViewComponent)

```erb
<section id="buttons" class="mb-12">
  <h3 class="text-xl font-semibold mb-2">Button</h3>
  <p class="text-gray-600 mb-6">Triggers an action. Use for primary calls-to-action, form submissions, and interactive controls.</p>

  <h4 class="text-sm font-medium text-gray-500 uppercase tracking-wide mb-3">Variants</h4>
  <div class="flex flex-wrap gap-4 mb-8">
    <%= render ButtonComponent.new(variant: :primary) { "Primary" } %>
    <%= render ButtonComponent.new(variant: :secondary) { "Secondary" } %>
    <%= render ButtonComponent.new(variant: :outline) { "Outline" } %>
    <%= render ButtonComponent.new(variant: :ghost) { "Ghost" } %>
    <%= render ButtonComponent.new(variant: :danger) { "Danger" } %>
  </div>

  <h4 class="text-sm font-medium text-gray-500 uppercase tracking-wide mb-3">Sizes</h4>
  <div class="flex flex-wrap items-center gap-4 mb-8">
    <%= render ButtonComponent.new(size: :sm) { "Small" } %>
    <%= render ButtonComponent.new(size: :md) { "Medium" } %>
    <%= render ButtonComponent.new(size: :lg) { "Large" } %>
  </div>

  <h4 class="text-sm font-medium text-gray-500 uppercase tracking-wide mb-3">States</h4>
  <div class="flex flex-wrap gap-4">
    <%= render ButtonComponent.new { "Default" } %>
    <%= render ButtonComponent.new(disabled: true) { "Disabled" } %>
    <%= render ButtonComponent.new(loading: true) { "Loading" } %>
  </div>
</section>
```

---

## React / Next.js

### Detection

- `package.json` contains `react`
- `next.config.*` exists (Next.js) or `src/` directory with React files

### Routing

**Next.js App Router:**
```
app/design-system/page.tsx
```

**Next.js Pages Router:**
```
pages/design-system.tsx
```

**React Router:**
```tsx
<Route path="/design-system" element={<DesignSystemPage />} />
```

### Skeleton

```tsx
"use client" // Next.js App Router

import { useState } from "react"

const categories = [
  { name: "Foundations", items: ["Colors", "Typography", "Spacing"] },
  { name: "Actions", items: ["Button", "IconButton"] },
  { name: "Forms", items: ["Input", "Select", "Checkbox"] },
  // ... remaining categories
]

export default function DesignSystemPage() {
  const [filter, setFilter] = useState("")

  const filtered = categories
    .map(cat => ({
      ...cat,
      items: cat.items.filter(i => i.toLowerCase().includes(filter.toLowerCase()))
    }))
    .filter(cat => cat.items.length > 0)

  return (
    <div className="flex h-screen">
      {/* Sidebar */}
      <nav className="w-64 shrink-0 border-r overflow-y-auto sticky top-0 h-screen p-4">
        <h1 className="text-lg font-bold mb-4">Design System</h1>
        <input
          type="text"
          placeholder="Filter..."
          className="w-full mb-4 px-3 py-2 border rounded"
          value={filter}
          onChange={e => setFilter(e.target.value)}
        />
        {filtered.map(cat => (
          <div key={cat.name} className="mb-3">
            <span className="font-semibold text-sm">{cat.name}</span>
            <ul className="ml-2 mt-1 space-y-1">
              {cat.items.map(item => (
                <li key={item}>
                  <a href={`#${item.toLowerCase()}`} className="text-sm hover:underline">
                    {item}
                  </a>
                </li>
              ))}
            </ul>
          </div>
        ))}
      </nav>

      {/* Main */}
      <main className="flex-1 overflow-y-auto p-8">
        {/* Sections rendered here */}
      </main>
    </div>
  )
}
```

### Example component section

```tsx
<section id="button" className="mb-12">
  <h3 className="text-xl font-semibold mb-2">Button</h3>
  <p className="text-gray-600 mb-6">Triggers an action.</p>

  <h4 className="text-sm font-medium text-gray-500 uppercase tracking-wide mb-3">Variants</h4>
  <div className="flex flex-wrap gap-4 mb-8">
    <Button variant="primary">Primary</Button>
    <Button variant="secondary">Secondary</Button>
    <Button variant="outline">Outline</Button>
  </div>

  <h4 className="text-sm font-medium text-gray-500 uppercase tracking-wide mb-3">Sizes</h4>
  <div className="flex flex-wrap items-center gap-4 mb-8">
    <Button size="sm">Small</Button>
    <Button size="md">Medium</Button>
    <Button size="lg">Large</Button>
  </div>

  <h4 className="text-sm font-medium text-gray-500 uppercase tracking-wide mb-3">States</h4>
  <div className="flex flex-wrap gap-4">
    <Button>Default</Button>
    <Button disabled>Disabled</Button>
    <Button loading>Loading</Button>
  </div>
</section>
```

---

## Vue / Nuxt

### Detection

- `package.json` contains `vue`
- `nuxt.config.*` exists (Nuxt)

### Routing

**Nuxt:**
```
pages/design-system.vue
```

**Vue Router:**
```js
{ path: '/design-system', component: () => import('./pages/DesignSystem.vue') }
```

### Skeleton

```vue
<template>
  <div class="flex h-screen">
    <nav class="w-64 shrink-0 border-r overflow-y-auto sticky top-0 h-screen p-4">
      <h1 class="text-lg font-bold mb-4">Design System</h1>
      <input
        v-model="filter"
        type="text"
        placeholder="Filter..."
        class="w-full mb-4 px-3 py-2 border rounded"
      />
      <div v-for="cat in filteredCategories" :key="cat.name" class="mb-3">
        <span class="font-semibold text-sm">{{ cat.name }}</span>
        <ul class="ml-2 mt-1 space-y-1">
          <li v-for="item in cat.items" :key="item">
            <a :href="`#${item.toLowerCase()}`" class="text-sm hover:underline">
              {{ item }}
            </a>
          </li>
        </ul>
      </div>
    </nav>

    <main class="flex-1 overflow-y-auto p-8">
      <!-- Sections here -->
    </main>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const filter = ref('')
const categories = [
  { name: 'Foundations', items: ['Colors', 'Typography', 'Spacing'] },
  { name: 'Actions', items: ['Button', 'IconButton'] },
]

const filteredCategories = computed(() =>
  categories
    .map(cat => ({
      ...cat,
      items: cat.items.filter(i => i.toLowerCase().includes(filter.value.toLowerCase()))
    }))
    .filter(cat => cat.items.length > 0)
)
</script>
```

---

## Svelte / SvelteKit

### Detection

- `package.json` contains `svelte`
- `svelte.config.*` exists

### Routing

**SvelteKit:**
```
src/routes/design-system/+page.svelte
```

### Skeleton

```svelte
<script>
  let filter = ''

  const categories = [
    { name: 'Foundations', items: ['Colors', 'Typography', 'Spacing'] },
    { name: 'Actions', items: ['Button', 'IconButton'] },
  ]

  $: filtered = categories
    .map(cat => ({
      ...cat,
      items: cat.items.filter(i => i.toLowerCase().includes(filter.toLowerCase()))
    }))
    .filter(cat => cat.items.length > 0)
</script>

<div class="flex h-screen">
  <nav class="w-64 shrink-0 border-r overflow-y-auto sticky top-0 h-screen p-4">
    <h1 class="text-lg font-bold mb-4">Design System</h1>
    <input
      type="text"
      placeholder="Filter..."
      class="w-full mb-4 px-3 py-2 border rounded"
      bind:value={filter}
    />
    {#each filtered as cat}
      <div class="mb-3">
        <span class="font-semibold text-sm">{cat.name}</span>
        <ul class="ml-2 mt-1 space-y-1">
          {#each cat.items as item}
            <li><a href="#{item.toLowerCase()}" class="text-sm hover:underline">{item}</a></li>
          {/each}
        </ul>
      </div>
    {/each}
  </nav>

  <main class="flex-1 overflow-y-auto p-8">
    <!-- Sections here -->
  </main>
</div>
```

---

## Astro

### Detection

- `package.json` contains `astro`
- `astro.config.*` exists

### Routing

```
src/pages/design-system.astro
```

### Skeleton

```astro
---
// Import framework components as islands
// import Button from '../components/Button.svelte'
---

<html>
<body>
  <div class="flex h-screen">
    <nav class="w-64 shrink-0 border-r overflow-y-auto sticky top-0 h-screen p-4">
      <h1 class="text-lg font-bold mb-4">Design System</h1>
      <div class="mb-3">
        <span class="font-semibold text-sm">Foundations</span>
        <ul class="ml-2 mt-1 space-y-1">
          <li><a href="#colors" class="text-sm hover:underline">Colors</a></li>
          <li><a href="#typography" class="text-sm hover:underline">Typography</a></li>
        </ul>
      </div>
    </nav>

    <main class="flex-1 overflow-y-auto p-8">
      <!-- Use client:load for interactive component islands -->
      <!-- <Button client:load variant="primary">Click me</Button> -->
    </main>
  </div>
</body>
</html>
```

---

## Plain HTML

### Detection

No framework markers found.

### Page location

```
design-system.html    (or design-system/index.html)
```

### Skeleton

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Design System</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { font-family: system-ui, sans-serif; }
    .layout { display: flex; height: 100vh; }
    .sidebar {
      width: 260px; flex-shrink: 0; border-right: 1px solid #e5e5e5;
      overflow-y: auto; position: sticky; top: 0; height: 100vh; padding: 1rem;
    }
    .main { flex: 1; overflow-y: auto; padding: 2rem; }
    .sidebar h1 { font-size: 1.125rem; font-weight: 700; margin-bottom: 1rem; }
    .sidebar input {
      width: 100%; margin-bottom: 1rem; padding: 0.5rem 0.75rem;
      border: 1px solid #d4d4d4; border-radius: 0.375rem;
    }
    .sidebar ul { list-style: none; margin-left: 0.5rem; }
    .sidebar a { font-size: 0.875rem; text-decoration: none; color: #374151; }
    .sidebar a:hover { text-decoration: underline; }
    .category-title { font-size: 0.875rem; font-weight: 600; margin-bottom: 0.25rem; }
    section { margin-bottom: 3rem; }

    @media (max-width: 768px) {
      .sidebar { display: none; }
      .main { padding: 1rem; }
    }
  </style>
</head>
<body>
  <div class="layout">
    <nav class="sidebar">
      <h1>Design System</h1>
      <input type="text" placeholder="Filter..." id="sidebar-filter">
      <div>
        <p class="category-title">Foundations</p>
        <ul>
          <li><a href="#colors">Colors</a></li>
          <li><a href="#typography">Typography</a></li>
          <li><a href="#spacing">Spacing</a></li>
        </ul>
      </div>
    </nav>

    <main class="main">
      <section id="colors">
        <h2>Colors</h2>
        <!-- Color swatches -->
      </section>
    </main>
  </div>

  <script>
    document.getElementById('sidebar-filter').addEventListener('input', function() {
      const query = this.value.toLowerCase()
      document.querySelectorAll('.sidebar a').forEach(link => {
        const match = link.textContent.toLowerCase().includes(query)
        link.parentElement.style.display = match ? '' : 'none'
      })
    })
  </script>
</body>
</html>
```

---

## Common Patterns Across All Frameworks

Regardless of framework, every template MUST include:

1. **Two-column layout** — sticky sidebar + scrollable main
2. **Sidebar filter** — text input that filters navigation items
3. **Anchor links** — every component section has an `id` for deep linking
4. **Category grouping** — sidebar items grouped by category (Foundations, Actions, Forms, etc.)
5. **Responsive collapse** — sidebar hides on mobile, content goes single-column
