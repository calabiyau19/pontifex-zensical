## Single Source of Truth for Shared Files

### Before

Media and shared files were spread across two locations on the network — one tied to the NUC5 server directly, and one on a separate storage server (samba-storage-vm). The same folders (movies, music, YouTube downloads) existed in both places, sometimes with different content in each, which made it unclear which copy was current. This also led to filesystem corruption on the drive, caused by more than one thing writing to it without coordination.

### Now

All shared files — movies, music, music backup, YouTube downloads, and other shared folders — live in exactly one place: the storage drive attached to **samba-storage-vm** (192.168.1.167). Nothing else on the network holds a separate copy. The drive was checked and repaired for filesystem errors as part of this project and is in a healthy state going forward.

## How to Access It

Everything on that drive is shared over the network using Samba, so any device can reach it as a regular network folder.

### From lpt-HP, using Nemo

Open Nemo, then go to the location:

```
smb://192.168.1.167/storage
```

Log in with username `mark` and the usual password. You'll see folders including `movies`, `music`, `music-backup`, `YouTube-downloads`, and `Blogger`.

Drag and drop files in and out of this location like any other folder — this is how to add a new movie, browse music, or grab a YouTube download.

Bookmark the location in Nemo's sidebar once connected, so it doesn't need to be re-typed each time.

## Obsidian Vault — Handled Differently

Obsidian is not accessed through Nemo or Samba. It stays synced in the background via **Syncthing**, between lpt-HP and **obsidian-vm** (a separate server from samba-storage-vm), plus the iPhone via the Mobius Sync app. Files don't need to be moved manually — Syncthing keeps everything matching automatically.

This is a deliberate difference from the media files above, not an inconsistency: Obsidian needs the vault to be fully local and editable on each device, so real-time sync fits it better than a shared network folder.

## Filesystem Repair Incident (Aug 15)

After the consolidation above was completed, a routine filesystem check (`e2fsck`) was run on the T7 drive to fix pre-existing corruption (unrelated files had ended up sharing the same disk space, confirmed via system logs predating this project). The check found and repaired real corruption, but as a side effect, the `movies` folder lost its contents.

**What was actually lost:** Blade Runner 2049, Disclosure Day, and Project Hail Mary. These were not recoverable from the drive and are being re-downloaded.

**What was recovered or confirmed unaffected:**
- Vikings Season 1 (all 9 episodes) — found intact in the filesystem's recovery folder and restored
- Music, music-backup, YouTube-downloads, Blogger — all initially appeared missing due to a stale/outdated view on the `/mnt/storage` mount point, but were confirmed fully intact once the mount was refreshed. No actual data loss.

**Lesson for future maintenance on this drive:** After any filesystem repair, verify actual folder contents directly (not just that the folder name exists) before considering the drive fully restored. If a folder appears empty or wrong after any mount/unmount cycle, remount it fresh before assuming data is lost — a stale mount view can look identical to real data loss.
