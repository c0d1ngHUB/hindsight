# Hindsight documentation site

The public documentation is built with Docusaurus. This directory is an npm
workspace of the repository; run the commands below **from the repository
root**. Node.js 20 or newer is required by this workspace's `package.json`.

## Install and preview

```bash
npm ci --workspace=hindsight-docs
./scripts/dev/start-docs.sh
```

The helper starts the site at `http://localhost:3000` without opening a browser
and sets `INCLUDE_CURRENT_VERSION=true`, so edits to current, unreleased docs
appear in the preview. To start the workspace directly with the same setting:

```bash
INCLUDE_CURRENT_VERSION=true npm run start --workspace=hindsight-docs -- --no-open
```

Without that flag, the configuration selects the released documentation
versions. Check the current-version preview when changing files in `docs/`.

## Where to edit

- `docs/`: current product, developer and SDK documentation.
- `docs-integrations/`: integration documentation and its frontmatter.
- `src/pages/`: standalone pages, including release changelogs.
- `versioned_docs/` and `versioned_sidebars/`: released snapshots; keep them
  unchanged when documenting an unreleased fix.
- `static/`: published assets, OpenAPI specification and bank-template schema.

The coding-agents integration has a separate source of truth:
[`hindsight-integrations/coding-agents/README.md`](../hindsight-integrations/coding-agents/README.md).
Do not edit its generated docs page or `skill/SKILL.md` by hand. From the root,
regenerate them with:

```bash
(cd hindsight-integrations/coding-agents && npm run skill:build)
node hindsight-docs/scripts/sync-coding-agents-doc.mjs
```

Follow [CLAUDE.md](../CLAUDE.md) for generation order, configuration documentation,
OpenAPI changes and release conventions. Changelog entries are created by release
scripts; describe unreleased fixes in the PR instead.

## Validate and build

```bash
npm run build --workspace=hindsight-docs
npm run serve --workspace=hindsight-docs
```

`build` runs code-tab parity, integration SEO and registration checks, generated
coding-agents documentation freshness, and template-schema validation before
Docusaurus generates `hindsight-docs/build/`. The serve command previews that
output. To build with current docs included, prefix the build command with
`INCLUDE_CURRENT_VERSION=true`.

The dependency-free checks can also be run individually from the root:

```bash
node hindsight-docs/scripts/check-code-parity.mjs
node hindsight-docs/scripts/check-integration-seo.mjs
node hindsight-docs/scripts/check-integrations.mjs
node hindsight-docs/scripts/sync-coding-agents-doc.mjs --check
```

Template validation needs the installed npm dependencies. The complete build is
the check that also compiles the site and resolves its links. Integration
registration also checks release tags when available; a checkout without
integration tags skips that reverse check. Fetch the relevant tags to reproduce
the release-registration check locally, as the deployment workflow does with
`fetch-depth: 0`.

## Publishing and forks

[The deployment workflow](../.github/workflows/deploy-docs.yml) builds the site,
generates the full LLM documentation with `uv run generate-llms-full`, uploads a
Pages artifact and deploys it through GitHub Actions. It runs for relevant pushes
to `main` or an explicit workflow dispatch and needs Node.js, uv and Pages
permissions.

The checked-in Docusaurus configuration still names the upstream project
(`vectorize-io/hindsight`) and `https://hindsight.vectorize.io`. A fork can build
and preview locally without changing these values. Before publishing a fork,
review the site URL, organization/project settings and Pages environment for
that fork; the generic Docusaurus `npm run deploy` command is not the repository's
configured Actions publishing path.
