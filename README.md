# FourFold Account Manager plugin hub

The list of community plugins for [FourFold Account Manager](https://github.com/CodySimonds65/FourFoldAccountManager).
FourFold's maintainers review every plugin before it is listed, and FourFold installs plugins only from here.

## For users

You don't need anything from this repository. In FourFold, click the wrench at the bottom of the plugin strip, then
**Plugin hub**. Installed plugins update themselves when a new reviewed version is listed.

## For plugin authors

How to write a plugin, and everything it can and can't do, is in
[PLUGIN_AUTHORS.md](https://github.com/CodySimonds65/FourFoldAccountManager/blob/main/PLUGIN_AUTHORS.md).

### List a plugin

1. Put the plugin in a **public GitHub repository**. It can be at the repository's root or in a folder.
2. Test the exact commit you want listed in FourFold, with developer mode on.
3. Fork this repository and add one file, `plugins/<your plugin id>.json`:

   ```json
   {
     "repository": "https://github.com/you/your-plugin",
     "commit": "0123456789abcdef0123456789abcdef01234567"
   }
   ```

   - The file's name is your plugin's `id` from `plugin.json`, plus `.json`.
   - `commit` is the full 40-character commit, in lowercase. A branch or a tag isn't accepted, because it can move.
   - If the plugin isn't at the repository's root, add `"path": "folder/inside/the/repo"`.

4. Open a pull request. A check runs, and its summary page shows what it found.
5. A maintainer reads the plugin's code at that commit. When they merge, the plugin is on the hub within a few
   minutes.

### Update a plugin

Raise `version` in your `plugin.json`, commit, and open a pull request that changes `commit` in your entry. A
maintainer reviews what changed since the listed commit.

### The rules the check enforces

- The pull request changes only your own entry file, `plugins/<your plugin id>.json`. A pull request that touches
  anything else is closed.
- `plugin.json` must be valid, and its `id` must match the entry's file name.
- An update must raise the version.
- A package is `plugin.json` plus the file types FourFold serves (`.html .js .mjs .css .json .txt .png .jpg .jpeg
  .gif .svg .webp .woff2`): at most 500 files and 5 MB. Other files in your repository are simply left out.
- No links (symlinks) in the repository.
- **No secrets.** The whole repository is scanned at that commit. Everything in a plugin is public and runs on
  users' machines, so a plugin can never hold a secret key. If the scan finds one, remove it **and revoke it** —
  it is already public in your repository. If it is a false positive, add a `gitleaks:allow` comment on that line
  and say so in the pull request.

### Access to any website

A plugin normally reaches only the sites its `plugin.json` declares. Access to any website (`"anySite": true` in
`plugin.json`) only works when a maintainer also sets `"anySite": true` in the plugin's entry here. Ask for it in the
pull request and say why the plugin needs it. Don't set it in the entry yourself.

### If a plugin is pulled

A maintainer can remove a plugin by deleting its entry and adding it to `removed.json` with a reason. FourFold then
switches it off for everyone who has it, and shows them the reason.

## For maintainers

See [REVIEWING.md](REVIEWING.md).

## How it works

- `plugins/<id>.json` — one entry per listed plugin.
- `removed.json` — plugins that were pulled.
- On a pull request, `check.yml` validates each changed entry with FourFold's own rules, builds its package, and
  scans the submitted repository for secrets. It never runs a plugin's code.
- On a merge to `main`, `publish.yml` packages new and changed plugins and uploads them, with `catalog.json`, to
  the [`catalog` release](../../releases/tag/catalog). FourFold downloads the catalog and packages from there and
  refuses a package whose size or SHA-256 doesn't match the catalog.
