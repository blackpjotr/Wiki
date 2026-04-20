---
title: Lidarr Tips and Tricks
description: Advanced tips, optimization techniques, and workflow improvements for experienced Lidarr users
published: true
date: 2026-04-20T13:09:31.684Z
tags: lidarr, tips, tricks, optimization, workflow, advanced, advanced tips
editor: markdown
dateCreated: 2021-08-14T15:15:51.656Z
---

# Lidarr Tips and Tricks

Recipes and workarounds for things that come up often enough to be worth documenting, but don't fit the install / troubleshoot / FAQ shape. Most of this is library-management content: filling in gaps in metadata, bulk-editing the library, and backup/restore hygiene.

For setup recipes (reverse proxy, VPN, Docker compose, hardlinks) see the [TRaSH Guides](https://trash-guides.info/) — they cover those topics in more depth than this wiki does.

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

## Backup and restore

{#backup-restore}

Lidarr's entire state lives in its [AppData directory](/lidarr/appdata-directory). Backing up is a matter of capturing that directory at rest; restoring is a matter of putting it back in place with matching permissions and paths. There is nothing in the database that is meaningful without the matching `config.xml` and vice-versa — back up the whole folder, not individual files.

### Backing up

Two ways to do it. Use whichever fits your workflow.

**Built-in backup (zip):**

1. System → Backup in the Lidarr UI.
2. Click **Backup**.
3. Download the zip that appears in the backup list.

This captures the database, config, and the minimum needed to restore. The backup excludes the logs folder and the MediaCover cache, both of which are regenerable.

**Filesystem copy:**

1. Stop Lidarr. This is the only way to guarantee the database is in a consistent state — SQLite's WAL can leave a running database in a state that is safe for the running process but not safe to copy.
2. Copy the entire AppData directory to a safe location. Include `.db-wal` and `.db-journal` siblings of `lidarr.db` if they exist.
3. Start Lidarr.

Filesystem backup is the right choice if you want to use existing backup tooling (Borg, restic, Duplicati, ZFS snapshots, etc.) — don't try to snapshot a live SQLite file.

### Restoring

> **Cross-OS restores are not supported.** Windows ↔ Linux and Windows ↔ macOS will not work because the path separators differ. Linux ↔ macOS may work since both use `/`, but is not officially supported. If you need to move between OSes, expect to edit every path in the database by hand.
{.is-warning}

**From a built-in zip backup:**

1. Install Lidarr if it isn't installed yet.
2. Start Lidarr.
3. System → Backup → **Restore Backup**.
4. Choose the zip file.
5. Click **Restore**.

**From a filesystem copy:**

1. Install Lidarr if needed and start it once so the AppData directory gets created in the expected place.
2. Stop Lidarr.
3. Delete the contents of the AppData directory, including any `.db-wal` / `.db-journal` files.
4. Copy your backup into place.
5. Start Lidarr.

As long as the root folder paths in the backup are still valid on the destination machine, Lidarr will pick up where it left off.

### Restore on Synology NAS

> CAUTION: this procedure requires root SSH access and is unnecessary on most systems. Use it only if the standard filesystem restore fails due to Synology's package-user permissions model.
{.is-warning}

1. Install Lidarr via the SPK package if it isn't installed.
2. Find the AppData directory — usually `/usr/local/Lidarr/var/.config/Lidarr/`.
3. Stop Lidarr.
4. SSH to the NAS as root.
5. Replace the database and restore:

   ```shell
   rm -r /usr/local/Lidarr/var/.config/Lidarr/Lidarr.db
   cp -f /tmp/Lidarr_backup/* /usr/local/Lidarr/var/.config/Lidarr/
   ```

6. Fix ownership and permissions:

   ```shell
   cd /usr/local/Lidarr/var/.config/Lidarr/
   chown -R Lidarr:users *
   chmod -R 0644 *
   ```

   > On some SynoCommunity versions the user is `sc-Lidarr` rather than `Lidarr` — `chown -R sc-Lidarr:Lidarr *`. Check what the existing files are owned by before the restore if you aren't sure.
   {.is-info}

7. Start Lidarr.

## See also

- [FAQ](/lidarr/faq) — the questions that come up most often
- [Concepts](/lidarr/concepts) — the model Lidarr uses to manage music
- [Metadata Troubleshooting](/lidarr/metadata-troubleshooting) — when MusicBrainz data is missing or stale
- [Import Troubleshooting](/lidarr/import-troubleshooting) — when downloads finish but don't import
- [Troubleshooting](/lidarr/troubleshooting) — general runtime issues
- [TRaSH Guides](https://trash-guides.info/) — community recipes for media server setups
