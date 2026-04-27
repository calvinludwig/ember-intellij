# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

JetBrains IntelliJ Platform **theme plugin** (UI category). The deliverable is theme JSON files plus editor color scheme XMLs consumed by `com.intellij.themeProvider` and `com.intellij.bundledColorScheme` — no Kotlin/Java code. Six variants ship: `Ember`, `Ember Soft`, `Ember Light`, plus three Islands counterparts (`Islands Ember`, etc.) that extend JetBrains' Islands base theme.

## Color sourcing — required

**All colors must come from `palette.json` at the repo root. Never hand-pick hex values in theme files.**

`palette.json` is the single source of truth for the Ember identity. It defines:

- `variants.{ember,ember-soft,ember-light}` — each with a `background` / `foreground` pair and a full `ramp` block (`base0..base8`) for backgrounds, panels, borders, and dim text levels.
- `accents` (dark variants) and `accents_light` (light variant) — named accent colors (`coral`, `orange`, `gold`, `olive`, `sage`, `steel`, `rose`, `mauve`), each with a `role` field documenting its intended semantic use (e.g. `coral` = keywords/cursor/links, `olive` = strings, `gold` = functions/types). **Honor those roles** when mapping colors to syntax tokens or UI keys — do not, for example, repurpose `olive` for buttons.

Workflow when adding a color to a `*.theme.json` or `*.xml`:

1. Pick the right named accent (or variant ramp level) from `palette.json` based on the documented role.
2. For `*.theme.json`: add it to the top-level `colors` block under a meaningful name with the hex copied from `palette.json`, then reference that name from `ui.*`.
3. For `*.xml` editor schemes: inline the hex (XML format does not support named refs). Keep a header comment pointing back to `palette.json` so the link is obvious.
4. If a UI key needs a color that doesn't exist in `palette.json`, add it to `palette.json` first (with a `role`), then pull it into the theme.

The upstream references in `README.md` (https://github.com/ember-theme/ember, https://embertheme.com) are background reading; `palette.json` is what the plugin builds from. Syntax-token → accent mapping mirrors the conventions from `ember-theme/nvim`.

## Build / run

Gradle-based using the **IntelliJ Platform Gradle Plugin** (`org.jetbrains.intellij.platform`). JDK 21.

- `./gradlew runIde` — launches a sandbox IDE with the plugin installed. **This is the only reliable way to test editor color schemes and Islands variants.** The "Preview Theme" gutter button on a `*.theme.json` only loads the UI block — it can't see `plugin.xml`, so `<bundledColorScheme>` registrations and `targetUi="islands"` are bypassed.
- `./gradlew buildPlugin` — produces `build/distributions/ember-intellij-<version>.zip` for marketplace upload or local install.
- `./gradlew verifyPlugin` — runs JetBrains' Plugin Verifier; the `.run/Run Verifications.run.xml` config wraps it.
- `./gradlew prepareSandbox` — stages the plugin in a sandbox without launching the IDE.

The IDE version targeted is set in `build.gradle.kts` via `intellijIdea("...")`. Bump that to test against newer platforms. `gradle.properties` carries the plugin's own `version` and `group`; `plugin.xml` carries `<idea-version since-build>` and the change-notes.

## Layout

- `palette.json` (repo root) — canonical color palette. See **Color sourcing** above.
- `src/main/resources/META-INF/plugin.xml` — plugin descriptor. Registers six themes via `<themeProvider>` (Islands ones carry `targetUi="islands"`) and three editor schemes via `<bundledColorScheme>`. **Both extension types are required** — `editorScheme:` in a theme JSON only points; the XML itself must be registered separately or the IDE silently falls back to the default scheme.
- `src/main/resources/theme/*.theme.json` — six theme JSON files. Top-level `colors` defines a named palette; `ui.*` keys reference those names. Islands variants set `parentTheme: "Islands Dark"`/`"Islands Light"` plus `Islands: 1` and `Island.borderColor`.
- `src/main/resources/theme/*.xml` — three editor color schemes (one per variant; Islands variants reuse the matching variant's XML via `editorScheme:`). Must start with `<?xml version="1.0" encoding="UTF-8"?>` — the platform's scheme loader rejects schemes without the declaration.
- `src/main/resources/META-INF/pluginIcon.svg` — shown in the Marketplace / Plugin Manager. Currently the JetBrains scaffold icon; replace before publishing.
- `build.gradle.kts`, `settings.gradle.kts`, `gradle.properties` — Gradle config.
- `.run/` — pre-configured run configs (`Run Plugin` → `runIde`, `Run Verifications` → `verifyPlugin`).
- `assets/` — README artwork only; not bundled in the plugin.

## Conventions worth knowing

- The `themeProvider` `id` and `bundledColorScheme` `id` attributes are stable identifiers. Do **not** regenerate them — changing them invalidates the theme/scheme for any user who already selected it.
- Each variant pair (regular + Islands) shares its editor scheme XML. The Islands variant's distinct look comes entirely from the theme JSON's `ui.*` overrides (canvas/panel split, transparent toolbar/status borders, `Islands` flag).
- Islands canvas/panel pairs are picked from `palette.json` ramps with verified ≥1.20:1 WCAG contrast. If you change a variant's `bg` or panel level, recompute the contrast.
- Author attribution: `Calvin Ludwig` in both `plugin.xml` `<vendor>` and each theme JSON's `author`. Keep them consistent.
