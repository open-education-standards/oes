# Hubs

A **hub** is a set of static files, served over HTTPS, that lists OPK
packages so anyone can browse and import them. It is the one place OPK
touches the network, and it is deliberately small: a hub needs no server,
no database and no accounts. Publishing a folder of files is enough to
run one.

A consumer, such as an LMS, can connect to any number of hubs and show
their packages together. No hub is special: an official hub is just a
hub that an app happens to list by default.

[[toc]]

## Scope of this version

This version defines **public hubs only**. Anything a hub lists can be
read by anyone. Authentication will be added in a later version, and the
rules below are written so that adding it breaks nothing:

- A hub that requires authentication **MUST** respond `401` or `403`.
- A consumer that receives either **MUST** treat the hub as "requires
  access, which isn't supported", not as an invalid hub, and **MUST NOT**
  let it affect any other hub it is connected to.

## Files

```
{base}/index.json                                         the hub index
{base}/packages/{namespace}/{name}.json                   one package, every version
{base}/packages/{namespace}/{name}/{version}/package.json the version's OPK manifest
{base}/packages/{namespace}/{name}/{version}/bundle.json  the version's bundled payload
{base}/packages/{namespace}/{name}/{version}/files/…      the package's own files
```

Only `index.json` has a fixed location. Every other file is found by
following URLs, so the layout above is a recommendation, not a
requirement.

**URLs inside hub documents are relative to the document that contains
them**, unless absolute. A hub can then be copied, mirrored or moved to
another host without rewriting a single file.

## The hub index

`index.json` describes the hub and lists one summary per package.

```json
{
  "opk_version": "0.2.0",
  "kind": "hub",
  "hub": {
    "title": "Example Hub",
    "description": "Public OES content.",
    "homepage": "./"
  },
  "packages": [
    {
      "name": "acme/databases",
      "title": "Databases",
      "description": "An introductory course on relational databases.",
      "entry_type": "course",
      "tags": ["databases", "sql"],
      "latest": "2.1.0",
      "url": "packages/acme/databases.json"
    }
  ]
}
```

- A package appears in `packages` only while it has at least one version
  that is not yanked. `latest` is the highest such version.
- **`moved_to`**: when a hub relocates, it sets `moved_to` to the absolute
  `https://` URL of its new `index.json`. Static hosts cannot redirect a
  JSON file, so a consumer **MUST** follow this field and **SHOULD**
  update the hub URL it has stored.

## The hub package document

Each package has one document listing every version ever published.

```json
{
  "opk_version": "0.2.0",
  "kind": "hub-package",
  "name": "acme/databases",
  "versions": [
    {
      "version": "2.1.0",
      "digest": "sha256-…",
      "published_at": "2026-10-13",
      "manifest": "databases/2.1.0/package.json",
      "bundle": "databases/2.1.0/bundle.json",
      "bundle_sha256": "sha256-…",
      "files": "databases/2.1.0/files/"
    },
    {
      "version": "2.0.0",
      "digest": "sha256-…",
      "published_at": "2026-09-01",
      "manifest": "databases/2.0.0/package.json",
      "bundle": "databases/2.0.0/bundle.json",
      "bundle_sha256": "sha256-…",
      "files": "databases/2.0.0/files/",
      "yanked": true,
      "yanked_reason": "Answer key in question 4 was wrong; use 2.1.0."
    }
  ]
}
```

- **`files`** is the directory holding the package's own files, exactly
  as listed in the manifest's `files`, at their package-relative paths.
  Relative URLs inside the bundle resolve there, so it SHOULD be the
  `files/` directory next to the bundle.
- **`bundle_sha256`** is the SHA-256 of the bundle file's bytes, so a
  consumer can verify the exact bytes it fetched. The package `digest`
  identifies the release; `bundle_sha256` checks the download.

## Rules

- **Everything on a public hub is public.** That includes every file in a
  package, so a package whose questions keep their answers in an
  `answer_key` file publishes those answers too. Bundles still withhold
  answer keys, but graded content that must stay secret does not belong
  on a public hub.
- **Package identity is hub plus name.** Two hubs may each list an
  `acme/databases`; they are different packages. A consumer **MUST** keep
  track of which hub a package came from.
- **Published versions are immutable.** A version's `digest` **MUST**
  never change, and a hub **MUST NOT** publish different content under an
  existing version. The `digest` in the hub package document **MUST**
  equal the one in the version's manifest.
- **Nothing is deleted; versions are yanked.** A yanked version stays
  fetchable, so anything already using it keeps working, but it is hidden
  from browsing and a consumer **MUST NOT** choose it for a new import.
- **Each version appears once** in a hub package document.
- **Dependencies resolve within the same hub.** A package's
  `dependencies` are looked up on the hub the package came from.
  Dependencies across hubs are not defined in this version.
- **Open access.** A public hub **MUST** be served over HTTPS with
  `Access-Control-Allow-Origin: *`, so browser-based apps can read it.
- **Assets belong to the hub.** A bundle served by a hub **SHOULD** point
  its `assets[].url` at the package's files under the hub, so a future
  version can protect them along with everything else. Videos and
  documents stay external, as [OVF](/specs/ovf/) and [ORF](/specs/orf/)
  already require.

## Consumers

- **Verify before use.** A consumer **MUST** check a fetched bundle
  against `bundle_sha256` before using it. A consumer that fetches the
  package's files **MUST** check each against the manifest's `files` and
  the whole against `digest`.
- **Cache by digest.** Manifests and bundles are immutable, so a consumer
  may cache them indefinitely, keyed by digest. `index.json` and hub
  package documents change, so a consumer **SHOULD** revalidate them.
- **Fail one hub at a time.** A hub that is unreachable, requires access
  or serves an invalid index **MUST NOT** stop a consumer using its other
  hubs.

## What a hub leaves to itself

How packages get onto a hub (pull requests, an upload form, automatic
publishing from tagged releases), who reviews them, search beyond what
`index.json` allows, and download statistics are all a hub's own
decisions. Two hubs can make them differently and still serve the same
consumers.
