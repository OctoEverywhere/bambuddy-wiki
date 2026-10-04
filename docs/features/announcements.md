---
title: Announcements
description: Messages from the Bambuddy maintainers, and exactly what Bambuddy fetches to show them
---

# Announcements

Short messages from the Bambuddy maintainers, shown inside Bambuddy: security fixes, breaking changes, new releases and calls for testers.

---

## :material-bullhorn: Where they show up

- **Sidebar:** a megaphone icon appears at the bottom of the sidebar, next to the System icon (directly above it when the sidebar is collapsed), for everyone who may see announcements, also when there are none. A green badge shows how many you haven't read.
- **List:** click it to open the list. Each message is one line (level, date, title) and opens when you click it. Unread messages have a green dot and a **New** label; opening a message marks it read.
- **Banner:** **important** and **critical** messages also show a banner above the page until you click **Got it** or open the message (**Read more** opens it straight away). **Info** messages never show a banner.

Read state is stored per user on the server, so a message you dismissed stays dismissed in every browser you sign in with.

**Earlier:** messages past their expiry move to a collapsed **Earlier** section at the bottom of the list, so you can still open them. History covers the last 12 months, up to 50 messages; it never counts as unread or shows a banner. Messages the maintainers **withdraw** (because they were wrong) are removed everywhere and don't appear in the history.

Messages are plain text. A message can carry one link, and only to `github.com` or `bambuddy.cool` (including `wiki.bambuddy.cool`); Bambuddy refuses any other link.

---

## :material-shield-check: What Bambuddy fetches

Bambuddy downloads one file, a minute after startup and then every 6 hours:

```
https://raw.githubusercontent.com/maziggy/bambuddy-notifications/main/feed.json
```

- It is a plain request to GitHub, the same host the update check uses. **No Bambuddy server is contacted.**
- **Nothing about your install is sent:** no ID, no version, no settings.
- Whether a message applies to you (Bambuddy version, beta channel, install type) is decided by your install, from what is in the file.

A new message therefore reaches a running install within about 6 hours (GitHub can also cache the file for up to 5 minutes). Once it arrives, it appears on any open Bambuddy page without a reload.

The file is public, and its [history](https://github.com/maziggy/bambuddy-notifications/commits/main) is the full record of every message ever sent, edited or withdrawn.

### Why a file on GitHub can't be faked

The file is signed with an Ed25519 key held by the maintainers; the matching public key is built into Bambuddy. A file that doesn't verify is ignored, so neither a copy of the repository nor anyone between you and GitHub can make Bambuddy show a message.

Each file also carries a number that only goes up. Bambuddy remembers the highest one it has seen and ignores anything older, so an old file can't be served again to bring back a withdrawn message.

If the download fails or the file is rejected, Bambuddy keeps showing the last good list. Offline and air-gapped installs simply never get any.

---

## :material-cog: Settings

**Settings → General → Updates**, section **Announcements**

| Setting | Default | |
|---|---|---|
| Receive announcements | On | Off: Bambuddy fetches nothing and shows nothing. |
| Show to all users | Off | Off: only administrators see them. Has no effect while authentication is off, where whoever runs Bambuddy sees them. |

---

## :material-file-document: What may be sent

- Security notices and urgent bug fixes
- Breaking changes and migration notes
- New releases
- Calls for testers

Never advertising, and never anything that asks you to enter data.
