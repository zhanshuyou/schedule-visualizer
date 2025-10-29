# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Schedule Visualizer** - a visual editor for creating and managing Cron and RRule expressions with a Chinese language interface. Built with SvelteKit 2 and Svelte 5, using Tailwind CSS 4 for styling.

## Development Commands

```bash
# Development
npm run dev              # Start dev server
npm run dev -- --open    # Start dev server and open in browser

# Building
npm run build            # Build for production
npm run preview          # Preview production build

# Code Quality
npm run check            # Type-check with svelte-check
npm run check:watch      # Type-check in watch mode
npm run lint             # Run ESLint and Prettier checks
npm run format           # Format all files with Prettier

# Testing
npm run test             # Run all tests (e2e + unit)
npm run test:unit        # Run unit tests with Vitest
npm run test:unit -- --run  # Run unit tests once (no watch)
npm run test:e2e         # Run Playwright e2e tests
```

## Architecture

### Tech Stack
- **Framework**: SvelteKit 2 with Svelte 5 (using new runes API)
- **Styling**: Tailwind CSS 4 with typography plugin
- **Testing**: Vitest for unit tests (browser mode with Playwright), Playwright for e2e tests
- **Deployment**: Vercel adapter configured
- **Language**: TypeScript with strict mode enabled

### Project Structure
```
src/
├── routes/
│   ├── +layout.svelte       # Root layout with app.css and favicon
│   └── +page.svelte          # Main page that renders ScheduleVisualizer
├── components/
│   └── schedule-visualizer.svelte  # Main component with dual-mode editor
├── lib/
│   ├── index.ts              # Library exports
│   └── assets/               # Static assets like favicon
e2e/                          # Playwright e2e tests
```

### Main Component: `schedule-visualizer.svelte`

This is the core component (src/components/schedule-visualizer.svelte:1-619) containing all the application logic:

**Dual Mode Architecture**:
- **Cron Mode**: Generates standard cron expressions (minute/hour/day/month/weekday)
- **RRule Mode**: Generates RFC 5545 recurrence rules (FREQ/INTERVAL/COUNT/UNTIL/BYDAY/BYMONTHDAY)

**Key Features**:
- Real-time expression generation using Svelte 5 reactive statements (`$:`)
- Human-readable Chinese descriptions for both modes
- Quick presets for common cron patterns (midnight, weekday 9am, every 5min, every 4hrs)
- Copy-to-clipboard functionality with visual feedback
- Weekday selector with multi-select for RRule mode
- Comprehensive input validation hints

**State Management**:
Uses Svelte 5 runes (`$props`, reactive `$:` statements) for state management:
- Cron state: `cronMinute`, `cronHour`, `cronDay`, `cronMonth`, `cronWeekday`
- RRule state: `rruleFreq`, `rruleInterval`, `rruleCount`, `rruleUntil`, `rruleByDay`, `rruleByMonthDay`

### Testing Setup

**Vitest Configuration** (vite.config.ts:7-35):
- Dual project setup: one for client-side tests, one for server-side
- Client tests run in browser mode with Playwright/Chromium
- Client tests match pattern: `src/**/*.svelte.{test,spec}.{js,ts}`
- Server tests match pattern: `src/**/*.{test,spec}.{js,ts}` (excluding svelte tests)
- Requires assertions enabled: `expect.requireAssertions: true`

**Playwright Configuration** (playwright.config.ts:1-9):
- E2e tests located in `e2e/` directory
- Runs against production build (`npm run build && npm run preview`)
- Tests run on port 4173

### TypeScript Configuration

Strict mode enabled with:
- `allowJs: true` and `checkJs: true` for JS file validation
- `esModuleInterop: true` for module compatibility
- `forceConsistentCasingInFileNames: true` for cross-platform consistency
- Path aliases handled by SvelteKit (e.g., `$lib` for src/lib)

## Key Considerations

### Svelte 5 Features
This project uses Svelte 5 with modern features:
- Runes API (`$props`, `$state`, `$derived`)
- Reactive statements with `$:` syntax
- `{@render}` for slot rendering in layouts
- When using the Svelte MCP server, always specify `desired_svelte_version: 5`

### Styling Approach
- Tailwind CSS 4 is configured via Vite plugin (no tailwind.config.js needed)
- Uses utility classes with arbitrary values
- Custom fonts loaded via Google Fonts (Inter)
- Responsive design with `md:` breakpoints
- Component uses both Tailwind utilities and scoped `<style>` tags

### Chinese Localization
All UI text is in Chinese. When modifying:
- Maintain Chinese labels and descriptions
- Keep Chinese weekday/month name mappings
- Preserve Chinese help text and tooltips
