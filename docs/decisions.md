# Design decisions

The main decisions behind OES, with the reasoning for each. The specs on
the [docs site](https://open-education-standards.github.io/oes/) are the
normative text; this page explains why they look the way they do.

## Shape of the format

**Static files, git-native, auth-agnostic.** OES content is plain JSON and
Markdown files. No database or server is needed to author or host it, it
can live in a git repository and be reviewed in a pull request, and who
may read it is left entirely to whoever hosts it.

**JSON is chosen for people as well as machines.** Documents are
pretty-printed with long, readable field names so that a one-word change
shows up clearly in a diff. JSON is the format that serves both a human
reviewer and a program adequately.

**The specs define only the JSON form.** There are three forms of OES
content, and the specification is the middle one:

```
authoring formats  →   question.json, course.json…   →   bundled payload
(optional tooling)     (the specification)               (the network form)
```

Authoring conveniences belong to tooling and are never part of the spec.
The bundled payload is specified (in OPK) because a registry serving
content to an independent application is an interop boundary.

**Compose by reference.** A course does not contain its lessons; it points
at them. Every reference is either a co-located `path` or an external
`*_url`, optionally with a `content_hash` so a consumer can verify it got
the bytes the author meant. Content can therefore be split, reused and
hosted in different places.

**Seven specs, each for a thing that is shared on its own.** A new spec is
justified only when it describes something people hand to each other
independently: a course (OCF), a practice set (OPF), a question (OQF), an
article (OAF), a video lesson (OVF), a reference document (ORF), and a
package (OPK). Anything that fails that test belongs as fields on an
existing spec or as an extension.

## Scope

**Content, not runtime state.** Progress, attempts, scores, enrolment and
grading policy are permanently out of scope. Fields such as
`time_limit_mins` and `max_attempts` record the author's intent, which a
platform may enforce; OES itself enforces nothing at runtime.

**Self-practice and graded use are both first-class.** By default a
question carries its answer inline, which is what self-practice wants. A
question meant for graded use moves its answer to a separate `answer_key`
file or URL, and the schema then forbids the answer from also appearing
inline, so the two can never disagree and a learner's client never needs
the answer.

**Provenance is recorded, platform-neutrally.** Any document can carry a
`source` describing where it was adapted from and under what original
licence. The platform name is free text, never an enum: OES does not
endorse or rank any source.

**OPK leaves registry policy to registries.** OPK defines a package's
name, version, manifest, digest and bundled payload. Authentication,
search, ranking, quotas and deprecation policy are a registry's own
decisions and are deliberately absent.

**Hubs are static files, public for now.** A hub is an `index.json` plus
one document per package, served over HTTPS, so anyone can run one by
publishing a folder: no server, database or accounts. Packages are
identified by hub plus name, published versions never change, and bad
versions are yanked rather than deleted. Authentication is left for a
later version; a hub that needs it answers `401`/`403` today, so adding
it breaks no existing consumer.

## Versioning and compatibility

**One version per spec, plus an OES release.** Each document carries its
own spec's version (`oqf_version`, `ocf_version`…), which changes only when
that spec's shape changes, so unrelated content never needs re-stamping.
The OES release version names a tested combination of all seven, with a
compatibility table on the Versioning & Conformance page.

**0.x versions may break, deliberately.** While every spec is pre-1.0,
breaking changes are made when they produce a clearly better format, and
they are batched into explicit version bumps with conformance fixtures.

**Conformance is a test suite, not prose.** `conformance/` holds valid and
invalid fixtures for every spec, run in CI. Conformance profiles (Core,
+Media, +Practice, +Graded, plus separately declared `code` and
`submission` support) are independent capabilities rather than a ladder,
because an implementation can need one without the others.

## Extensibility

**`x_` fields for everything the spec doesn't cover.** Any field starting
with `x_` is reserved for extensions and ignored by compliant parsers.
Extensions that prove useful to more than one implementation are
candidates for the spec itself; the shared registry in `extensions/`
avoids two vendors inventing the same field twice.

## Schemas

**JSON Schema draft-07.** Widely supported by validators and editors.

**Shared definitions are copied, not `$ref`'d across files.** A few
definitions (`source`, `reference`, `related_item`) appear in several
specs. They are duplicated into each schema so every schema file stands
alone, and `npm run check-shared-definitions` fails CI if the copies ever
drift apart.

**The site must point at current versions.**
`npm run check-doc-schema-versions` fails CI if a page references an
outdated schema, or if a spec's version badge or download link isn't the
current version, except where an old version is shown on purpose.

**Editor support without a global mapping.** Documents may carry a
`$schema` hint, and the docs give a workspace `.vscode/settings.json`.
OES filenames (`course.json`, `lesson.json`…) are too generic for a
global, editor-wide schema association to be safe.

## Licensing and repositories

**Specs under CC BY 4.0.** Attribution means every derivative or fork of
the specs has to say it is based on OES, which keeps forks traceable to
the original. The small amount of code here is Apache 2.0.

**Quality is protected by conformance, not by licence.** A licence cannot
require an implementation to be good. The conformance suite, and later a
documented compatibility policy, is the mechanism for that.

**The reference tooling lives in a separate repo.**
[`oes-tooling`](https://github.com/open-education-standards/oes-tooling)
holds the npm packages and their publish credentials, so this repo can stay
easy to contribute to without any secrets in it. The tooling reads the
schemas from here; nothing in this repo needs to know the tooling exists.
