# Symlink Notes

Portable Markdown shortcuts for Obsidian Desktop and Mobile. Symlink Notes keeps
the shortcut itself as ordinary Markdown and leaves the target note as the single
canonical copy of its content.

A shortcut is an ordinary note:

```yaml
---
symlink: Problems/Weak Bedroom Wi-Fi.md
---
```

Opening it opens the target in the same tab. The plugin never copies or
synchronizes note contents, and the shortcut remains readable Markdown even when
the plugin is disabled.

Version 0.1.1 requires Obsidian 1.5.7 or newer.

## Install

Install **Symlink Notes** from the Obsidian Community plugins directory:

1. Open **Settings** → **Community plugins** in Obsidian.
2. Select **Browse**, search for **Symlink Notes**, then select **Install**.
3. Enable Symlink Notes after installation.

For the design rationale behind Markdown-level shortcuts rather than filesystem
symlinks, see
[Symlink Notes: Portable Markdown Shortcuts for Obsidian](https://deadhedd.com/2026/09/30/symlink-notes-portable-markdown-shortcuts-for-obsidian/).

## Use

Open a Markdown note and run **Symlink Notes: Create symlink to active note** from the command palette. Choose an existing folder, or `/ (vault root)`. The shortcut uses the active note's filename. Creation leaves the active note open and refuses to overwrite an existing file or folder. Paths in generated YAML are quoted to preserve special characters.

Renaming or moving a target, including moving its containing folder, updates the `symlink` property of incoming shortcuts. Other properties and the note body are preserved; Obsidian may reformat YAML during its frontmatter update. Moving a shortcut itself leaves its target unchanged. Targets must be moved while the plugin is enabled for automatic updates to occur.

Chains resolve at navigation time, with loop detection and a maximum of 20 redirects. Missing, invalid, non-Markdown, circular, and excessively deep targets produce a notice and leave the original shortcut open for repair. Empty or non-string properties and malformed YAML behave as ordinary notes. Deleting a target leaves its shortcuts intact.

To repair a broken shortcut, edit its frontmatter and reopen it. Working shortcuts redirect immediately; to edit one manually, temporarily disable the plugin or use an external editor. With the plugin disabled, every shortcut remains readable Markdown.

## Behavior and limits

- Uses public Obsidian workspace, vault, metadata, and YAML APIs. No filesystem symlinks or Node.js APIs run in the plugin.
- Startup builds an in-memory index; file and metadata events maintain it. Navigation reads only the opened note and its chain, without scanning the vault.
- Redirects use the existing Markdown leaf, preserving its editor mode and avoiding focus changes for background tabs. Public navigation events run after opening, so a brief glimpse of the shortcut is possible. Native history can retain a shortcut entry, which will redirect again when opened.
- Frontmatter changes are limited to `symlink`. Backlinks, search, graph, File explorer, and Quick switcher use Obsidian's normal behavior.
- Absolute paths, URLs, `.`/`..` paths, attachments, and folder targets are unsupported.

Public API reference: [Obsidian TypeScript definitions](https://github.com/obsidianmd/obsidian-api/blob/master/obsidian.d.ts).

## Development

```sh
npm ci
npm run check
```

`npm run check` is the canonical repository verification command. It runs strict TypeScript checking, the test suite, and the production build in that order. `npm run dev` watches and rebuilds `main.js`. `npm run typecheck` checks strict TypeScript independently.

For a repository-owned release artifact, run:

```sh
npm ci
npm run release:check
```

The release command runs the canonical repository check before creating
`release/`. It creates exactly `main.js` and `manifest.json`, then reports
the version, output directory, and lowercase SHA-256 hashes for both files. No
stylesheet or runtime npm dependencies are needed.

## Release verification evidence

After `npm run release:check` succeeds, use the exact `release/main.js` and `release/manifest.json` it produced. Copy those files unchanged into a clean Desktop vault and a clean Mobile vault, then complete the smoke path in `docs/release-verification/template.md`. Copy the template to `docs/release-verification/<version>.md`, fill in the command output and test results, and check in the record even when a platform is `Fail` or `Blocked`. Stop if either recorded SHA-256 hash does not match the files under test.

Tests use Node's test runner with an in-memory Obsidian API mock and real YAML parsing. They cover resolution, creation, collisions, rename/move events, metadata/body preservation, restart reconstruction, invalid targets, loops, depth limits, background leaves, navigation races, unload, and write failures. They do not replace testing the plugin inside Obsidian.

Manual smoke test in a disposable desktop/mobile vault:

1. Create `Problems/Target.md` with some body text and a `Household` folder. Use the command to create `Household/Target.md`.
2. Open the shortcut from File explorer, Quick switcher, and an internal link. Check current, split, and new/background tabs; verify the target opens without unrelated tabs closing.
3. Repeat creation in `Household`; verify the existing shortcut is unchanged. Try the vault root as a destination.
4. Rename the target, move it to another folder, then move its containing folder. Inspect the shortcut frontmatter and verify unrelated properties/body content survive.
5. Move the shortcut, then rename the target again. Verify the shortcut still follows it.
6. Delete the target. Open the shortcut, verify the notice, and edit the broken path. Reopen after repairing it.
7. Create a two-hop chain, a circular pair, and an ordinary note. Verify correct navigation and safe circular-link handling.
8. Restart Obsidian. Verify shortcut navigation and target rename updates still work. Disable the plugin and verify shortcuts remain ordinary readable notes.

## License

Symlink Notes is licensed under the [MIT License](LICENSE).
