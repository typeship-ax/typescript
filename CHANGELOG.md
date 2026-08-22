# Changelog

## 0.3.0 (2026-08-22) (9 breaking)

### Changed
- `generate.run()`
  - **breaking** body-field-added: outputs: OutputId[] (required)
  - **breaking** body-field-removed: platforms
  - **breaking** body-field-removed: language
- `projects.create()`
  - **breaking** body-field-added: outputs: OutputId[] (required)
  - body-field-added: packages: Packages
  - **breaking** body-field-removed: platforms
  - **breaking** body-field-removed: languages
  - **breaking** body-field-removed: destinations
  - **breaking** body-field-removed: package_names
  - **breaking** body-field-removed: destination
  - **breaking** body-field-removed: package_name
  - documentation-changed: summary or description changed
- `projects.update()`
  - body-field-added: outputs: OutputId[]
  - body-field-added: packages: Packages
  - **breaking** body-field-removed: platforms
  - **breaking** body-field-removed: destination
  - **breaking** body-field-removed: languages
  - **breaking** body-field-removed: destinations
  - **breaking** body-field-removed: package_names
  - **breaking** body-field-removed: package_name
- `projects.generate()`
  - documentation-changed: summary or description changed

### Package contract (breaking)
- **Breaking:** CLI platform removed
- **Breaking:** MCP platform removed
- **Breaking:** MCP registry name removed (was `dev.typeship/typeship`)
- **Breaking:** Executable `typeship` removed (was `dist/cli.js`)
- **Breaking:** Executable `typeship-ax` removed (was `dist/cli.js`)
- **Breaking:** Executable `typeship-mcp` removed (was `dist/mcp.js`)
