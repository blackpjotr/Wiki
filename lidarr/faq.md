---
title: Lidarr FAQ
description: Frequently asked questions and common issues with solutions for Lidarr music management
published: true
date: 2026-04-20T13:02:12.278Z
tags: lidarr, troubleshooting, faq, questions, help, common-issues
editor: markdown
dateCreated: 2021-06-14T14:33:41.344Z
---

# Lidarr FAQ

> This page answers the questions that come up most often in the Lidarr help channel, Reddit, and forums. Longer troubleshooting flows and conceptual material live on dedicated pages — the FAQ links out to those when a full walkthrough is warranted.
{.is-info}

If you don't find your question here, the most common landing spots are [Metadata Troubleshooting](/lidarr/metadata-troubleshooting) (anything about missing or out-of-date MusicBrainz data), [Import Troubleshooting](/lidarr/import-troubleshooting) (downloads that finish but do not import), [Troubleshooting](/lidarr/troubleshooting) (general runtime issues), and [Tips and Tricks](/lidarr/tips-and-tricks) (power-user recipes). Ask in [Discord](https://lidarr.audio/discord) or on the [subreddit](https://reddit.com/r/lidarr) if the wiki doesn't answer it — recurring questions feed back into this FAQ.

## Basics

### How does Lidarr work?

Lidarr does *not* regularly search for album files that are missing or have not met their quality goals. Instead, it queries your indexers and trackers at a steady cadence for *all* newly-posted releases, then compares that feed with the list of albums it's monitoring and downloads what matches. At an RSS interval of 15–60 minutes this amounts to 24–100 queries per day, which covers a library of any size — but only going forward.

For albums released in the past, use **Wanted → Missing** or **Wanted → Cutoff Unmet**, or use the search button on the album's page. Adding an album with the *Start search for missing album* checkbox ticked does the same thing at add-time.

If Lidarr has been offline for a while, it pages back through each indexer to find the last release it processed so that nothing is missed, provided the indexer supports paging and the outage wasn't too long.

**When Lidarr does search actively** (as opposed to passively watching the RSS feed):

- User- or API-triggered search from the album/artist page, or from Wanted → Missing / Cutoff Unmet.
- Adding an artist or album with *Add and Search*.
- Albums discovered during an artist metadata refresh (for example, a new album MusicBrainz adds after you've already added the artist).

### How do I update Lidarr?

Lidarr runs on one of three release branches. Pick one under **Settings → General → Updates** (show advanced):

- `master` — default, stable. Updated roughly monthly after testing in `develop` and `nightly`.
- `develop` — beta. New features and fixes land here after `nightly`. Updated weekly or biweekly and tagged `pre-release` on GitHub.
- `nightly` — alpha. Built on every commit that passes CI. Required branch for plugin support (see [Plugins](/lidarr/plugins)). Expect occasional breakage and be prepared to recover a failed update yourself.

> **Switching from `nightly` or `develop` back to `master` may not be possible without restoring an older database backup.** Database schema migrations are one-way — if a newer branch introduced columns `master` does not know about, the older version will refuse to start with errors like *Error parsing column N*. If you already switched and need to go back, restore from a pre-switch backup.
{.is-danger}

**Installing the update:**

- **Native installs** — System → Updates. Any available update has an Install button.
- **Docker** — repull your image tag and recreate the container. Do *not* run the in-app updater inside a Docker container; that is a primary Docker anti-pattern and can put the image and the mounted data out of sync.

Docker tag-to-branch mapping:

|                                                                    | `master` (stable) | `develop` (beta) | `nightly` (alpha) |
| ------------------------------------------------------------------ | ----------------- | ---------------- | ----------------- |
| [hotio](https://hotio.dev/containers/lidarr)                       | `release`         | `testing`        | `nightly`         |
| [LinuxServer.io](https://docs.linuxserver.io/images/docker-lidarr) | `latest`          | `develop`        | `nightly`         |

> **Branch-switching caveat.** Identical-version branch swaps are safe. Swapping forward (e.g. `master` → `develop`) is typically safe. Swapping backward (e.g. `develop` → `master`) is the path that can fail. If in doubt, check in Discord with the development team before switching, and take a backup either way.
{.is-warning}

## Finding music and MusicBrainz

### Why doesn't artist X show up in search?

A few common causes:

- **Name variant mismatch.** Lidarr searches by the artist's *primary* MusicBrainz name. If you're searching by an alias, a translation, or a stylisation that differs from the primary name, the result may not appear. Search by the MusicBrainz ID instead — prepend `lidarr:` to the MBID, for example `lidarr:9255f594-b912-4bdf-87a2-ada04502a459` for Muse. See [How can I find a MusicBrainz ID?](#how-can-i-find-a-musicbrainz-id) below.
- **Not in MusicBrainz.** Lidarr only knows about artists MusicBrainz knows about. If the artist is absent from MB entirely, the fix is to add them there — see [Metadata Troubleshooting → Updating MusicBrainz](/lidarr/metadata-troubleshooting#updating-musicbrainz). New additions can take a refresh cycle (up to ~1 hour) to reach Lidarr.
- **Unknown release type on every album.** An artist with nothing but releases MB has catalogued as `unknown` will appear to be "missing" relative to most metadata profiles. The artist is there; the metadata profile is filtering everything out. See [Why does Lidarr only show studio albums, how do I find singles or EPs?](#why-does-lidarr-only-show-studio-albums-how-do-i-find-singles-or-eps) and [Metadata Troubleshooting](/lidarr/metadata-troubleshooting).

Release groups work the same way — `lidarr:<release-group-mbid>` looks up a specific album directly.

### Why does Lidarr only show studio albums, how do I find singles or EPs?

Lidarr filters what to track on each artist through a **Metadata Profile** (Settings → Profiles → Metadata Profiles). The default profile includes only *Studio* albums, which is why singles, EPs, live albums, compilations, and remix collections are absent on a fresh install.

Adjust the profile rather than per-artist settings for a library-wide change:

- **Album Types** — pick which MusicBrainz release-group types (Album, Single, EP, Live, Compilation, Remix, Soundtrack, Other) to include.
- **Release Statuses** — Official is included by default; Promotion, Bootleg, and Pseudo-Release are opt-in. Most releases that appear missing are catalogued as `official`; if nothing is showing up for an artist and you know they've released music, check whether the releases in MB have an `unknown` status and fix that upstream.

> **A single is not a "short album"** — singles are their own release-group type in MusicBrainz. Enabling "Single" in your Metadata Profile doesn't show you short albums; it adds single-type release groups (usually one track each) to your library.
{.is-info}

To apply a changed profile to existing artists, go to Library, select the artists, and use Edit → Metadata Profile. You can also manually import a specific recording from an album that *is* in your library.

### How can I find a MusicBrainz ID?

1. Search the artist, release group, or release on [MusicBrainz](https://musicbrainz.org/search). For albums, set the search type to **Release Group**.
2. Open the entity page. The MBID appears under the **Details** tab, or at the end of the URL (the UUID after the last slash).

![musicbrainz_id_detail_tab.png](/images/musicbrainz_id_detail_tab.png)

> **Release vs Release Group is the single most common mistake.** Lidarr's *album* = MusicBrainz's *release group*, not its *release*. A release group is "the 2005 album"; a release is "the 2005 US CD pressing" or "the 2005 UK vinyl." The **Release Group** MBID is what Lidarr wants. If you paste a release MBID, either the lookup fails or you get unexpected results.
{.is-warning}

### I cannot find a release in Lidarr but it is on MusicBrainz

Three common causes:

- **Propagation lag.** Lidarr's copy of MusicBrainz refreshes on an hourly cycle via the Servarr metadata server. If you edited MB in the last hour, wait a refresh cycle before troubleshooting further.
- **Unknown release type.** If the MB release group has `Type: Unknown` or `Status: Unknown`, most metadata profiles filter it out. Fix the type at MusicBrainz.
- **Metadata server cache needs busting.** Rare but it happens — especially after big MB edits. The full flow, including the `!refresh` bot command, is on [Metadata Troubleshooting](/lidarr/metadata-troubleshooting).

If none of those apply, the full troubleshooting flow lives on the [Metadata Troubleshooting](/lidarr/metadata-troubleshooting) page.

## Importing and renaming

### I'm having trouble importing my artists, what could it be?

Imports fail in one of a few ways:

- **The download finished but Lidarr won't import it.** This is almost always a match-quality rejection — Lidarr couldn't find a MusicBrainz release that resembles the file closely enough. See [Import Troubleshooting](/lidarr/import-troubleshooting) for the match-quality rules, the fingerprinting fallback, when manual import is the expected path, and how to drive it.
- **You're importing an existing library and the match fails.** See [Importing an Existing Library](/lidarr/importing-existing-library) for the fresh-install walkthrough and the lenience rules that apply to library imports.
- **Lidarr can't read or write the folder.** The user account Lidarr runs as must have read and write access to both the download folder and the destination root folder. On Linux this is usually a UID/GID/permissions issue on a mount; on Windows this is usually Lidarr running as `LocalService` which can't reach a network share. See [Why can't Lidarr see my files on a remote server](#why-cant-lidarr-see-my-files-on-a-remote-server) for the Windows case.
- **Untagged or badly-tagged files.** Files with generic filenames like `track01.mp3` and no tags give Lidarr nothing to match on. Run a tagger — [MusicBrainz Picard](https://picard.musicbrainz.org/) or [Harmony](https://harmony.pulsewidth.org.uk/) — before importing. See [Import Troubleshooting → Untagged or badly-tagged files](/lidarr/import-troubleshooting#untagged-or-badly-tagged-files).

### How can I rename my artist folders?

{#rename-folders}

> The same process applies for moving or changing artist paths.
{.is-info}

For a handful of artists:

1. Open the artist page.
2. Click the Edit button in the top nav.
3. Change the path (or the root folder, which moves the folder under a different root).
4. Select *Yes, move the files* if prompted.

For a bulk rename:

1. Library → **Select Artists**.
2. Tick the artists that need their folders renamed.
3. Click **Edit**.
4. Set the **Root Folder** to the same root folder those artists already use (this is the trigger that causes Lidarr to re-evaluate each artist's folder name against the current naming settings).
5. Select *Yes, move the files*.

If a rename appears to have happened in Lidarr but the folder name on disk is unchanged, see [Why does Lidarr keep trying to rename the same folders?](#why-does-lidarr-keep-trying-to-rename-the-same-folders) below — that is usually a case-only rename on Windows, which is a filesystem-level no-op.

> **Renaming outside Lidarr breaks the link between Lidarr's database and the files on disk.** Lidarr tracks files by path. If you rename a folder at the OS level, Lidarr will regard the files as missing and may re-download them. Always rename through the Lidarr UI when possible.
{.is-warning}

### Why does Lidarr keep trying to rename the same folders?

Almost always a case-only rename on Windows. Windows filesystems treat `Artist` and `artist` as the same path — so when Lidarr attempts to rename `ARTIST` to `Artist`, the operation reports success but the folder name on disk does not change. Lidarr then sees the folder still needs renaming, and the cycle continues.

The fix is to rename the folder manually using a two-step rename (`Artist` → `Artist_tmp` → `Artist`) so the filesystem actually commits the case change. On Linux and macOS this is not an issue — those filesystems are case-sensitive.

## Lists and automation

### Why are lists sync times so long and can I change it?

List sync is intentionally slow. Lists are a *"add eventually"* tool, not an *"add now"* tool — the original sync cadence caused the upstream Servarr metadata server to be overwhelmed by users running 10-minute list refreshes.

If you need faster feedback for a specific list:

- Trigger a list refresh manually (Settings → Import Lists → the list → **Refresh**).
- Script the refresh via the API — `POST /api/v1/command` with body `{"name": "ImportListSync", "definitionId": <list-id>}` using your Lidarr API key. See [API Docs](https://lidarr.audio/docs/api) for the full command reference. Typical pattern: a cron on your own side that polls the source (Spotify playlist, Last.fm top-tracks, etc.) and pokes Lidarr when something changed.
- Add the releases directly in Lidarr rather than going through a list.

> **Spotify import lists are subject to Spotify's own API rate limits.** Large playlists (thousands of tracks) or many concurrent lists can take time to resolve. If a Spotify list appears to be stuck, check the Lidarr log at the Debug level — you'll see the 429s from Spotify if that's the cause. Waiting out the rate-limit window resolves it.
{.is-info}

### Does Lidarr download lyrics, liner notes, or other extras?

No — by design. Lidarr's job is to fetch the release audio files and tag/organise them; it does not bundle lyrics, liner notes, or other secondary files.

For lyrics, use a tag-aware companion tool:

- [beets](https://beets.io/) — a music tagger and library manager; its `lyrics` plugin fetches lyrics into tag fields during beets import.
- [MusicBrainz Picard](https://picard.musicbrainz.org/) — has community plugins for lyrics and extra tag sources.

These can run alongside Lidarr against the same library; they just need to be sequenced so they don't fight over file ownership.

### What about Custom Scripts / Custom Formats / post-processing?

Lidarr supports two distinct extension points:

- **Custom Scripts** — hook scripts that fire on Lidarr events (on-grab, on-import, on-rename, etc.). Useful for external notifications, custom file copies, or integration with a local library manager. See [Custom Scripts](/lidarr/custom-scripts) for setup and the list of events.
- **Custom Formats** — rules for scoring and filtering releases based on release-name patterns, indexer flags, source, language, etc. Used to prefer (or reject) specific sources, encoders, or quality profiles beyond what the base quality definitions allow. See [Settings → Custom Formats](/lidarr/settings#custom-formats-2).

Custom Formats are scored; Custom Scripts are event-driven. They don't overlap in scope.

## Integrations and external tools

### Does Lidarr support Deemix, slskd, or similar tools?

Deemix and slskd are third-party tools, not Lidarr plugins per se — Deemix is a Deezer downloader, slskd is a Soulseek daemon. Historically there was no built-in way to integrate them with Lidarr, and people would cobble together import scripts.

**This changed with the plugin architecture.** As of the `nightly` branch, Lidarr supports third-party plugins that add indexers and download clients for streaming services, peer-to-peer networks, and other sources. This is the fully-supported way to extend Lidarr's source coverage — not a workaround. See [Plugins](/lidarr/plugins) for install instructions and the current compatibility notes.

Common community plugins at time of writing cover exactly this ground — Soulseek, Deezer, Tidal, and similar — and are linked from the Plugins page. Plugin names and availability shift faster than this FAQ can track, so the Plugins page is the current reference.

Running the plugin branch requires switching to `nightly` (see [How do I update Lidarr?](#how-do-i-update-lidarr)). Database schema migrations mean you cannot easily switch back to `master` or `develop` afterwards without restoring a pre-switch backup.

### Which download clients does Lidarr support?

Usenet (NZB): SABnzbd, NZBGet, NZBVortex, Pneumatic, usenet blackhole.

Torrent: qBittorrent, Transmission, Deluge, rTorrent / ruTorrent, uTorrent, Vuze, Flood, Hadouken, torrent blackhole.

Additional clients are available via [Plugins](/lidarr/plugins) on the `nightly` branch — primarily streaming-service and P2P-network clients not covered by the built-in list.

For setup recipes, the [TRaSH Guides — Downloaders](https://trash-guides.info/Downloaders/) section covers the common clients in more depth than the Lidarr wiki does.

### Does Lidarr integrate with Plex, Emby, or Jellyfin?

Not directly. Lidarr manages the library on disk; media servers read that library and serve it to clients. The common setup pattern is to share the root folder:

1. Lidarr writes files to `/media/music` (or your preferred path).
2. Plex / Emby / Jellyfin have a music library configured against the same `/media/music`.
3. Lidarr fires an on-import / on-rename Custom Script (optional) that pokes the media server's API to refresh the affected folder, avoiding a full library rescan.

There is no Lidarr plugin for any media server. The integration is at the filesystem level, and the Custom Script for library-refresh on import is the only wiring required.

### VPNs, Jackett, and the *ARRs

See the dedicated [VPN Guide](/vpn). Short version: only your BitTorrent client needs to be behind a VPN in most jurisdictions; the *Arr apps themselves should not be, because shared VPN endpoints cause rate limits, captchas, and Cloudflare blocks for metadata lookups.

## Errors and authentication

### Forced Authentication

If Lidarr is reachable from outside your LAN you should have authentication enabled. Trackers and indexers increasingly require it.

As of Lidarr v2, **authentication is mandatory** — `AuthenticationType` and `AuthenticationMethod` are required in the config file.

#### Authentication Method

- `Basic` — browser-native username/password pop-up. Deprecated and will be removed in a future major version.
- `Forms` — login page. Recommended for all UI-exposed installs.
- `External` — disables app authentication entirely. Only for installs behind an external authentication layer (Authelia, Authentik, nginx auth). Configurable via the config file only. Do not use this unless the external auth is actually enforced on the request path.

#### Authentication Required

If the UI only needs auth on remote access (not on LAN), set **Settings → General → Security → Authentication Required** to *Disabled For Local Addresses*. The config-file equivalent is `<AuthenticationType>DisabledForLocalAddresses</AuthenticationType>`. The other valid value is `Enabled`.

### Help I have locked myself out

{#help-i-have-forgotten-my-password}

To reset a forgotten username or password, disable authentication by editing `config.xml` in your [Lidarr Appdata Directory](/lidarr/appdata-directory):

1. Stop Lidarr.
2. Open `config.xml` in a text editor.
3. Find the `<AuthenticationMethod>…</AuthenticationMethod>` line (it will say `Basic` or `Forms`). **If there are two entries in the file, delete both.**
4. Remove the entire line.
5. Start Lidarr.

The UI will now open without a password, and prompt you to set a new password and authentication method on first access.

### I am getting an error: Database disk image is malformed

- **Error says *Error creating log database*** — the corruption is in `logs.db`. Rename or delete that file; it only contains command history and log entries, none of which are load-bearing. Lidarr recreates it on next start.
- **Error says *Error creating main database* or just *database disk image is malformed*** — the corruption is in `lidarr.db`, which is the real database. Do *not* delete it. Follow the recovery steps below.

Your options, in order of increasing effort:

1. **Restore from backup.** See [Tips and Tricks → Backup/Restore](/lidarr/tips-and-tricks#backup-restore). This is always the best first move.
2. **Try the SQLite `.recover` command.** See [Useful Tools → Recovering a corrupt DB](/useful-tools#recovering-a-corrupt-db) for the CLI path, or [Recovering a corrupt DB (GUI)](/useful-tools#recovering-a-corrupt-db-ui) for the Windows / GUI path.
3. **Start fresh.** Delete `lidarr.db` (and the `.db-wal` / `.db-journal` siblings) and let Lidarr recreate from scratch. You will lose your library metadata and have to re-add artists.

Common causes and their preventable forms:

- **Running the database on a network drive (NFS, SMB, or other non-local storage).** SQLite is not designed for this and will corrupt sooner or later — see the [SQLite upstream warning](https://www.sqlite.org/draft/useovernet.html). The AppData folder (Docker: `/config` mount) must be on local storage. This is the single most common root cause.
- **Permissions.** If the Lidarr user cannot write to the database file, SQLite can leave it in a corrupt state. Usually only bites new installs, migrated installs, or systems where the running user/group was recently changed.
- **mergerFS with `direct_io` enabled.** SQLite uses mmap, which mergerFS `direct_io` does not support — see [mergerFS docs](https://github.com/trapexit/mergerfs#plex-doesnt-work-with-mergerfs). Remove `direct_io` from the mergerFS options.

### I use Lidarr on a Mac and it suddenly stopped working. What happened?

Most likely one of the databases is corrupt — a known macOS issue when the system sleeps or crashes during a database write. See [I am getting an error: Database disk image is malformed](#i-am-getting-an-error-database-disk-image-is-malformed) above for recovery.

If the database is not the cause and Lidarr still won't start, post in the [subreddit](https://reddit.com/r/lidarr) or [Discord](https://lidarr.audio/discord) with the logs.

### Why can't Lidarr see my files on a remote server

{#why-can-lidarr-not-see-my-files-on-a-remote-server}

For all operating systems, the user Lidarr runs as must have both read and write access to the remote share.

**Linux mount flags:**

- NFS mounts — ensure `nolock` is set in the mount options.
- SMB mounts — ensure `nobrl` is set in the mount options.

**Windows — the common cause is the service account:**

Lidarr installed as a service runs under `LocalService` by default. `LocalService` does not have access to protected remote file shares, which is almost always why a networked path appears empty or permission-denied in the Lidarr UI.

Two fixes:

1. **Run the Lidarr service as a user that has share access.**
   - Administrative Tools → Services → stop the Lidarr service.
   - Right-click → Properties → Log On tab.
   - Change the service user to a domain or local account with access to the share.
   - Start the service.

2. **Use a UNC path instead of a mapped drive.** Mapped drives are a per-user construct; `LocalService` cannot see `Z:` even if your desktop session can. Configure Lidarr paths as `\\server\share\path` instead, and make sure the share permits access to whichever user the service runs as.
