# Espressif Skills for AI

This folder contains agent skills and instructions for working with Espressif projects. Skills provide structured workflows, best practices, and reference material that coding agents use when completing tasks in this repository.

## Skills

### ESP-IDF

| Skill | Description | Path |
|---|---|---|
| `esp-idf` | ESP32 firmware development: build/flash/monitor workflows, EIM CLI setup, ESP-IDF code conventions, and Component Registry API usage. | `skills/esp-idf/esp-idf/` |
| `esp-idf-components` | Guide creation of ESP-IDF components inside a project or as reusable component-manager packages, including create-component scaffolding, dependency management, structure, Kconfig usage, and publication decisions. | `skills/esp-idf/esp-idf-components/` |
| `esp-idf-v6-migration` | Migrate existing ESP-IDF application repositories from ESP-IDF 5.x to 6.0 using cumulative migration guides, version detection, build failure triage, and subsystem-specific fixes. | `skills/esp-idf/esp-idf-v6-migration/` |

### Espressif Developer Portal

| Skill | Description | Path |
|---|---|---|
| `espressif-portal-articles` | Writing articles for the Espressif Developer Portal, including content creation, editing, and publishing. | `skills/espressif-developer-portal/espressif-portal-articles/` |
| `espressif-portal-hugo` | Working with Hugo static site generator for the Espressif Developer Portal, including content structure, theme customization, and site deployment. | `skills/espressif-developer-portal/espressif-portal-hugo/` |

## Agent Instructions

The `skills/esp-idf/AGENTS.md` file contains repository-specific coding agent rules for ESP-IDF projects, covering environment setup, build/flash workflows, component management, Kconfig conventions, and safety guidelines.

## Templates

The `templates/` folder contains scaffolding templates for new components and references used by the skills above.

## How to install

```bash
npx skills add <SKILL NAME>
```

## How to update

```bash
npx skills update <SKILL NAME>
```

## Installed skills

```bash
npx skills list
```
