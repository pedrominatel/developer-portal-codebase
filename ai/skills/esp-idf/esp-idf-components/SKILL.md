---
name: esp-idf-components
description: Guide creation of ESP-IDF components inside a project or as reusable component-manager packages, including create-component scaffolding, dependency management, structure, Kconfig usage, and publication decisions.
compatibility: Works with standard ESP-IDF command-line tools.
---

# ESP-IDF Components

Use this skill when creating new ESP-IDF components either inside an existing project or as standalone reusable components.

## When to Use

- Creating a new component under `components/` inside an ESP-IDF project
- Creating a standalone ESP-IDF component intended for reuse across projects
- Bootstrapping a reusable component with `idf.py create-component <component-name>`
- Deciding whether to use a registry dependency or write a local component
- Adding `idf_component_register(...)` entries and dependency declarations
- Defining public headers, private implementation files, and component configuration
- Preparing a component to be reused across projects or published externally
- Reviewing the expected layout, manifest, examples, and CI expectations for component-manager packages

## Workflow

### 1. Decide whether a new component is needed
- Search the ESP Component Registry first using the public registry API or website.
- If an existing registry component fits, prefer adding it as a dependency instead of creating a duplicate local or standalone component.
- If the code is project-specific, tightly coupled to app behavior, or unlikely to be reused, create it inside the project under `components/`.
- If the code is generic and likely to be shared across applications, create it as a standalone component with its own metadata and documentation.
- If more than one registry component looks plausible, present the options and let the user choose.

### 2. Choose the component mode
- **Project-local component:** use this when the component belongs to one application and should live under `components/<component_name>/`.
- **Standalone component-manager package:** use this when the component should be reusable across projects, versioned independently, or prepared for publication through the ESP Component Registry.
- If the user has not decided, recommend the mode based on expected reuse and coupling to the application.

### 3. Check the project manifest first
- Before creating a new project-local component, check whether the dependency should be added in a manifest such as `main/idf_component.yml`.
- For registry dependencies, prefer:
  - `idf.py add-dependency "namespace/component^version"`
- Use `idf.py add-dependency ...` when you are consuming an existing component.
- Use `idf.py create-component <component-name>` when you are bootstrapping a new reusable component that you will own.
- Keep versions explicit when reproducibility matters.

### 4. Bootstrap with the right workflow
- For a simple project-local component, create `components/<component_name>/` manually with only the files the project needs.
- For a reusable component-manager package, prefer `idf.py create-component <component-name>` and then add packaging files such as `idf_component.yml`, `README.md`, and `LICENSE`.
- Keep the template-generated structure minimal first; add examples, CI, and extra metadata only when the component is intended for reuse or publication.
- For concrete commands, scaffold layout, manifest notes, and CI expectations, read [Component Manager Reference](#component-manager-reference).

### 5. Use the right structure
- For a project-local component, create it under `components/<component_name>/`.
- For a standalone component, use the component root as the working root.
- Project-local layout:
  - `components/<component_name>/CMakeLists.txt`
  - `components/<component_name>/include/<component_name>.h`
  - `components/<component_name>/<component_name>.c`
- Standalone layout:
  - `CMakeLists.txt`
  - `include/<component_name>.h`
  - `<component_name>.c`
- Standalone components usually also need:
  - `idf_component.yml`
  - `README.md`
  - `LICENSE`
- Add more source files only when the implementation justifies the split.
- Keep public headers in `include/`. Keep private headers next to the private source files unless they must be shared across component source files.

### 6. Register the component correctly
- Use `idf_component_register(...)` in the component `CMakeLists.txt`.
- Declare `SRCS` and `INCLUDE_DIRS` explicitly.
- Use `REQUIRES` for public dependencies exposed through the component's public headers.
- Use `PRIV_REQUIRES` for implementation-only dependencies.
- Do not rely on incidental transitive dependencies.

Example:

```cmake
idf_component_register(
    SRCS "my_component.c"
    INCLUDE_DIRS "include"
    REQUIRES esp_timer
    PRIV_REQUIRES driver
)
```

### 7. Define clean component boundaries
- Keep the public API small and stable.
- Avoid leaking unrelated ESP-IDF headers through public headers unless they are part of the intended API surface.
- Prefer opaque handles, explicit init/deinit functions, and `esp_err_t` return values for non-trivial components.
- Split unrelated responsibilities into separate components instead of building one large utility component.

### 8. Use configuration files deliberately
- Use `Kconfig` for component-local options.
- Use `Kconfig.projbuild` only when the option must appear at project scope in `menuconfig`.
- Namescape configuration symbols, for example `MY_COMPONENT_*`.
- Every new config option should include:
  - clear prompt text
  - sensible default
  - brief help text
- After changing `Kconfig` or `Kconfig.projbuild`, run `idf.py reconfigure` before building.

### 9. Decide whether the component should stay local or be reusable
- Keep it local when it is specific to one product, board, or application.
- Make it reusable when it implements a generic driver, protocol adapter, algorithm, or helper that can be shared across projects.
- Reusable components should be self-contained and should not depend on application-specific globals, secrets, or hardcoded board choices.

## Reusable Component Checklist

Use this when a component is intended for reuse or publication.

- Add `idf_component.yml` if the component will be consumed through the component manager.
- Add a `README.md` that explains purpose, configuration, dependencies, and usage.
- Add a `LICENSE` file if the component will be shared outside the project.
- Include at least one example or test app when the behavior is non-trivial.
- Add CI that at minimum builds supported targets and validates example or test apps when they exist.
- Keep Kconfig options and defaults minimal and documented.
- Verify the component builds cleanly for its supported targets.
- Use the registry upload flow only when the component is actually intended to be published.
- Require a new `idf_component.yml` `version` before CI publishes an updated registry release.

## Registry Search Notes

- Search endpoint:
  - `GET /api/components?q={query}&page=1&per_page=10`
- Component details:
  - `GET /api/components/{namespace}/{name}`
- Example README pattern:
  - `https://components-file.espressif.com/components/{namespace}/{name}/{version}/examples/{example_path}/readme.md`

## Avoid

- Creating a new local component before checking for an existing dependency
- Using the reusable component template for a one-off project-local helper under `components/`
- Putting all app code into `main/` when it has clear component boundaries
- Exposing private implementation details in public headers
- Using `REQUIRES` when `PRIV_REQUIRES` is sufficient
- Editing generated files under `build/`
- Adding broad config churn to `sdkconfig` when defaults belong in `sdkconfig.defaults*`

## Component Manager Reference

Use this section when the task involves reusable components, registry packaging, or the ESP-IDF component-manager CLI workflow.

Official docs:
- https://docs.espressif.com/projects/idf-component-manager/en/latest/index.html
- https://docs.espressif.com/projects/idf-component-manager/en/latest/use/how_to_add_dependency.html
- https://docs.espressif.com/projects/idf-component-manager/en/latest/use/how_to_update_dependencies.html
- https://docs.espressif.com/projects/idf-component-manager/en/latest/publish/tutorial_to_package_and_upload.html
- https://docs.espressif.com/projects/idf-component-manager/en/latest/reference/manifest_file.html

### CLI Workflow

- Create a new reusable component scaffold:
  - `idf.py create-component <component-name>`
- Add an existing registry component to a project or component manifest:
  - `idf.py add-dependency "namespace/component^version"`
- Add a dependency from a non-default registry when needed:
  - `idf.py add-dependency --registry-url https://components-staging.espressif.com "namespace/component^version"`
- Add a dependency from Git when the component is not consumed from the registry:
  - `idf.py add-dependency <component-name> --git https://github.com/<org>/<repo>.git --git-path components/<component-name> --git-ref <ref>`
- Update the resolved dependency set and refresh `dependencies.lock` when constraints changed:
  - `idf.py update-dependencies`
- Create a starter manifest if a component is missing `idf_component.yml`:
  - `compote manifest create`

Prefer `idf.py create-component` when starting a new reusable component you own. Prefer `idf.py add-dependency` when consuming an existing component instead of duplicating it locally.

### Scaffolded Layout

`idf.py create-component <component-name>` creates the minimal local layout:

```text
<component-name>/
├── CMakeLists.txt
├── include/
│   └── <component-name>.h
└── <component-name>.c
```

This minimal layout is enough for local use inside an ESP-IDF project.

For a reusable or publishable component-manager package, add:

```text
<component-name>/
├── CMakeLists.txt
├── idf_component.yml
├── include/
│   └── <component-name>.h
├── LICENSE
├── README.md
└── <component-name>.c
```

Notes:
- `idf_component.yml` is required for registry uploads.
- `version` is the only required manifest field, but `description`, `url`, `repository`, and `license` are strongly recommended.
- Either the manifest `license` field or a `LICENSE` or `LICENSE.txt` file must be present for registry uploads.
- Examples placed under `examples/` are auto-discovered by the registry; use the manifest `examples` field only when examples live elsewhere.
- The manifest `files` field controls upload packaging and Git dependency contents. It is optional unless the default file filtering is not sufficient.

### CI Expectations

For simple project-local components under `components/`, CI can stay at the application level. Do not force standalone component packaging or publish automation for one-off internal helpers.

For reusable components:
- Build the component in at least one example or test app against the supported ESP-IDF target set.
- If the component ships examples, make CI build those examples.
- If the component has non-trivial behavior, include at least one example or test app so CI validates integration, not only syntax.
- Keep packaging metadata valid in CI by checking that `idf_component.yml`, `README.md`, and licensing are present and consistent before publishing.
- Treat registry upload automation as optional until the component is actually intended to be published.

For publishable components:
- Prefer GitHub Actions for automated uploads when the source is hosted on GitHub.
- Use staging uploads first when validating packaging, permissions, or namespace setup.
- Increment the `version` field in `idf_component.yml` before pushing a releasable change; CI can only publish a new registry release when the manifest version is new.
- Publish to production only after the component metadata, version, and examples are ready.
- Keep secrets such as registry tokens out of workflow YAML and store them in CI secrets.

### When Not to Use the Template

Do not default to the reusable component-manager scaffold when:
- the code is tightly coupled to a single application
- the component will only live under `components/` in one project
- there is no realistic reuse, versioning, or publication need

In those cases, create a minimal local component manually and keep the structure as small as the project needs.
