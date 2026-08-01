# GitHub Profile Statistics Card Design

## Goal

Create a concise GitHub profile README for `jhou1` whose visible content is a dynamically generated SVG statistics card. The card should resemble the supplied reference: a clean two-column activity summary with compact icons, contribution metrics, and notable contribution badges.

The profile should emphasize verifiable public GitHub activity rather than a written biography, job title, or technology list.

## Visible Content

The README will embed a single theme-aware SVG card. The card will contain:

- Identity: GitHub avatar, display name, account age, and follower count.
- Activity: commits authored, pull requests reviewed, pull requests opened, and issues opened.
- Repository summary: repositories contributed to, public repositories owned, stars received, and forks received.
- Notable contributions: compact badges for the highest-signal organizations and repositories.

Metrics that cannot be calculated reliably from the official GitHub APIs will be omitted instead of approximated or mislabeled.

## Contribution Definitions

- **Commits authored:** public commits returned by GitHub commit search for `author:jhou1`.
- **Pull requests reviewed:** unique pull requests returned by GitHub issue search for `reviewed-by:jhou1 type:pr`.
- **Pull requests opened:** pull requests returned by GitHub issue search for `author:jhou1 type:pr`.
- **Issues opened:** issues returned by GitHub issue search for `author:jhou1 type:issue`.
- **Repositories contributed to:** the union of repositories discovered across commits, opened pull requests, opened issues, and reviewed pull requests.
- **Notable organizations:** organization owners ranked by the user's contribution activity across the collected data.
- **Notable repositories:** repositories ranked by contribution activity, with merged pull requests weighted above other activity.

Counts represent public GitHub activity visible to the workflow token. Private activity is not included.

## Card Layout

The generated SVG will use a wide, two-column layout inspired by the supplied screenshot:

1. The upper-left area shows identity information.
2. The lower-left area lists activity metrics with small line icons.
3. The upper-right area shows a contribution heat strip and repository totals.
4. The lower-right area shows repository popularity totals.
5. The bottom area shows notable organization and project badges.

The SVG will include light and dark color variables selected with `prefers-color-scheme`. Text will remain legible when GitHub renders the asset through its image proxy. The layout will use fixed SVG coordinates to avoid relying on HTML or CSS features that GitHub README sanitization may remove.

## Components

### Data collector

A Python script will call GitHub's official REST API using only the Python standard library. It will:

- Read `GITHUB_TOKEN` when available.
- Paginate search and repository endpoints.
- Reject incomplete search results.
- Normalize repositories by `owner/name`.
- Distinguish organization owners from user owners through the GitHub user endpoint.
- Calculate totals, rankings, and badge contents.

### SVG renderer

The same script will render deterministic SVG markup from the normalized statistics model. External avatar images will be embedded or represented with safe fallbacks so that the card remains renderable from the repository.

The generated artifact will be stored at `assets/github-profile-stats.svg`.

### README

`README.md` will contain only the embedded statistics card wrapped in a link to the `jhou1` GitHub profile, plus non-visible generation markers if required. It will not contain a biography, job title, or technology-stack section.

### GitHub Actions workflow

A scheduled workflow will:

1. Check out the profile repository.
2. Run the generator with the repository `GITHUB_TOKEN`.
3. Run tests before accepting the generated asset.
4. Commit and push only when the SVG changes.

The workflow will run weekly and support `workflow_dispatch` for manual refreshes. It will request only the permissions required to read public GitHub data and write the generated file back to the repository.

## Failure Handling

- HTTP failures, rate limits, malformed responses, and incomplete GitHub search results will fail the generator.
- The generator will write to a temporary file and replace the SVG only after data collection and rendering complete successfully.
- A failed workflow will preserve the last valid card.
- Empty result sets for core activity queries will be treated as suspicious and will not overwrite an existing non-empty card.
- Missing optional fields such as a display name or avatar will use deterministic fallbacks.

## Verification

Automated tests will cover:

- API pagination and deduplication;
- classification of pull requests versus issues;
- unique reviewed-PR counting;
- repository union and organization filtering;
- contribution ranking and stable ordering;
- SVG escaping and required visible labels;
- light/dark theme styles;
- failure behavior for incomplete or invalid API results.

Before delivery, the generated SVG will be parsed as XML and visually rendered for inspection. The workflow YAML and README asset path will also be validated.

## Scope

This implementation does not publish a separate website, use a third-party statistics service, expose private contribution details, or add prose sections to the profile. It is limited to the profile README, generator, tests, generated SVG asset, and update workflow.
