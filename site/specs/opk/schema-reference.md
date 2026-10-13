# Schema Reference

Machine-readable JSON Schema (draft-07):

- `package.json` → [`schemas/opk/v0.2.0/package.schema.json`](/schemas/opk/v0.2.0/package.schema.json)
- A hub's `index.json` → [`schemas/opk/v0.2.0/hub.schema.json`](/schemas/opk/v0.2.0/hub.schema.json)
- A hub package document → [`schemas/opk/v0.2.0/hub-package.schema.json`](/schemas/opk/v0.2.0/hub-package.schema.json)

## `package.json`

| Field | Type | Required | Description |
|---|---|---|---|
| `$schema` | string | no | Optional editor hint (e.g. VS Code) pointing at this document's JSON Schema for inline validation/autocomplete. Not part of the OES content model. |
| `opk_version` | string | **yes** | Any `0.2.x` — see [Versioning & Conformance](/conformance#versioning-policy). |
| `name` | string | **yes** | `{namespace}/{package}`, both kebab-case — e.g. `"acme/databases"`. The namespace is what a registry grants ownership of; OPK takes no position on how. |
| `version` | string | **yes** | [SemVer](https://semver.org/) for the package's *content*, independent of every spec version inside it. See [Versioning a package](./file-structure#versioning-a-package). |
| `entry` | object | **yes** | The document a consumer opens first: `{ "type": "course" \| "set" \| "article" \| "video" \| "resource", "path": "course.json" }`. Named explicitly so no consumer has to guess or probe. |
| `title` | string | **yes** | Human-readable name, for a listing. |
| `description` | string | no | One or two sentences, for a listing. |
| `authors` | string[] | no | GitHub usernames. |
| `license` | string | no | SPDX identifier. Strongly recommended on anything published — content with no stated license is content nobody can safely reuse. |
| `tags` | string[] | no | Free-form, for discovery. |
| `language` | string | no | BCP 47 tag for the content's primary language. |
| `oes_version` | string | no | The [OES release](/conformance#the-oes-release-version) this package was authored against, e.g. `"0.3.0"`. Lets a consumer check compatibility once instead of inspecting every document's own version field. |
| `requires` | object | no | Profiles a consumer must implement to render this package — `{ "profiles": ["Core", "+Practice"] }`. See [Conformance profiles](/conformance#conformance-profiles). A consumer that doesn't implement them SHOULD refuse rather than render partially. |
| `dependencies` | object | no | `{ "{namespace}/{package}": "{semver range}" }` — other packages this one needs. A release range, not a location. |
| `digest` | string | no | `sha256-{hex}` over the package's contents. See [Integrity](./file-structure#integrity). Absent while authoring; a registry SHOULD require it on publish. |
| `files` | object[] | no | The manifest: `{path, sha256, bytes}` per file. Absent while authoring; a registry SHOULD require it on publish, since it is what makes `digest` reproducible and lets a consumer verify a partial fetch. |
| `source` | object | no | Where this package was originally sourced or adapted from — see [Content provenance](/conformance#content-provenance-for-imported-adapted-content). |

## Example

```json
{
  "$schema": "https://open-education-standards.github.io/oes/schemas/opk/v0.2.0/package.schema.json",
  "opk_version": "0.2.0",
  "name": "acme/databases",
  "version": "2.1.0",
  "title": "Databases: From Tables to Transactions",
  "description": "A one-semester introduction to relational databases.",
  "authors": ["example-author"],
  "license": "CC-BY-4.0",
  "language": "en",
  "tags": ["databases", "sql"],
  "oes_version": "0.3.0",
  "entry": { "type": "course", "path": "course.json" },
  "requires": { "profiles": ["Core", "+Practice"] },
  "dependencies": { "acme/sql-basics": "^1.2.0" }
}
```

`digest` and `files` are omitted above because they are added at publish
time by whatever packages the directory, not typed by an author.

## A hub's `index.json`

See [Hubs](./hubs) for the rules these fields carry.

| Field | Type | Required | Description |
|---|---|---|---|
| `$schema` | string | no | Optional editor hint. |
| `opk_version` | string | **yes** | Any `0.2.x`. |
| `kind` | string | **yes** | Always `"hub"`. |
| `hub` | object | **yes** | `{ title, description?, homepage? }`, for display. `homepage` is a URL relative to `index.json`, or absolute. |
| `moved_to` | string | no | Absolute `https://` URL of the hub's new `index.json`, set when the hub relocates. A consumer MUST follow it. |
| `packages` | object[] | **yes** | One summary per package with at least one version that isn't yanked. |

Each entry in `packages`:

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | **yes** | `{namespace}/{package}`, as in `package.json`. |
| `title` | string | **yes** | For a listing. |
| `description` | string | no | For a listing. |
| `entry_type` | string | no | The entry document's type: `course`, `set`, `article`, `video` or `resource`. |
| `tags` | string[] | no | For search. |
| `latest` | string | **yes** | The highest version that isn't yanked. |
| `url` | string | **yes** | The package's hub package document, relative to `index.json` or absolute. |

## A hub package document

| Field | Type | Required | Description |
|---|---|---|---|
| `$schema` | string | no | Optional editor hint. |
| `opk_version` | string | **yes** | Any `0.2.x`. |
| `kind` | string | **yes** | Always `"hub-package"`. |
| `name` | string | **yes** | `{namespace}/{package}`. |
| `versions` | object[] | **yes** | Every version ever published, yanked ones included, each once. At least one. |

Each entry in `versions`:

| Field | Type | Required | Description |
|---|---|---|---|
| `version` | string | **yes** | SemVer, as in the version's manifest. |
| `digest` | string | **yes** | `sha256-{hex}`, equal to the manifest's `digest`. Never changes once published. |
| `published_at` | string | no | `YYYY-MM-DD`. |
| `manifest` | string | **yes** | The version's `package.json`, relative to this document or absolute. |
| `bundle` | string | **yes** | The version's [bundled payload](./bundled-payload), relative to this document or absolute. |
| `yanked` | boolean | no | Hidden from browsing and never chosen for a new import, but still fetchable. Default `false`. |
| `yanked_reason` | string | no | Why, for display. |

## Validating

```bash
ajv validate -s schemas/opk/v0.2.0/package.schema.json -d package.json
ajv validate -s schemas/opk/v0.2.0/hub.schema.json -d index.json
ajv validate -s schemas/opk/v0.2.0/hub-package.schema.json -d "packages/*/*.json"
```

## Extension fields

Any field matching `^x_[a-z0-9_]+$` is permitted and is intentionally left
unvalidated by the core schema. See [Extensions](/specs/ocf/extensions)
for the shared convention.
