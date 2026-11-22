# Contributing to the Berlioz documentation

This repository contains the user documentation of the [Berlioz Framework](https://getberlioz.com). The framework
itself lives in the [Berlioz monorepo](https://github.com/BerliozFramework/Berlioz).

## Workflow

- Branch from `3.x` and name your branch `docs/<topic>`.
- Open a pull request against `3.x`. Its title follows
  [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/): `docs(<package>): <description>`, for
  example `docs(router): document wildcard route parameters`.
- When the pull request documents a change of the framework, link the framework issue or pull request.
- The **Docs** workflow must pass. See [Checks](#checks) to run it locally.

## Writing checklist

Before you open a pull request, read your page as its reader would and answer four questions. They follow the
principles of plain language (ISO 24495-1): readers must get what they need, find it, understand it and use it.

### Is it relevant?

- Start from what the reader wants to do, not from how the framework is built.
- Say who is concerned when a section does not apply to everyone, for example "If you wrote your own queue class".
- Mark new behavior with the version that introduces it. See [Version markers](#version-markers).
- When a change affects existing projects, add an entry to the [upgrade guide](#upgrade-guide).

### Is it findable?

- Document each topic in one place, and link to it from the other pages instead of repeating it.
- Write headings that describe the content. Do not skip a heading level.
- Link every new page from at least one other page, and add it to `index.md` when it is a main topic.
- Add every new configuration key to the default configuration shown in `getting-started/config.md`.

### Is it understandable?

- Write short sentences with one idea each. Use the active voice, the present tense and "you".
- Put the condition before the instruction: "If you use PHP-FPM, set ...".
- Use the same word for the same thing on every page.
- Say what a setting does and what happens by default before you describe the details.
- Write in American English.

### Is it usable?

- Give a complete example that works when copied: imports, configuration keys, expected result.
- For a procedure, list the steps in the order the reader performs them.
- State the defaults, the limits and the errors the reader can meet.
- Check each statement against the framework code or its tests.

## Conventions

### Pages

- One file per page, with a single level-one heading.
- The front matter controls the summary of the website:
    - `breadcrumb`: position of the page in the summary, one entry per level;
    - `summary-order`: position of the page among its siblings. To order the parent levels too, give one value per
      breadcrumb level, separated by `;` (for example `;;5`);
    - `summary-visible`: set it to `false` to hide the page from the summary;
    - `keywords`: search keywords.
- Wrap lines at 120 characters. Tables and code are exempt.
- Give a language to every fenced code block (`php`, `json`, `bash`, `twig`, `text`...).
- Indent nested lists with 4 spaces. Start and end every table row with a pipe.

### Callouts

Use only these four forms. The website renders them as highlighted blocks.

```markdown
> 🆕 **Info**: *Since version 3.4*
>
> What is new in this version.

> ℹ️ **Note**: Additional information.

> ⚠️ **Warning**: Something the reader must not miss.

> 💡 **Tip**: An optional recommendation.
```

Keep plain blockquotes for quotations of an external source.

### Version markers

- Mark a new section with `> 🆕 **Info**: *Since version X.Y*` under its heading.
- Mark a single list item or sentence with `*(since version X.Y)*`.
- Mark a deprecation with `> ⚠️ **Warning**: *Deprecated since version X.Y*`. Say what replaces the feature and in
  which version it will be removed. Add it to the "Preparing for version 4" table of
  `getting-started/upgrading.md`.

### Upgrade guide

Each minor version has a section in `getting-started/upgrading.md`. Put every entry in one of these groups:

1. **Required actions**: what every project must do when it upgrades.
2. **Behavior changes**: what works differently, grouped by feature.
3. **Deprecations**: what still works but must be migrated before the next major version.
4. **If you extend the framework**: changes of interfaces and of overridable methods.

Start each entry with a bold label, say what changed, then what the reader must do, and link to the detailed page.

### Links and headings

- Link to other pages with a relative path to the `.md` file: `[Routing](../http/routing.md)`.
- Link to a section with the anchor GitHub generates for its heading: lowercase, spaces replaced with hyphens,
  punctuation removed. `### Snapshot in progress` becomes `#snapshot-in-progress`.
- Renaming a heading changes its anchor. Search the repository for links to the old anchor before you rename it.

## Checks

The **Docs** workflow runs on every pull request:

- [markdownlint](https://github.com/DavidAnson/markdownlint-cli2) checks the Markdown structure (heading levels,
  code block languages, lists...);
- [lychee](https://github.com/lycheeverse/lychee) checks the internal links and their anchors.

Run them locally with Docker, from the root of the repository:

```bash
docker run --rm -v "$PWD:/workdir" davidanson/markdownlint-cli2:v0.24.2 "**/*.md"
docker run --rm -v "$PWD:/docs" -w /docs lycheeverse/lychee:0.24.2 \
    --offline --include-fragments=anchor-only --no-progress "**/*.md" .github/CONTRIBUTING.md
```
