---
name: drupal-canvas
description: Drupal Canvas Code Components - a framework for building interactive, server-side-rendered components in Drupal. Covers scaffolding with @drupal-canvas/create, the Nebula starter template, component architecture, and integration with Acquia Source Site Builder.
applyTo: "**/*.component.tsx,**/*.metadata.json,**/*.styles.css"
---

# Drupal Canvas Code Components

Canvas Code Components enable interactive, server-side-rendered components in Drupal
using modern JavaScript patterns with full Drupal integration.

## Quick Start

### Scaffold a New Canvas Project

```bash
npx @drupal-canvas/create my-project
```

This creates a complete Drupal project with Canvas Code Components pre-configured,
including Drupal core with Canvas module, example components, DDEV local development
setup, and the Nebula starter template.

### Install Full Canvas Skills

For comprehensive component development guidance (7 detailed skills):

```bash
npx skills add drupal-canvas/skills
```

## What Are Code Components?

Code Components are Drupal's approach to interactive, reusable UI components that are:

- **Server-side rendered** — Full SSR with hydration, not client-only
- **Drupal-integrated** — Access entities, fields, views, and Drupal APIs
- **Composable** — Nest components, pass props, emit events
- **Styled** — Scoped CSS with design token support
- **Type-safe** — Full TypeScript support with prop validation

## Component Structure

```
my-component/
├── my-component.component.tsx   # Component logic
├── my-component.metadata.json   # Drupal integration metadata
├── my-component.styles.css      # Scoped styles
└── my-component.utils.ts        # Optional utilities
```

## Example Component

```tsx
import { type ComponentProps } from '@drupal-canvas/sdk';

export default function MyComponent({ title, items }: ComponentProps) {
  return (
    <div className="my-component">
      <h2>{title}</h2>
      <ul>
        {items.map((item, i) => (
          <li key={i}>{item.label}</li>
        ))}
      </ul>
    </div>
  );
}
```

## Ecosystem

### Nebula Starter Template

The recommended starting point for new Canvas projects:
- Pre-configured Drupal with Canvas module
- Example components demonstrating all patterns
- DDEV local development setup

### Acquia Source Site Builder

Canvas components integrate with Acquia's Source Site Builder for:
- Visual component placement and configuration
- Content editor-friendly component management
- Design system integration
- Multi-site component sharing

### Full Canvas Skills Collection

```bash
npx skills add drupal-canvas/skills
```

Covers 7 topics:
1. **Component Definition** — Creating and registering components
2. **Metadata** — Drupal integration via metadata.json
3. **Composability** — Nesting, props, events, slots
4. **Styling** — Scoped CSS, design tokens, theming
5. **Utilities** — Helper functions and shared logic
6. **Data Fetching** — Entity queries, views integration, API calls
7. **Upload** — Deploying components to Drupal

## Resources

- **Scaffolding**: `npx @drupal-canvas/create`
- **Canvas Skills**: `npx skills add drupal-canvas/skills`
- **Drupal Canvas Module**: Available via Composer
