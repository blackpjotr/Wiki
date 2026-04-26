---
title: Davo's Community Lidarr Guide
description: Like TRaSH Guides, but Davo for Lidarr
published: true
date: 2026-04-26T21:45:04.421Z
tags: lidarr, guide, configuration, quality
editor: markdown
dateCreated: 2024-02-27T14:10:01.585Z
---


# Lidarr Tips and Tricks

Recipes and workarounds for things that come up often enough to be worth documenting, but don't fit the install / troubleshoot / FAQ shape. Most of this is library-management content: filling in gaps in metadata, bulk-editing the library, and backup/restore hygiene.

For setup recipes (reverse proxy, VPN, Docker compose, hardlinks) see the [TRaSH Guides](https://trash-guides.info/) — they cover those topics in more depth than this wiki does.

## Folder structure

### Separating download folders from your library

Lidarr requires that your download folder and your music library root folder are **not the same location**. This is not just a best practice — mixing them causes imports to fail or loop, since Lidarr cannot reliably tell what is a finished download and what is already part of the library.

A clean layout keeps three locations separate:

```
/data/
  downloads/          ← download client writes here
  music/              ← Lidarr root folder (your library)
```

Lidarr moves (or hardlinks) files from `downloads/` into `music/` on import. Once imported, the file in `downloads/` is the copy the download client owns; the file in `music/` is the copy Lidarr manages.

> For hardlinks to work — which avoids any extra disk space during import — both paths must be on the same physical filesystem. If `downloads/` and `music/` are on different drives or volumes, Lidarr will copy instead of hardlink. See [Concepts → Hardlinks and completed downloads](/lidarr/concepts#hardlinks-and-completed-downloads).
{.is-info}

### Multiple download clients

If you run more than one download client (for example, both a Usenet client and a torrent client), give each its own download subfolder **and** configure a matching category in Lidarr:

```
/data/
  downloads/
    usenet/           ← SABnzbd / NZBGet category: "lidarr"
    torrents/         ← qBittorrent / Deluge category: "lidarr"
  music/
```

In **Settings → Download Clients**, set each client's **Category** to a unique value (e.g., `lidarr`) and set the **Download Client**'s own category to match. Lidarr only monitors items in the configured category — anything outside it is invisible, which prevents cross-contamination between clients and between applications sharing the same client.

> Radarr, Sonarr, and Lidarr can share the same download client safely as long as each application uses a different category. Never point two applications at the same category — they will fight over each other's downloads.
{.is-info}

### Remote path mapping

If Lidarr and your download client run in separate containers or on separate machines, the path the download client reports for a completed download may differ from the path Lidarr sees when it tries to access the same file. Use **Settings → Download Clients → Remote Path Mappings** to translate between the two. See [Troubleshooting → Remote Path Mapping](/lidarr/troubleshooting#remote-path-mapping) for the full explanation.

## Library maintenance

### Missing artist images

Artist images in Lidarr are pulled from whatever the Servarr metadata server aggregates from its upstream sources. Lidarr itself does not reach out to any specific image service — it renders whatever URLs the metadata server returns.

If an artist is showing no image or an obviously-wrong image:

1. Check the artist on [MusicBrainz](https://musicbrainz.org/). If the MB entity has no image linked, there is no image for Lidarr to render. MB accepts community contributions for artist and album artwork; see MB's [How to Contribute](https://musicbrainz.org/doc/How_to_Contribute) page.
2. Wait for the metadata refresh to propagate. Lidarr's copy of metadata refreshes hourly from the Servarr metadata server; the metadata server's own cache has its own propagation window measured in hours, sometimes longer.
3. Trigger an artist refresh in Lidarr once you believe upstream has updated (Artist page → Refresh button, or Library → Artist Editor → Update).

> Lidarr has no "upload an image locally" option. Everything it displays has to exist upstream. This is the same model as the Release / Release Artist data itself — see [Concepts](/lidarr/concepts).
{.is-info}

### Missing album images

{#missing-album-images}

Album cover art is sourced from MusicBrainz's [Cover Art Archive](https://coverartarchive.org/) via the Servarr metadata server.

1. Open the release group on MusicBrainz and check whether cover art is present in the Cover Art Archive.
2. If not, you can upload a cover yourself — MusicBrainz has a [Cover Art guideline](https://musicbrainz.org/doc/Cover_Art) covering acceptable sources and quality.
3. After upload, allow roughly an hour for the metadata server cache to pick it up, then refresh the album in Lidarr.

Covers uploaded directly to the Cover Art Archive are available to Lidarr faster than other metadata corrections, because CAA has a more direct path into the metadata server than the MusicBrainz entity data does.

### Mass delete artists

Use **Library → Artist Editor** (formerly "Mass Editor"):

1. Select the artists to remove (Shift-click for ranges; Ctrl/Cmd-click for individual additions).
2. Click **Delete** at the bottom of the screen.
3. Choose whether to also delete files on disk.
   - *Remove from Lidarr only* keeps your files; use this when the artist was added in error but you want to keep the music.
   - *Remove from Lidarr and delete files* is destructive and irreversible. Make sure you have a backup first if in doubt.

The same flow works for bulk changes that aren't deletion — root folder moves, quality profile swaps, monitoring toggles — via the other bulk-action buttons at the bottom of Artist Editor.

## Custom Formats

Custom formats let you score releases by source, release group, and other title characteristics so Lidarr can prefer or avoid them automatically. The examples below are a useful starting point for a FLAC-focused library. Import each block via **Settings → Custom Formats → Import**.

### Quality definitions for FLAC

Before setting up custom format scoring, consider tightening the FLAC quality definition to filter out single-file CUE+FLAC rips, which present as a single very large track rather than properly split files.

In **Settings → Quality**, set the FLAC row to:

| | Min | Preferred | Max |
|---|---|---|---|
| FLAC | 0 | 895 | 1400 |
| FLAC 24bit | 0 | 895 | 1495 |

The Max value rejects releases whose computed bitrate exceeds a realistic ceiling for split-track FLAC, which catches most CUE+single-file rips without affecting normal releases.

### Example custom formats

#### Preferred Groups

Boosts releases from groups known for consistent quality and accurate tagging.

```json
{
  "name": "Preferred Groups",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "DeVOiD",
      "implementation": "ReleaseGroupSpecification",
      "negate": false,
      "required": false,
      "fields": { "value": "\\bDeVOiD\\b" }
    },
    {
      "name": "PERFECT",
      "implementation": "ReleaseGroupSpecification",
      "negate": false,
      "required": false,
      "fields": { "value": "\\bPERFECT\\b" }
    },
    {
      "name": "ENRiCH",
      "implementation": "ReleaseGroupSpecification",
      "negate": false,
      "required": false,
      "fields": { "value": "\\bENRiCH\\b" }
    }
  ]
}
```

#### CD

Tags releases identified as a CD source.

```json
{
  "name": "CD",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "CD",
      "implementation": "ReleaseTitleSpecification",
      "negate": false,
      "required": false,
      "fields": { "value": "\\bCD\\b" }
    }
  ]
}
```

#### WEB

Tags releases identified as a web (streaming) source.

```json
{
  "name": "WEB",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "WEB",
      "implementation": "ReleaseTitleSpecification",
