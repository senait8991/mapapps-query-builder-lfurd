# Copilot Instructions for mapapps-query-builder

## Project Overview

This repository contains the **Query Builder** bundle for [map.apps 4.x](https://www.conterra.de/en/products/map-apps.html) (con terra GmbH). It provides two OSGi/AMD bundles that enable users and administrators to create custom complex queries on spatial data stores.

- **`dn_querybuilder`** – Core bundle: UI widgets (Vue.js/Vuetify), query model, query controller, and metadata analyzer for building and executing complex queries against ArcGIS feature stores.
- **`dn_queryplaceholder`** – Companion bundle: placeholder providers (date, user, extent, app name) that can be embedded in query definitions.

**Requirement:** map.apps ≥ 4.12.0

## Tech Stack

- **Build system:** Apache Maven (`pom.xml`), with the `mapapps-maven-plugin` managing bundle packaging.
- **JavaScript bundles:** AMD modules (map.apps framework), Vue.js 2.x + Vuetify components (`.vue` files).
- **Type checking:** TypeScript typings via `@conterra/ct-mapapps-typings` (devDependency in `package.json`).
- **Linting:** ESLint (`.eslintrc`), Stylelint – configs from `eslint-config-ct-prodeng` and `stylelint-config-ct-prodeng`.
- **Testing:** Mocha + Chai (unit tests), Puppeteer (integration/UI tests) via `@conterra/mapapps-mocha-runner`.
- **CI:** GitHub Actions (`.github/workflows/`): `devnet-bundle-snapshot.yml` (push/PR to master) and `devnet-bundle-release.yml` (manual release).

## Repository Layout

```
mapapps-query-builder/
├── .github/
│   ├── copilot-instructions.md   # This file
│   └── workflows/                # GitHub Actions CI workflows
├── .vscode/tasks.json            # VS Code build tasks (Jetty server, install)
├── pom.xml                       # Maven build descriptor (mapapps-maven-plugin)
├── build.properties              # Local dev: set mapapps.remote.base here
├── package.json                  # Node devDependencies (linting, testing tools)
├── .eslintrc                     # ESLint configuration
├── tsconfig.json                 # TypeScript config (for typings/IDE support)
└── src/
    ├── main/
    │   ├── config/assembly.xml   # Maven assembly descriptor
    │   └── js/
    │       ├── apps/             # Sample map.apps applications (sample, sample_selectionactions)
    │       └── bundles/
    │           ├── dn_querybuilder/        # Core query builder bundle
    │           │   ├── manifest.json       # Bundle descriptor (components, dependencies)
    │           │   ├── module.js           # AMD module entry point
    │           │   ├── main.js             # Main entry
    │           │   ├── QueryBuilderWidget.vue
    │           │   ├── FieldWidget.vue
    │           │   ├── QueryBuilderWidgetModel.js
    │           │   ├── QueryBuilderWidgetFactory.js
    │           │   ├── EditableQueryBuilderWidgetFactory.js
    │           │   ├── QueryController.js
    │           │   ├── QueryToolController.js
    │           │   ├── MetadataAnalyzer.js
    │           │   ├── MemoryStore.js
    │           │   ├── CachingStore.js
    │           │   ├── config/             # Admin config widget (ct-mapapps Config UI)
    │           │   ├── css/styles.css
    │           │   ├── nls/                # i18n (en + de)
    │           │   └── tests/             # Mocha unit tests
    │           └── dn_queryplaceholder/   # Placeholder provider bundle
    │               ├── manifest.json
    │               ├── Replacer.js
    │               ├── DatePlaceholderProvider.js
    │               ├── AuthenticationPlaceholderProvider.js
    │               ├── AppNamePlaceholderProvider.js
    │               └── ExtentPlaceholderProvider.js
    └── test/
        ├── resources/            # Test resources
        └── webapp/               # Jetty test web application (WEB-INF/web.xml, JS test init)
```

## Build & Development Commands

### Prerequisites

- JDK 8+ and Maven 3.x installed.
- Node.js (for linting/testing tools in `package.json`).
- A running map.apps server (for development). Set the URL in `build.properties`:
  ```
  mapapps.remote.base=http://YOURSERVER/ct-mapapps-webapp-VERSION
  ```

### Build Commands

```bash
# Full build (compile + package + compress JS bundles)
mvn clean install -Pcompress

# Build with explicit remote base (no build.properties required)
mvn install -Dmapapps.remote.base=http://YOURSERVER/ct-mapapps-webapp-VERSION

# Build using build.properties file
mvn install -Denv=dev -Dlocal.configfile=/ABSOLUTE/PATH/TO/build.properties
```

### Development Server

```bash
# Start embedded Jetty server with file watch (requires mapapps.remote.base)
mvn clean jetty:run -Pwatch-all

# Stand-alone Jetty server (includes map.apps dependencies locally)
mvn clean jetty:run -P'watch-all,include-mapapps-deps'
```

### Running Tests

Tests are executed as part of the Maven build. The test runner uses Puppeteer to run Mocha tests in a headless browser against the Jetty server.

```bash
mvn test
```

### Linting (JavaScript)

```bash
# Install Node dependencies first
npm install

# Run ESLint
npx eslint src/main/js --ext .js,.vue
```

## Key Architectural Concepts

- **Bundle components** are declared in `manifest.json` using the map.apps OSGi component model. Components reference each other via `provides`/`references` declarations.
- **Vue.js widgets** are created via factory classes (`QueryBuilderWidgetFactory`, `EditableQueryBuilderWidgetFactory`) that instantiate Vue components with bound models.
- **Query model** is managed by `QueryBuilderWidgetModel.js` (reactive state) and executed by `QueryController.js`.
- **Complex Query Language** (CQL) is used for query expressions. See: https://docs.conterra.de/en/mapapps/latest/developersguide/concepts/complex-query.html
- **i18n** strings are in `nls/bundle.js` (English) and `nls/de/bundle.js` (German).

## Code Conventions

- JavaScript files use AMD `define(...)` module syntax (map.apps framework convention).
- Vue SFCs (`.vue`) use Vue 2.x options API with Vuetify components.
- License header (Apache 2.0) must be present in all source files – see `apache2-license-header.txt`.
- `indent_style = space`, `indent_size = 4`, `end_of_line = lf`, `charset = utf-8` (see `.editorconfig`).
- Do not use `var`; prefer `const`/`let`.

## Testing

Unit tests are located in `src/main/js/bundles/dn_querybuilder/tests/`. The test entry point is `all.js` which imports individual test modules. Tests use Mocha + Chai assertion style.
