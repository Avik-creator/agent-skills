# Releasing a version

Read only when the project has consumers (a package, an API others call, a deployed client) and the user asks for a version bump, a tag or a changelog. The agent prepares the release; the user pushes the tag.

## The bump (Semantic Versioning)
- **MAJOR**: incompatible API changes.
- **MINOR**: functionality added in a backward compatible manner.
- **PATCH**: backward compatible bug fixes.
- Judge the bump by what consumers can observe, not by the size of the diff. If you're unsure whether a change breaks anything, treat it as breaking and ask. Why: a surprise major costs less than a broken consumer.
- Below 1.0.0, anything may change at any time. Say so instead of promising compatibility.
- A breaking change gets a migration note, and a deprecation window when the old path can live on.

## The changelog (Keep a Changelog)
- The newest version comes first. Keep an `Unreleased` section at the top and add to it with each change, while the impact is fresh.
- Group entries under Added, Changed, Deprecated, Removed, Fixed, Security.
- Write each entry for consumers: what they notice and what they must do. Not internal detail, and not a dump of commit messages.

## The tag
- The tag is the source of truth for the version. Derive the version from it rather than editing it by hand in several files.
- Show the commands. The user runs them, because pushing a tag publishes it:
  ```bash
  git tag -a v<x.y.z> -m "Release <x.y.z>"
  git push origin v<x.y.z>
  ```
- Move the `Unreleased` entries under the new version in the same change, before the tag.
