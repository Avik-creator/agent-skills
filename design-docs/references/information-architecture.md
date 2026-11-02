# Information architecture

Read when planning a product's or site's structure before visual design: site map, navigation, page content priority, user flows, URL rules, naming. Information architecture (IA) is the skeleton. It comes after the brief, and before visual design and building.

## Before the interview

1. Find the brief. Look for an existing design brief or PRD in the repo, for example `.design/*/DESIGN_BRIEF.md`, `docs/`, or a PRD written earlier. If there are several, use the most recent or ask which feature this is for. If there is none, ask what is being built and for whom.
2. Read the structure that already exists, and extend it rather than replace it. Why: an IA that ignores what is built is a rewrite proposal, not a plan.
   - Routes: Next.js `app/` or `pages/`, React Router, Vue Router, SvelteKit routes, static HTML files
   - Navigation: header, sidebar, navbar, breadcrumb, footer components
   - Layouts: root and nested layouts, page wrappers
   - URLs: existing slugs, dynamic segments, query parameter conventions
   - Content: CMS models, API routes, MDX or content folders

## Interview

Ask with a recommended answer each time:

- What do users most need to find or do? Rank by how often.
- How many levels of navigation are acceptable?
- Which content grows over time, and which is fixed?
- Do different kinds of users need different entry points?
- Which page or view do users spend most of their time on?

## Document template

```markdown
# Information architecture: <product>

## Site map
Every page or view, nested by indentation, with its URL.
- Home `/`
  - Projects `/projects`
    - Project `/projects/:projectId`
- Settings `/settings`
  - Profile `/settings/profile`

## Navigation
- Primary: what is in the main navigation (and the most items it may hold)
- Secondary: sidebars, tabs, or links inside a section
- Utility: account, settings, help
- Mobile: how navigation changes on small screens

## Content priority
### <page>
1. <most important content> - why it comes first
2. <next> - why
3. <below the fold>

## User flows
See flows-and-diagrams.md for the format.

## Naming
| Concept | Label in the interface | Why this word |
|---|---|---|

## Shared layout and navigation components
| Component | Used on | Differences |
|---|---|---|

## Growth
Which sections grow, and how the structure copes: pagination, filters, search, archives.

## URL rules
- Pattern, such as `/section/item-slug`
- Dynamic segments
- Query parameters for filtering, sorting and pages
```

## Rules

- Pick one word per concept, and use it in the interface, the URLs and the docs. Why: users read two words as two different things.
- Put the most frequent task within one click or tap of where users land.
- Every page in the site map is reachable from the navigation or from a named flow. An orphan page is either a missing link or a page to cut.
