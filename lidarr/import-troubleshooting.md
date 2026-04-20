---
title: Import Troubleshooting
description: Why a download finishes but Lidarr does not import it — match-quality thresholds, the manual-import path, and when human intervention is necessary
published: true
date: 2026-04-20T13:06:15.307Z
tags: lidarr, troubleshooting, plugins, import, matching
editor: markdown
dateCreated: 2026-04-20T13:06:15.307Z
---

# Import Troubleshooting

> The download finished, but Lidarr will not import it. This page covers the match-quality rules Lidarr applies to imports, why an otherwise-valid file can still be rejected, when human intervention is required, and how to use manual import to force an outcome when the matcher cannot.
{.is-info}

If you have not imported anything yet and are setting up a fresh library, start with [Importing an Existing Library](/lidarr/importing-existing-library) instead — this page is for diagnosing imports that do not auto-complete, not for the first-time setup.

## Why Lidarr rejects an import

Every file Lidarr considers importing runs through a decision pipeline. For an import to succeed, the file must *pass every specification*. A single rejection reason anywhere in the pipeline stops the import. The pipeline is strict by design: incorrect imports are hard to undo, so Lidarr errs on the side of refusing rather than guessing.

The most common rejection category is **match quality** — Lidarr could not find a release that resembles the file closely enough.

### The match-quality thresholds

Lidarr computes a normalised *distance* score between what it sees on disk (file tags, filenames, duration, track count) and each candidate release in its metadata. Lower distance = closer match. The score is converted to a similarity percentage for user-facing messages.

For a **new download**, Lidarr requires:

- **Album overall:** at least 80% similarity (distance ≤ 0.20).
- **Worst track within the album:** at least 60% similarity (distance ≤ 0.40).

If the album scores 85% overall but one track inside it only matches at 50%, the whole import is rejected — that one poor track is enough to fail the album.

For **importing existing library files** (when you point Lidarr at a root folder you already had on disk), the rules are more lenient:

- **Album overall:** still 80% — but `missing_tracks` and `unmatched_tracks` are excluded from the score. This is why an existing library can import successfully even if some files are missing or there are extras.

Standalone track imports outside the album context use a **60% similarity** threshold (distance ≤ 0.40).

> These are not configuration values — they are constants in the Lidarr source code. See [Where this lives in the source](#where-this-lives-in-the-source) below.
{.is-info}

### What goes into the distance score

The distance score is a weighted combination of how well each field matches. The weights are adapted from [beets' default configuration](https://beets.readthedocs.io/), with MusicBrainz IDs treated as the strongest signals:

| Signal | Weight | Notes |
|---|---|---|
| `recording_id` | 10.0 | MusicBrainz Recording MBID. If present and correct, imports almost always succeed. |
| `album_id` | 5.0 | MusicBrainz Release MBID. |
| `artist` | 3.0 | Artist name (Levenshtein distance after normalisation). |
| `album` | 3.0 | Album/release title (same). |
| `track_title` | 3.0 | Per-track title. |
| `track_artist` | 2.0 | Per-track artist (matters for compilations). |
| `track_length` | 2.0 | Duration in seconds. |
| `tracks` | 2.0 | Track-count agreement. |
| `source` | 2.0 | Source medium (digital / CD / vinyl / etc). |
| `media_count`, `media_format`, `year`, `track_index` | 1.0 each | |
| `unmatched_tracks` | 0.9 | Penalty for files that don't map to any MB track. |
| `missing_tracks` | 0.6 | Penalty for MB tracks with no corresponding file. |
| `country`, `label`, `catalog_number`, `album_disambiguation` | 0.5 each | Tiebreakers, not primary signals. |

The practical takeaway: **MusicBrainz Recording IDs and Release IDs are the dominant signals**. Files tagged with correct MBIDs (by MusicBrainz Picard, for example) almost always import. Files without MBIDs rely on artist/album/title matching and are far more sensitive to tag quality.

## The fingerprinting fallback

If the best candidate release is still worse than threshold, Lidarr can attempt to identify the files by acoustic fingerprint instead of tags. This is the fallback path that sometimes rescues imports where the tags are wrong or missing.

Fingerprinting kicks in when any of the following is true:

- Album distance is worse than 0.15 (below ~85% match).
- There are extra local files that do not map to MB tracks.
- There are MB tracks with no corresponding file.
- The worst track's distance is worse than 0.40 (below 60% match).

Whether fingerprinting is *allowed* is controlled by the **Allow Fingerprinting** setting under Settings → Media Management:

- `Never` — fingerprinting is disabled.
- `New Files` — fingerprint new downloads only (default on most installs).
- `Always` — fingerprint even on library imports.

Fingerprinting is slower than tag-based matching and hits an external service. If downloads regularly land in a state where fingerprinting runs but does not rescue them, the root cause is usually that the files do not exist in MusicBrainz at all — no fingerprint match is possible for a release that is not in the database.

## When human intervention is required

Some situations cannot be resolved by the automatic matcher, no matter how the settings are tuned. These are the cases where manual import is the expected path, not a workaround.

### The release is not in MusicBrainz

Lidarr only imports against releases that exist at MusicBrainz (via the metadata server). If the specific pressing or edition you have is not in MusicBrainz, the matcher has nothing to match *to*. Two options:

- Use **manual import** and point Lidarr at an existing release in the same release group (a different pressing). The files will be attached to that release — the metadata shown in Lidarr will match the attached release, not what is on disk.
- Add the missing release to MusicBrainz (see [Metadata Troubleshooting → Updating MusicBrainz](/lidarr/metadata-troubleshooting#updating-musicbrainz)) and re-import once it propagates.

### The release exists but is missing tracks

Some MB releases are catalogued with incomplete track lists. If your files have 12 tracks and the MB release only lists 10, `unmatched_tracks` applies a significant penalty (weight 0.9) and the import can fail even with otherwise-good metadata. The fix is either to complete the MB release or to manually import.

### Untagged or badly-tagged files

Without tags, Lidarr matches on the filename and duration only. This usually works for well-named files (`01 - Artist - Album - Track.flac`) but breaks on generic filenames (`track01.mp3`). Run a tagger — [MusicBrainz Picard](https://picard.musicbrainz.org/) is the reference tool — before asking Lidarr to import.

### Plugin-added content types

The nightly branch supports third-party plugins for indexers and download clients — streaming services, peer-to-peer networks, and other sources. See [Plugins](/lidarr/plugins) for the install path and current compatibility notes.

Content pulled through plugins sometimes does not fit Lidarr's core import model cleanly: single tracks, partial releases, streaming-service rips, and per-track purchases are all common plugin outputs that the core matcher was not originally designed to handle. The tag data may be correct for the source but incomplete against a MusicBrainz release group, and the automatic matcher will not resolve it. Manual import is the expected path for these cases until the plugin-specific import flows mature — it is not a workaround, it is the intended workflow.

## Import list items: how a list entry becomes an album

Import lists (Spotify, Last.fm, Trakt, etc.) feed entries into Lidarr that have to be resolved to a MusicBrainz release group before anything can be added. The resolution depends on whether the list provides an MBID:

- **If the list item has a MusicBrainz album (release group) ID**, Lidarr does a direct lookup by that ID. No ambiguity — the ID selects exactly one release group.
- **If the list item is only a title string** (common with Spotify and Last.fm, which don't expose MBIDs), Lidarr asks the Servarr metadata server to search for an album matching that title and artist, and takes **whichever result the server returns first**. There is no client-side re-ranking. The metadata server decides the ordering.

This matters because MusicBrainz distinguishes singles from albums at the release-group level — a track called *Blinding Lights* exists on both the "Blinding Lights" single release group and the *After Hours* album release group. If the metadata server's search happens to rank the single higher than the album (track-count match, exact title match, locale, or any other factor the server uses), the list entry resolves to the single, not the album.

Practical consequences:

- A track you expect to see under its album may get added under the single's release group instead.
- Lidarr is not designed to manage a library of single-track release groups; a library that has drifted into lots of singles is hard to reason about later.
- If you want a specific album to be the canonical entry, add it directly from the artist's discography in Lidarr rather than relying on a list's title-based resolution. The discography view shows release groups unambiguously.

There is no user-facing setting to bias the metadata server's ordering — it is an upstream decision.

## Using manual import

Manual import is the override path when the automatic matcher cannot resolve a file. Two entry points:

- **Activity → Queue →** the download appears there with a reason. From the row, choose **Manual Import**.
- **Wanted → Missing → Manual Import** and point it at the folder on disk.

The manual-import dialog lets you pick:

- The artist (if the matcher got the artist wrong).
- The album (if the matcher picked the wrong release group).
- The specific release within the release group (pressing, format, region).
- Per-track mappings when the dialog cannot auto-align tracks.

Manual import **bypasses the specification pipeline entirely**. When you click Import, Lidarr does not re-run match-quality, free-space, not-currently-unpacking, upgrade-or-not, or any other specification — the decisions are treated as approved by user choice. The rejections shown inside the manual-import dialog are informational, to help you decide what to import; they are not blockers once you commit.

The only checks still enforced at manual-import time are filesystem- and DB-level:

- The target artist's folder must sit under a configured root folder. If Lidarr cannot find the destination path inside any root folder, the file is rejected with *Destination artist folder X is not in a Root Folder*.
- The track must not already be imported (per-run dedup across the batch).
- The destination path must not already exist on disk (otherwise: *Failed to import track, Destination already exists*).
- Filesystem permissions must allow the move or copy.
- The artist and album must exist in the DB or be addable from MusicBrainz. If the MBID the dialog selected cannot be resolved or added, the import fails with *Failed to add missing album* / *Failed to add missing artist*.

If manual import fails, the reason will be one of the above — a filesystem or DB failure, not a match-quality rejection.

## Where this lives in the source

For advanced readers who want to verify the rules or trace a specific rejection, the relevant code lives under [`src/NzbDrone.Core/MediaFiles/TrackImport/`](https://github.com/Lidarr/Lidarr/tree/develop/src/NzbDrone.Core/MediaFiles/TrackImport) in the Lidarr source tree:

- **Thresholds:**
  - [`Specifications/CloseAlbumMatchSpecification.cs`](https://github.com/Lidarr/Lidarr/blob/develop/src/NzbDrone.Core/MediaFiles/TrackImport/Specifications/CloseAlbumMatchSpecification.cs) — album-level and worst-track thresholds (0.20 and 0.40).
  - [`Specifications/CloseTrackMatchSpecification.cs`](https://github.com/Lidarr/Lidarr/blob/develop/src/NzbDrone.Core/MediaFiles/TrackImport/Specifications/CloseTrackMatchSpecification.cs) — standalone track threshold (0.40).
- **Scoring model:**
  - [`Identification/Distance.cs`](https://github.com/Lidarr/Lidarr/blob/develop/src/NzbDrone.Core/MediaFiles/TrackImport/Identification/Distance.cs) — weighted field contributions; comment at the top notes the weights are adapted from beets' defaults.
- **Identification flow and fingerprinting:**
  - [`Identification/IdentificationService.cs`](https://github.com/Lidarr/Lidarr/blob/develop/src/NzbDrone.Core/MediaFiles/TrackImport/Identification/IdentificationService.cs) — `Identify()` implements the four-step flow (group → candidates → best match → fingerprint if worse than threshold); `ShouldFingerprint()` documents when fingerprinting triggers.
- **Other specifications that can also reject an automatic import** (these do *not* run for manual import):
  - `AlreadyImportedSpecification.cs`, `AlbumUpgradeSpecification.cs`, `FreeSpaceSpecification.cs`, `MoreTracksSpecification.cs`, `NoMissingOrUnmatchedTracksSpecification.cs`, `NotUnpackingSpecification.cs`, `ReleaseWantedSpecification.cs`, `SameTracksImportSpecification.cs`, `UpgradeSpecification.cs`.
- **Manual-import flow:**
  - [`Manual/ManualImportService.cs`](https://github.com/Lidarr/Lidarr/blob/develop/src/NzbDrone.Core/MediaFiles/TrackImport/Manual/ManualImportService.cs) — `Execute()` builds `LocalTrack` objects from user choices and creates approved `ImportDecision`s directly, bypassing the specification pipeline. The only pre-import rejection is the root-folder check.
  - [`ImportApprovedTracks.cs`](https://github.com/Lidarr/Lidarr/blob/develop/src/NzbDrone.Core/MediaFiles/TrackImport/ImportApprovedTracks.cs) — the post-approval path. Handles the remaining checks (dedup, filesystem, artist/album add) for both automatic and manual imports.
- **Import-list resolution:**
  - [`src/NzbDrone.Core/ImportLists/ImportListSyncService.cs`](https://github.com/Lidarr/Lidarr/blob/develop/src/NzbDrone.Core/ImportLists/ImportListSyncService.cs) — `MapAlbumReport()` takes `.FirstOrDefault()` from the metadata-server search for title-only list entries. Direct MBID lookups go through `SkyHookProxy.SearchForNewAlbum` with a `lidarr:<mbid>` query.
- **Tests:** fixtures under [`src/NzbDrone.Core.Test/MediaFiles/TrackImport/`](https://github.com/Lidarr/Lidarr/tree/develop/src/NzbDrone.Core.Test/MediaFiles/TrackImport) — `IdentificationServiceFixture.cs`, `AlbumDistanceFixture.cs`, `TrackDistanceFixture.cs`, and `ImportDecisionMakerFixture.cs` are the most useful starting points for understanding how specific cases score.

## See also

- [Importing an Existing Library](/lidarr/importing-existing-library) — the fresh-install import walkthrough
- [Metadata Troubleshooting](/lidarr/metadata-troubleshooting) — for problems where the release is missing or wrong at MusicBrainz
- [FAQ](/lidarr/faq) — shorter answers that didn't fit this page
- [MusicBrainz Picard](https://picard.musicbrainz.org/) — tag files with MBIDs before importing for a dramatically higher success rate
