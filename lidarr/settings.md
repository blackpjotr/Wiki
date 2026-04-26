---
title: Lidarr Settings
description: Complete configuration guide for Lidarr settings including media management, profiles, quality definitions, and metadata preferences
published: true
date: 2026-04-26T16:32:53.365Z
tags: lidarr, settings, configuration, quality, profiles, metadata, media
editor: markdown
dateCreated: 2021-06-14T21:36:07.513Z
---


# Lidarr Settings

This page covers all settings available in Lidarr. For field-level detail on a specific area, jump directly to the relevant section below.

# Media Management

{#media-management}

## Track Naming

### Rename Tracks

When enabled, Lidarr renames imported track files according to the format strings below. When disabled, files are imported using their original filenames — folder structure is still managed by Lidarr, but individual filenames are left as-is.

> Renaming files that are currently being seeded by a torrent client will break seeding unless you are using hardlinks. See [Concepts — Hardlinks](/lidarr/concepts#hardlinks-and-completed-downloads) before enabling this.
{.is-warning}

### Replace Illegal Characters

Replaces characters in filenames that are not valid on the target filesystem (e.g. `: / \ * ? " < > |` on Windows). When disabled, Lidarr will not sanitize filenames and imports may fail on restricted filesystems.

## Naming Format

The format strings below use tokens to build file and folder names. For a full token reference see the [Naming Guide](/lidarr/naming-guide).

> Advanced settings must be enabled (**Settings → Show Advanced**) to reveal the format fields.
{.is-info}

### Standard Track Format

The filename template for regular single-disc track files. The file extension is appended automatically.

Example: `{Album Title}/{track:00} {Track Title}` → `Blood on the Tracks/01 Tangled Up in Blue.flac`

### Multi Disc Track Format

The filename template for tracks on releases with more than one disc. Use `{medium:00}` to include the disc number.

Example: `{Album Title}/{medium:00}-{track:00} {Track Title}` → `Mellon Collie/01-01 Mellon Collie and the Infinite Sadness.flac`

### Artist Folder Format

The folder name template for each artist at the root folder level. Tokens are limited to artist-level fields.

Example: `{Artist Name}` → `/music/The Beatles/`

## Folders

| Setting | Default | Description |
|---|---|---|
| **Create Empty Artist Folders** | Off | Create an artist folder immediately when an artist is added, even before any tracks are imported. |
| **Delete Empty Folders** | Off | Automatically remove empty artist and album folders after files are deleted or moved. |

## Importing

| Setting | Default | Description |
|---|---|---|
| **Skip Free Space Check** | Off | Skip the available disk space check before importing. Only enable this if Lidarr cannot correctly detect free space (some network shares and unusual storage setups). |
| **Minimum Free Space** | 100 MB | Lidarr will refuse to import if available space in the root folder falls below this value. |
| **Use Hardlinks Instead of Copy** | On | Use hardlinks when the source and destination are on the same filesystem. Hardlinks avoid copying data and allow seeding to continue. Falls back to copy if hardlinks are not supported. |
| **Import Extra Files** | Off | Import additional files with the same base name alongside audio files (e.g. cover art, booklets). |
| **Extra File Extensions** | (empty) | Comma-separated list of file extensions to import alongside audio when **Import Extra Files** is enabled. Example: `jpg,png,pdf,cue` |

## File Management

| Setting | Default | Description |
|---|---|---|
| **Unmonitor Deleted Tracks** | Off | When a track file is deleted from disk outside of Lidarr, automatically unmonitor that track. |
| **Download Propers and Repacks** | Prefer and Upgrade | How to handle proper/repack releases. **Prefer and Upgrade** grabs and upgrades to propers when found. **Do not Upgrade Automatically** includes them in scores but won't auto-grab. **Do not Prefer** treats them as equal to the original. |
| **Analyse Audio Files** | On | Read audio file metadata (bitrate, sample rate, bit depth) to improve quality detection. Disabling this makes quality detection rely solely on filename parsing. |
| **Rescan Artist Folder after Refresh** | Always | When to rescan an artist folder after a metadata refresh. **Always** rescans every time. **After Manual Refresh** only rescans when triggered manually. **Never** disables rescanning. |
| **Watch Library for File Changes** | On | Monitor the library folder for external file changes (additions, deletions, renames). Disabling this means Lidarr only discovers changes during scheduled rescans. |

## Permissions

These settings apply to Linux and macOS only. Leave disabled on Windows.

| Setting | Default | Description |
|---|---|---|
| **Set Permissions** | Off | Set file and folder permissions on imported files. |
| **chmod Folder** | 755 | Octal permission mode applied to folders on import (e.g. `755` = rwxr-xr-x). |
| **chown Group** | (empty) | Group to assign to imported files and folders. The Lidarr process user must be a member of this group. |

## Recycling Bin

| Setting | Default | Description |
|---|---|---|
| **Recycle Bin** | (empty) | Path to a recycling bin folder. When Lidarr deletes files, they are moved here rather than permanently deleted. Leave empty to skip the recycle bin. |
| **Recycle Bin Cleanup** | 7 days | Number of days before files in the recycle bin are permanently deleted. Set to `0` to disable automatic cleanup. |

## Root Folders

Root folders are the top-level directories where Lidarr stores your library. Each imported artist gets a subfolder inside a root folder.

Click **Add (+)** to add a root folder. The path must exist and Lidarr must have read and write access to it. A root folder must not overlap with your download client's output directory.

> Do not point a root folder at a cloud storage mount (Dropbox, OneDrive, Google Drive). Lidarr writes audio tags and metadata frequently; cloud storage APIs have rate limits that will cause failures.
{.is-warning}


## Metadata Profiles

{#metadata-profiles}

Metadata profiles control which release types are visible per artist. Releases whose type is not included in the assigned profile are hidden and will not be monitored or searched.

### Fields

| Field | Description |
|---|---|
| **Name** | Profile name. |
| **Release Types** | Checkboxes for each MusicBrainz release type. Only releases matching a checked type appear for artists using this profile. |

### Release Types

| Type | Description |
|---|---|
| **Albums** | Standard studio albums. |
| **Singles** | Single releases (one or a few tracks). |
| **EPs** | Extended plays — longer than a single, shorter than an album. |
| **Broadcasts** | Live broadcasts, radio sessions, and similar. |
| **Others** | Release types not covered by the above categories. |

Secondary types (Compilation, Soundtrack, Spokenword, Interview, Live, Remix, DJ-Mix, Mixtape, Demo, etc.) can be included or excluded as additional filters on top of the primary type selection.

> Release types are determined by MusicBrainz. If a release you expect to see is missing, check its entry on MusicBrainz — the type may be set to `Unknown`, which Lidarr cannot filter on, or the primary type may be one you have unchecked in your profile.
{.is-info}


## Release Profiles

{#release-profiles}

Release profiles filter and score releases based on their titles. Use them to require certain terms, reject others, or shift scoring without creating a full custom format.

### Fields

| Field | Description |
|---|---|
| **Must Contain** | Comma-separated list of terms (or regex patterns) that a release title must include. Releases not matching are rejected entirely. |
| **Must Not Contain** | Comma-separated list of terms a release title must not include. Releases matching any term are rejected. |
| **Preferred** | Terms with associated scores. Positive scores boost a release; negative scores penalise it. Multiple terms can share a score by separating them with commas. |
| **Include Preferred when Renaming** | If enabled, the matched preferred term is appended to the filename during rename. Useful for tagging releases from specific groups in the filename. |
| **Indexers** | Restrict this profile to specific indexers. Leave empty to apply to all indexers. |
| **Tags** | Restrict this profile to artists with matching tags. Leave empty to apply to all artists. |


# Quality

{#quality}

The Quality page defines size thresholds for each quality level. Lidarr uses these to validate that a release's reported size is consistent with the claimed quality, catching mislabelled releases.

For audio, size limits are expressed in **kilobits per second (kbps)** — Lidarr computes a bitrate from the file size and duration and compares it to the configured range.

| Column | Description |
|---|---|
| **Quality** | The quality name (e.g. FLAC, MP3-320, MP3-256). |
| **Min** | Minimum acceptable bitrate in kbps. Releases below this are rejected. Set to `0` to disable the lower bound. |
| **Preferred** | The bitrate Lidarr aims for when scoring releases. Releases at this level receive a higher score than those at the minimum. |
| **Max** | Maximum acceptable bitrate in kbps. Releases above this are rejected. Set to `0` for no upper limit. |

> FLAC is lossless and does not have a consistent bitrate — its effective bitrate varies by content. The FLAC entry in quality definitions is used primarily for minimum/maximum file-size sanity checks rather than strict bitrate enforcement.
{.is-info}


# Download Clients

{#download-clients}

> Information on supported download clients can be found at the [Supported](/lidarr/supported#download-clients) page.
{.is-info}

## Overview

Lidarr sends download requests to a configured client, monitors the client's queue via its API, and imports finished files into the library. The download client and Lidarr must both be able to read and write to the same filesystem path — mismatched paths are the most common cause of import failures.

## How Downloading Works

### Usenet

1. Lidarr sends a download request to the Usenet client with a configured category label.
2. Lidarr monitors the client's queue via its API for items in that category.
3. When the download completes, Lidarr reads the reported file path, scans it for audio files, and imports them into the library.
4. Atomic moves (instantaneous moves within the same filesystem) are used by default. If the download folder and library folder are on different filesystems, Lidarr falls back to copy + delete, which uses more I/O and temporary disk space.

### BitTorrent

1. Lidarr sends a download request to the torrent client with a category label.
2. Lidarr monitors the client's queue for items in that category.
3. When the download completes, Lidarr imports the files. If the download folder and library folder are on the same filesystem, a **hardlink** is created — the file appears in both locations without using extra disk space, and seeding continues uninterrupted.
4. The torrent client retains the original files so seeding can continue. Lidarr will request removal only after seeding is complete (when **Remove** is enabled and the seed goal is met).

> The download folder and library root folder must be on the **same filesystem** for hardlinks to work. In Docker, both must be mounted through the same volume or bind mount. See [Concepts — Hardlinks](/lidarr/concepts#hardlinks-and-completed-downloads) and [TRaSH's Hardlink Guide](https://trash-guides.info/hardlinks) for setup details.
{.is-info}

## Download Client Settings

Click **Add (+)**, choose a client type, and fill in the connection details.

### Common Fields (all clients)

| Field | Description |
|---|---|
| **Name** | Label for this client, shown in activity and logs. |
| **Enable** | Whether Lidarr actively monitors this client. |
| **Host** | Hostname or IP of the download client. |
| **Port** | Port the client's web UI or API listens on. |
| **Use SSL** | Connect to the client over HTTPS. |
| **(Advanced) URL Base** | Path prefix for the client, needed when behind a reverse proxy (e.g. `/sabnzbd`). |
| **Username / Password** | Credentials if the client requires authentication. |
| **Category** | Category or label to apply to Lidarr's downloads. Lidarr only monitors items in this category — setting one is strongly recommended to avoid conflicts with other applications sharing the same client. |
| **Recent Priority** | Priority level for recently-released music. |
| **Older Priority** | Priority level for back-catalogue music. |
| **(Advanced) Client Priority** | Order among multiple clients of the same type. `1` = highest priority; `50` = lowest. Round-robin is used among equal-priority clients. |

### Usenet-Only Fields

| Field | Description |
|---|---|
| **API Key** | API key to authenticate with the Usenet client. |

### Torrent-Only Fields

| Field | Description |
|---|---|
| **Post-Import Category** | Category to assign after import. Note: setting this disables completed-download removal, since Lidarr can no longer identify the torrent as one it manages. |
| **Initial State** | Whether new torrents start paused or downloading immediately. |

### Torrent Client Seed Goal Compatibility

Lidarr can set seed ratio and time goals via the torrent client's API when a torrent is added, but not all clients support this.

| Client | Seed Ratio | Seed Time |
| :---: | :---: | :---: |
| Deluge | ✓ | ✗ |
| Download Station | ✗ | ✗ |
| Flood | ✓ | ✓ |
| Hadouken | ✗ | ✗ |
| qBittorrent | ✓ | ✓ |
| rTorrent | ✓ | ✓ |
| Torrent Blackhole | ✗ | ✗ |
| Transmission | ✓ | Idle Limit only |
| uTorrent | ✓ | ✓ |
| Vuze | ✓ | ✓ |

## Completed Download Handling

| Setting | Description |
|---|---|
| **Enable** (Advanced, global) | Automatically import completed downloads from the download client. Disabling this means Lidarr will never import anything — leave enabled unless you have a specific reason to disable it. |
| **Remove** (per-client) | After import, ask the download client to remove the completed item. For torrents, removal only occurs when the client reports seeding is complete and the torrent is paused/stopped. |

### Failed Download Handling

Failed download handling is available for SABnzbd and NZBGet only. It is not supported for torrent clients.

| Setting | Description |
|---|---|
| **Redownload** | When a download fails, automatically search for a replacement. |
| **(Advanced) Remove** | Remove the failed download from the client when the failure is detected. |

When a failure is detected, Lidarr logs it, optionally removes the failed item, searches for a replacement, and blocklists the failed release so it is not grabbed again automatically.

## Remote Path Mappings

Remote path mappings are needed when Lidarr and the download client see the same files at different paths — for example, when they run on separate machines or in separate Docker containers with different volume mount paths.

A mapping translates a remote path (as reported by the download client) to a local path (as Lidarr accesses it). Add a mapping per client under **Settings → Download Clients → Remote Path Mappings**.

> If both Lidarr and the download client are in Docker containers on the same host with matching volume mounts, a remote path mapping is usually not needed. See [TRaSH's Remote Path Mapping guide](https://trash-guides.info/Radarr/Radarr-remote-path-mapping/) for diagnosis and setup.
{.is-info}


# Connect

{#connections}

> Information on supported connection types can be found at the [Supported](/lidarr/supported#notifications) page.
{.is-info}

Connections send notifications or trigger actions when events occur in Lidarr. Common uses include Discord or Slack notifications on import, Plex library updates, and custom scripts.

Click **Add (+)** and select a connection type. Most connections share these fields:

| Field | Description |
|---|---|
| **Name** | Label for this connection. |
| **On Grab** | Trigger when Lidarr sends a release to a download client. |
| **On Release Import** | Trigger when a downloaded release is successfully imported. |
| **On Upgrade** | Trigger when a file is upgraded to a better quality. |
| **On Rename** | Trigger when files are renamed. |
| **On Artist Added** | Trigger when an artist is added to Lidarr. |
| **On Artist Deleted** | Trigger when an artist is removed. |
| **On Album Delete** | Trigger when an album is deleted. |
| **On Track Retag** | Trigger when audio tags are rewritten. |
| **On Health Issue** | Trigger when a health check fails. |
| **On Health Restored** | Trigger when a health check recovers. |
| **On Application Update** | Trigger when Lidarr updates to a new version. |
| **Include Health Warnings** | Include `Warning`-level health issues in health notifications (not just `Error`). |

For **Custom Script** connections, see the [Custom Scripts](/lidarr/custom-scripts) page for the full list of environment variables available per event.


# Tags

{#tags}

Tags link artists, indexers, delay profiles, and release profiles together. A tag on an artist and a matching tag on a delay profile means that delay profile applies to that artist. Without a matching tag, the default (untagged) delay profile applies.

Tags are particularly useful for:

- Assigning a specific indexer to specific artists (e.g. a private tracker that only carries jazz — tag it `jazz` and tag the relevant artists `jazz`).
- Assigning a non-default delay profile to a subset of artists.
- Restricting a release profile to certain artists.
- Tracking which import list added an artist.

> Tags do not affect quality profiles or metadata profiles. Those are assigned directly to each artist.
{.is-info}


# UI

{#ui}

## Calendar

| Setting | Description |
|---|---|
| **First Day of Week** | Day the week starts on in the calendar view. |
| **Week Column Header** | Date format shown in the calendar's week-column headers. |

## Dates

| Setting | Description |
|---|---|
| **Short Date Format** | Format used for dates in compact contexts (e.g. album lists). |
| **Long Date Format** | Format used for dates in expanded contexts. |
| **Time Format** | 12-hour (`1:30 PM`) or 24-hour (`13:30`) time display. |
| **Show Relative Dates** | Show dates as "3 days ago" or "in 2 weeks" instead of absolute dates. |

## Style

| Setting | Description |
|---|---|
| **Enable Color Impaired Mode** | Adjust the UI color palette for better visibility with common color vision deficiencies. |
| **Expand Albums by Default** | Expand the album list on artist pages by default instead of showing collapsed view. Separate toggles are available for Albums, EPs, Singles, Broadcasts, and Other release types. |

## Language

| Setting | Description |
|---|---|
| **UI Language** | Language for the Lidarr interface. Translations are community-maintained; coverage varies. |
