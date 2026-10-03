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
2. Test the exact commit you want listed in FourFold, with developer mode on. The check runs on Linux, where names are
   case-sensitive: `panel`, `icon` and `path` must match the real file and folder names letter for letter, even
   though Windows lets a mismatch work in the dev folder.
3. Fork this repository and add one file, `plugins/<your plugin id>.json`:

   ```json
   {
     "repository": "https://github.com/you/your-plugin",
     "commit": "0123456789abcdef0123456789abcdef01234567"
   }
   ```

   - The file's name is your plugin's `id` from `plugin.json`, plus `.json`.
   - `repository` must be exactly `https://github.com/owner/repo`, with no trailing slash or `/tree/...`.
   - `commit` is the full 40-character commit, in lowercase. A branch or a tag isn't accepted, because it can move.
   - If the plugin isn't at the repository's root, add `"path": "folder/inside/the/repo"`. `path` is written with
     forward slashes, uses only letters, digits, `.`, `_` and `-`, and has no trailing slash.

4. Open a pull request that changes only that one file. A maintainer has to approve the check before it runs; its
   summary page then shows what it found. If it fails, fix the problem in your plugin's repository and put the new
   commit in your entry.
5. A maintainer reads the plugin's code at that commit. When they merge, the plugin is on the hub within a few
   minutes.

### Update a plugin

Raise `version` in your `plugin.json`, commit, and open a pull request that changes `commit` in your entry. A
maintainer reviews what changed since the listed commit.

### The rules the check enforces

- The pull request changes only your own entry file, `plugins/<your plugin id>.json`. A pull request that touches
  anything else is closed.
- `plugin.json` must be valid, and its `id` must match the entry's file name.
- `version` is three numbers joined by dots, such as `1.4.0`, and each number can be at most 2147483647. An update
  must raise it, comparing number by number (`1.10.0` is higher than `1.9.0`).
- The check downloads your whole repository at that commit, not only the plugin's folder. It must be public, under
  50 MB as a download (200 MB unpacked), and hold at most 20,000 files. No path in it can have more than 63 parts,
  counting the folders from the repository root and the file's name.
- A package is `plugin.json` plus every file of a type FourFold serves (`.html .js .mjs .css .json .txt .png .jpg
  .jpeg .gif .svg .webp .woff2`) in the plugin's folder: at most 500 files and 5 MB in all. Every such file goes in,
  whether or not your page loads it, so keep tests and screenshots in another folder. Everything else is left out,
  such as `README.md` and `LICENSE`, and so is any file or folder whose name starts with a dot. Your panel page, and
  everything it loads, must be in the package, so none of them can start with a dot.
- Links (symlinks) are left out. So is any file with a `\` or a `:` in its path.
- The files that go in must unpack on Windows: a file or folder name can't contain `< > " | ? *` or a control
  character, can't end in a dot or a space, and no two files can differ only by capital letters.
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
  scans the submitted repository for secrets. It never runs a plugin's code. `guard.yml` runs from `main`, so a pull
  request can't change it, and fails a pull request from a fork that changes anything but entry files.
- On a merge to `main`, `publish.yml` packages new and changed plugins and uploads them, with `catalog.json`, to
  the [`catalog` release](../../releases/tag/catalog). FourFold downloads the catalog and packages from there and
  refuses a package whose size or SHA-256 doesn't match the catalog. The publish also keeps the catalog it replaces
  as `catalog.previous.json`, which only the workflows read.
