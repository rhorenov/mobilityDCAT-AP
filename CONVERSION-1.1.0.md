# Conversion of release 1.1.0 to the build system

This is a record of how the 1.1.0 sources on `release/1.1.0` were converted to
the build and publishing tooling of `main`: what was taken from where, what was
changed by hand, and how the result was checked. It is kept so that the
conversion can be reviewed and retraced later. How to build and publish is in
`DEVELOPMENT.md`.

## Starting point

| Ref | Commit | Role |
|-----|--------|------|
| `release/1.1.0` | `fc6cb37` | Base of the conversion: the repository as it was when 1.1.0 was the current release, identical to `gh-pages` at that commit |
| `main` | `d79e21d` | Source of the build tooling and workflows |
| `gh-pages` | `2886909` | Published state the result was compared against |
| `mobilityDCAT-AP/respec-style` | `7d3eef5` (last change to `respec-napcore.js`, 2023-07-14) | Source of the NAPCORE ReSpec profile |

## Which source was used

Before the conversion, `release/1.1.0` held four copies of 1.1.0 that had
drifted apart:

| Copy | What it was | Used as |
|------|-------------|---------|
| `drafts/1.1.0-draft-0.1/` | The ReSpec source (`index.html`, `config.js`, `tables/`), with later edits that were never published (e.g. `e0cd0c9`, 2026-03-11) | **The source**, moved to `src/` |
| `releases/index.html` and the root of `releases/` | The rendered page, edited by hand after rendering; the most up-to-date published copy; target of w3id.org before `releases/latest/` | Reference: its hand edits were ported to the source; its extra assets were copied |
| `releases/1.1.0/` | An older rendered copy, without the later hand edits; its `figures/` directory is missing, so its UML diagrams do not load | Reference only |
| `drafts/latest/` | The 3.0.0 draft, moved to `src/` on `main` | Not used |

The Turtle ontology was the same in all three 1.1.0 copies. The SHACL shapes
were the same apart from line endings (CRLF in the draft, LF in the published
copies). The examples were the same files.

## Moved and renamed

| From | To |
|------|----|
| `drafts/1.1.0-draft-0.1/` | `src/` |
| `drafts/1.1.0-draft-0.1/mobilitydcat-ap_v1.1.0.ttl` | `src/mobilitydcat-ap.ttl` |

`src/shaclShapes/` and `src/enterpriseArchitectFiles/` keep their names, which
are part of the published URLs under `releases/1.1.0/`. (On `main` the SHACL
directory is `src/shacl/`.)

## Removed

| File | Why |
|------|-----|
| `src/mobilitydcat-ap_v1.1.0.rdf`, `src/mobilitydcat-ap_v1.1.0.jsonld` | Generated from the Turtle by the build. The committed files were not equivalent to the Turtle: about 35 triples differ, mostly OWL declarations from a Protégé export. |
| `src/scripts/*.sh`, `src/scripts/*.php` | Old build scripts, replaced by the Python build from `main` |
| `src/README.md` | Pointed at `drafts/latest/`, which is the 3.0.0 draft |

## Copied from the root of `releases/`

`releases/latest/` on `gh-pages` is a copy of the root of `releases/`.
Publishing empties the target directory first, so these files had to be in the
source or the first publish would have removed them.

| From | To |
|------|----|
| `releases/figures/Metadata overview_wikipage_240328.png` | `src/figures/` |
| `releases/figures/Properties_vs_GUI_elements_Wikipage_240130.png` | `src/figures/` |
| `releases/figures/mobilityDCAT-AP_minimum-profile.png` | `src/figures/` |
| `releases/enterpriseArchitectFiles/mobilityDCAT-AP-model-documentation.html` | `src/enterpriseArchitectFiles/` |
| `releases/enterpriseArchitectFiles/readme.md` | `src/enterpriseArchitectFiles/` (replaces the older readme; adds the link to the documentation page, from `d9b51c4`) |

`mobilityDCAT-AP.qea` was already the same file as the root copy.
`releases/1.1.0/enterpriseArchitectFiles/mobilityDCAT-AP.qea` differs from both
by one byte and was not used.

## Copied from `main`

Unchanged copies from `main` at `d79e21d`:

- `package.json`, `package-lock.json` (respec 36.0.0, html-validate 10.13.1)
- `.python-version`, `.gitattributes`, `.htmlvalidate.json`, `.gitignore`
- `src/scripts/check-refs.py`, `clean.py`, `mark-draft.py`, `serve.py`
- `.github/workflows/build-check.yml`, `build-release.yml`,
  `reusable-publish-gh-pages.yml`

Copied and adapted:

| File | Change from `main` |
|------|--------------------|
| `pyproject.toml`, `uv.lock` | `pyshacl` removed (only `validate-examples.py` used it); `uv.lock` regenerated with `uv lock` |
| `.mise.toml` | `validate-examples` task removed; `serialise` description updated |
| `src/scripts/serialise.py` | Serialises only the ontology. The examples (Turtle and RDF/XML, both maintained by hand) and the SHACL shapes (Turtle only) are published as they are. |
| `src/scripts/copy-assets.py` | Copies `mobilitydcat-ap.ttl`, `examples/`, `figures/`, `shaclShapes/`, `enterpriseArchitectFiles/` |
| `src/scripts/build-spec.py` | `--use-local` dropped: it only replaces a ReSpec loaded from w3.org, and this page loads the NAPCORE profile from `src/js/`. `--haltonerror` added when the `CI` environment variable is set, so a failed reference lookup fails the build on GitHub |
| `.github/workflows/reusable-build.yml` | SHACL imports cache steps removed (only `validate-examples.py` used them) |

Not copied: `validate-examples.py` (written for the 3.0.0 shapes, which import
DCAT-AP 3.0.1; the 1.1.0 shapes import DCAT-AP 2.0.1); `build-main.yml`,
`build-draft.yml`, `promote-latest.yml` (their triggers never fire from a
release branch); the documents `REPOSITORY.md` and `PROCEDURES.md` (they stay
on `main` and are referred to from `DEVELOPMENT.md`).

## Line endings

With `.gitattributes` from `main`, Turtle is checked out with LF. The three
1.1.0 SHACL files in `src/shaclShapes/` and the four 1.0.x SHACL files in
`releases/` and `drafts/` were stored with CRLF and were renormalised. The
content is otherwise unchanged (checked with `git diff --ignore-cr-at-eol`).

## NAPCORE ReSpec profile

| File | Change |
|------|--------|
| `src/js/respec-napcore.js` | New: copy of `respec-napcore.js` from `mobilityDCAT-AP/respec-style` at `7d3eef5` (ReSpec 25.16.5), with a header comment and two changes: `RE:"Recommendation",` added to the status labels; the three specref URLs moved from `specref.herokuapp.com` to `api.specref.org` |
| `src/index.html` | Loads `./js/respec-napcore.js` instead of `https://mobilitydcat-ap.github.io/respec-style/respec-napcore.js` |
| `src/config.js` | `specStatus: "ED"` → `"RE"`; `canonicalURI` `drafts/latest/` → `releases/1.1.0/`; `prevRecURI` `releases/1.0.1/` → `releases/1.1.0/` |

Why: the page loaded the profile live, so the build depended on that host and
on whatever it served. The profile cannot be replaced by the ReSpec pinned in
`package.json`, which is stock W3C and would change the header and status text.

The `config.js` values are the ones the published page shows. The published
page was rendered with `specStatus: "RE"`, which selects the stylesheet
`NAPCORE-RE.css`, but the profile has no label for `RE`, so it rendered as just
"NAPCORE", with a status sentence ending "as a .". The published HTML was then
edited by hand: "Recommendation" added to the header, the status sentence
changed to "as a recommendaton.", and the endorsement sentence deleted. Adding
the missing label makes the build produce the header without hand edits.

The profile looks up the bibliography at specref.herokuapp.com, which now
answers 404; specref moved to api.specref.org. With the upstream URLs the build
still succeeds, but 26 entries of the bibliography read "Reference not found."
and the about 50 links to sections of other specifications
(`data-cite="FOAF#term_Agent"` and the like) have no target.

Removing the header comment and undoing the two changes gives a file
byte-identical to upstream.

## Hand edits ported from the published page

These edits were made to `releases/index.html` after it was rendered and never
reached the source. A build from the source would have undone them.

| Published edit | Fix in the source |
|----------------|-------------------|
| `56dd3e7`: release date in the document history | `src/tables/document-history.html`: `18/12/2024` → `17/01/2025` |
| `faf4ed9`: Daham Mohammed Mustafa among the editors of the ReSpec documentation | `src/index.html`: added as the first item of that list |
| `8f829c7` (#164): stray `-->` before the Data Service section | `src/index.html`: `<!-- Data Service; PL edited 04.07.23- -->-->` → `<!-- Data Service; PL edited 04.07.23 -->` |
| `8ba434a` (#164): stray `>` before the Category Scheme table | `src/tables/class-category-scheme.html`: `<!-- PL edits 05.07.23 -->>` → `<!-- PL edits 05.07.23 -->` |
| `8ba434a` (#164): stray `#">intended information service<` | `src/tables/optional-properties-for-dataset.html`: text removed from the `<tr>` of `dataset-intended-information-service` |
| `8ba434a` (#164): stray `>` after "licence type" | `src/index.html`: `<li>-licence type</li>>` → `<li>-licence type</li>` |
| `8b0c12e` (#178): wording of the Annex G list of SHACL files | `src/index.html`: "extend and inherit from" → "extend"; " — " → ": " in the list; dashes → commas around "including Python scripts, Docker setup, and test cases" |

One fix with no published counterpart, in the same commit:

| Defect | Fix |
|--------|-----|
| The Mobility Data Standard class table was missing from a build of the source: its `data-include` had no `.html`. The published page has the table. | `src/index.html`: `./tables/class-mobility-data-standard` → `./tables/class-mobility-data-standard.html` |

Checked and not ported, because the source already had it: the order of
`foaf:mbox` and `foaf:workplaceHomepage` in the agent tables (`c690a85` on the
published page, `e531599` in the source), Daham Mohammed Mustafa as editor in
`config.js` (`ef494ac`), and the content-negotiation links (`8154676`).

## Source-only fix

`e0cd0c9` removed the `data-cite` link around "described" in the privacy
considerations but kept its closing tag. `src/index.html`: `described</a>` →
`described`.

## Documentation

| File | Content |
|------|---------|
| `DEVELOPMENT.md` | New: how to build, how the branch differs from `main`, how publishing works |
| `AGENTS.md`, `CLAUDE.md` | New: rules for coding agents on this branch; `CLAUDE.md` imports `AGENTS.md`, as on `main` |
| `README.md` | "Structure of the repository" describes this branch and `src/` |

## Verification

A full build from a fresh clone, run with the steps of `mise run lint`:

- `check-refs.py`: no broken local references.
- `html-validate`: 212 errors, the baseline for this branch. 162 are trailing
  whitespace in the ReSpec CLI output; the rest (`prefer-native-element`,
  `wcag/h63`, one `parser-error`) are also on the published page. The
  `close-order` and `no-implicit-close` errors of the published page are gone,
  because the stray tags are fixed.
- Every file published under `releases/1.1.0/` and `releases/latest/` on
  `gh-pages` (`2886909`) is in `dist/`.
- The generated `.rdf`, `.jsonld` and `.nt` are byte-identical between builds.
- `build-check.yml` on GitHub succeeded.

The visible text of the build was compared with the published root page
`releases/index.html` (text extracted from `#abstract` up to the references).
The header is the same: "NAPCORE Recommendation 17 January 2025", the same
links and the same stylesheet. The remaining differences are:

| Difference | Cause |
|------------|-------|
| "as a Recommendation." instead of "as a recommendaton." | Generated by the profile instead of typed by hand |
| "Publication as a Recommendation does not imply endorsement by the European Commission." is present | Generated by the profile; the hand edit had deleted it |
| "sub-working group 4.4 of the (NAPCORE) [NAPCORE] consortium" instead of "of the NAPCORE project [NAPCORE] consortium" | `e0cd0c9`, never published |
| "Data Catalog Vocabulary (DCAT) [DCAT]", with a new reference to DCAT 3 | `e0cd0c9`, never published |
| `[RDF-SCHEMA]` instead of `[RDF-SCHEMA]]` | `e0cd0c9`, never published |
| `[EU-EIP-CMC].` instead of `[EU-EIP-CMC].)` | `e0cd0c9`, never published |

The bibliography has the entries of the published page, plus `[DCAT]` from
`e0cd0c9`. Specref is looked up at build time, and its data has changed since
the page was published: `[DCAT-AP]` now cites the current DCAT-AP (10 July
2025) instead of version 2.0.1, `[VOCAB-DCAT-2]` the edition of 22 August 2024;
the other differences are in author and publisher names and in the form of the
URLs (e.g. `rfc-editor.org/info/rfc2119/` instead of `rfc-editor.org/rfc/rfc2119`).

The HTML is not byte-identical to the published page: that page was saved from
a browser, the build is written by the ReSpec CLI, so whitespace and markup
details differ.

## What the first publish changes on `gh-pages`

The first push of these commits to `release/1.1.0` replaces `releases/1.1.0/`,
and `releases/latest/` while `LATEST_RELEASE` on `main` reads `1.1.0`.
Compared with what is published now:

- the text differences in the table above, and the bibliography as specref
  has it today;
- `releases/1.1.0/` gets `figures/`, so its UML diagrams load again, and the
  Enterprise Architect documentation page;
- `mobilitydcat-ap.rdf` and `mobilitydcat-ap.jsonld` are regenerated from the
  Turtle (about 35 triples different from the published files);
- `mobilitydcat-ap.nt` is new.

The files in the root of `releases/` are not touched.

## Open points

- `publishDate` in `src/config.js` is still 2025-01-17. The hotfix procedure on
  `main` sets it to the date of the correction.
- `[DCAT-AP]` cites the current DCAT-AP, as specref returns it, not version
  2.0.1, which 1.1.0 is built on (its SHACL shapes import DCAT-AP 2.0.1). A
  `localBiblio` entry in `src/config.js` would pin it; to be decided by the
  editors.
- The `e0cd0c9` edits (including the DCAT 3 reference) become public with the
  first publish; to be confirmed by the editors.
- `releases/` and `drafts/` are still on the branch as copies of what was
  published by hand. They are not built or published from here.
- `REPOSITORY.md` on `main` still says that `release/1.1.0` has no build.
- The change log (Annex H.1) links `./mobilitydcat-ap_shacl_shapes.ttl`, which
  was removed in `f7442a9`; the link is broken on the published page too.
- There is no SHACL validation of the 1.1.0 examples; `validate-examples.py`
  from `main` would have to be adapted to the 1.1.0 shapes.
