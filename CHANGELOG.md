# Changelog

All notable changes to this package are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), versioning follows
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.1] - 2026-08-17

### Fixed

- **Backend labels stayed untranslated.** The inspector showed raw translation
  identifiers such as `Waterproof.News:NodeTypes.Document.Article:properties.publicationDate`
  instead of the field label. Two independent causes:
  - `Settings.yaml` declared only `Neos.Neos.fusion.autoInclude`. Without
    `Neos.Neos.userInterface.translation.autoInclude` the XLIFF catalogues are
    never shipped to the backend JavaScript, so no property label could resolve.
  - Inspector group labels used the identifier `ui.inspector.groups.<name>`.
    Neos generates `groups.<name>` (see `NodeTypeEnrichmentService::getInspectorElementTranslationId()`),
    so every group heading fell back to its raw key.

  Both are corrected for German and English. No configuration change is required
  in projects using the package; a cache flush after the update is enough.

## [1.0.0] - 2026-08-14

### Added

- Initial release: article, article index, article teaser, article list and feed
  node types for Neos CMS 9, built on Flowpack.Listable and Sitegeist.Taxonomy.
- Category and year filters on the index, combined and surviving pagination.
- Atom and RSS 2.0 feed with `<link rel="alternate">` on the index.
- JSON-LD `Article` structured data on article pages.
- Backend labels as XLIFF for German and English.

[1.0.1]: https://github.com/dhuettner/neos-news/compare/v1.0.0...v1.0.1
[1.0.0]: https://github.com/dhuettner/neos-news/releases/tag/v1.0.0
