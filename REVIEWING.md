# Reviewing a submission

Merging a pull request here puts code on users' machines. The sandbox limits what a plugin can do, but review is
what users are trusting.

## Before reading code

- [ ] On the pull request's own **Files changed** tab (GitHub's, not the check's summary: a pull request that
      edits the workflow can make the summary say anything): exactly one file, under `plugins/`. Look before you
      click **Approve and run**. "Entry files only" must be green.
- [ ] The check passed. Open its summary: name, author, sites, cards and the file list are what you expect.
- [ ] The secret scan passed. If the author marked something `gitleaks:allow`, look at it yourself; the summary says
      how many files carry that comment. The scan is a net for accidents, not a control: it skips images, fonts,
      `.svg` files and `node_modules/`, and anything the tool leaves out when it unpacks (links, names with `:` or
      `\`).
- [ ] One plugin per pull request, and the pull request touches only that plugin's entry.
- [ ] The commit is on a branch of the repository the entry names. Open the commit on GitHub: a banner saying it
      "does not belong to any branch on this repository" means it comes from someone's fork — reject it.
- [ ] The first part of the plugin's id fits its author (a stranger shouldn't list `fourfold.something`).
- [ ] The name, short label and author don't pose as FourFold, a built-in plugin, or another author.
- [ ] The plugin's name and author are readable: not blank-looking characters, not letters buried in stacked marks.
- [ ] If the summary shows the any-website WARNING, or a "Changed:" line you didn't expect, stop and ask why.

## Reading the code

For a new plugin, read everything at the commit. For an update, use the compare link in the check's summary.

- [ ] **Network.** Every request goes to a declared site, and each declared site is needed. Look for hosts built
      from strings, and for `fetch`, `XMLHttpRequest`, `WebSocket`, `EventSource`, `sendBeacon`, `<img>`, `<link>`
      and CSS `url()`.
- [ ] **What leaves.** Account labels, in-game names, XP and stats go nowhere the description doesn't say.
- [ ] **Channels FourFold can't lock.** Reject a plugin that uses any of these:
  - WebRTC in any spelling (`RTCPeerConnection`, `webkitRTCPeerConnection`, data channels);
  - `<link rel="dns-prefetch">` or `<link rel="preconnect">`;
  - iframes;
  - `window.print`;
  - file pickers or drag-and-drop of files;
  - clipboard writes the user didn't ask for;
  - Web Audio or other sound.
- [ ] **Storage.** `fourfold.storage` for settings; no large IndexedDB or Cache use without a reason.
- [ ] **Imitation.** Nothing in the panel looks like FourFold's own screens: no sign-in boxes, no "FourFold needs…"
      messages, no fake update prompts.
- [ ] **Links.** `openExternal` targets are sensible places to send a user.
- [ ] **Secrets.** No keys or tokens, however obfuscated. A plugin that needs a secret key can't be listed.
- [ ] **Obfuscation.** Minified third-party libraries are fine when they are a known library at a known version;
      the author's own code must be readable.
- [ ] **Behaviour.** Run it in FourFold with developer mode on, from that exact commit.

## Any-website access

Set `"anySite": true` in the entry only when the plugin's purpose needs it (a general link previewer, say) and you
have read every request it makes. It also lets the plugin open any https link in the user's browser.

## Merging

- [ ] If the author pushes or changes the entry after you read the code, read again: you approve one commit.
- Squash-merge. The publish workflow runs on `main`; check that it finished and that the plugin appears in FourFold.

## Pulling a plugin

1. Delete `plugins/<id>.json`.
2. Add `{ "id": "<id>", "reason": "…" }` to `removed.json`. Users see the reason, so write it for them.
3. Commit both in one pull request and merge. FourFold switches the plugin off at each user's next hub check
   (at start, every 6 hours, or when they open the hub).

Never merge a pull request that deletes an entry without adding it to `removed.json`, unless you mean to delist it
silently: users who have it installed are told it is no longer on the hub.

When a plugin was pulled for cause, delete its `.zip` files from the `catalog` release too: they stay downloadable
otherwise. A release holds at most 1,000 files; when it nears that, delete zips no listed plugin names any more.

To list it again later, remove the line from `removed.json` and restore the entry. Listing a pulled plugin again lets
its old version run on machines that still have it, until they update. If the old version must never run again, keep
its id in `removed.json` and list the fixed plugin under a new id.

## Repository settings

The hub relies on all of these. The workflows can't set them.

1. Settings → Actions → General → "Approval for running fork pull request workflows from contributors":
   **Require approval for all external contributors.** The default only asks for first-time contributors.
2. Settings → Actions → General → Workflow permissions: **Read repository contents** only, and "Allow GitHub
   Actions to create and approve pull requests" off. The publish workflow asks for write access itself.
3. Settings → Actions → General → Actions permissions: allow only actions created by GitHub. Nothing else is used.
4. Settings → Rules (or Branches) → `main`: require a pull request; require the checks **Check submission** and
   **Entry files only** (a check appears in the picker only after it has run once); block force pushes and
   deletion. Leave the administrator bypass on, or the hub's own changes can't be merged.
5. No collaborators. Anyone with write access can run a changed publish workflow from a branch, or edit the
   release's files by hand, whatever protects `main`.
6. Settings → Rules → a tag rule for `catalog` (after the first publish has created it): restrict updates and
   deletions. Deleting the tag takes the catalog away from every user.
7. Settings → General → Releases: immutable releases **off**. The publish replaces `catalog.json` in place.
8. Settings → Code security: secret scanning and push protection **on**.
9. Settings → General → Pull Requests: auto-merge **off**. A merge is a deliberate click.

Changes to the hub itself (`.github/`, the documents) never start the submission check. Merge your own such
changes as an administrator. Never merge a stranger's change to `.github/`: a workflow on `main` holds the token
that writes the catalog.

Until part 3 of the plugin system is on the app's `main` branch, both workflows build the hub tool from the app's
`feature/plugin-hub` branch. After that, change `ref:` in both workflows to `main`.
