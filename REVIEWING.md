# Reviewing a submission

Merging a pull request here puts code on users' machines. The sandbox limits what a plugin can do, but review is
what users are trusting.

## Before reading code

- [ ] "Files changed" shows exactly one file, under `plugins/`. If the pull request touches `.github/` or anything
      else, its check result means nothing: close it.
- [ ] The check passed. Open its summary: name, author, sites, cards and the file list are what you expect.
- [ ] The secret scan passed. If the author marked something `gitleaks:allow`, look at it yourself.
- [ ] One plugin per pull request, and the pull request touches only that plugin's entry.
- [ ] The commit is on a branch of the repository the entry names. Open the commit on GitHub: a banner saying it
      "does not belong to any branch on this repository" means it comes from someone's fork — reject it.
- [ ] The first part of the plugin's id fits its author (a stranger shouldn't list `fourfold.something`).
- [ ] The name, short label and author don't pose as FourFold, a built-in plugin, or another author.
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

- Squash-merge. The publish workflow runs on `main`; check that it finished and that the plugin appears in FourFold.

## Pulling a plugin

1. Delete `plugins/<id>.json`.
2. Add `{ "id": "<id>", "reason": "…" }` to `removed.json`. Users see the reason, so write it for them.
3. Commit both in one pull request and merge. FourFold switches the plugin off at each user's next hub check
   (at start, every 6 hours, or when they open the hub).

Never merge a pull request that deletes an entry without adding it to `removed.json`, unless you mean to delist it
silently: users who have it installed are told it is no longer on the hub.

To list it again later, remove the line from `removed.json` and restore the entry.

## Repository settings

- Secret scanning and push protection: on.
- `main`: changes by pull request only, with the "Check submission" check required.
- A pull request that changes nothing under `plugins/` and not `removed.json` (a README or workflow change) doesn't
  start the check, so the required check never reports and GitHub blocks the merge. Merge those yourself as an
  administrator.
- Until part 3 of the plugin system is on the app's `main` branch, both workflows build the hub tool from the app's
  `feature/plugin-hub` branch. After that, change `ref:` in both workflows to `main`.
