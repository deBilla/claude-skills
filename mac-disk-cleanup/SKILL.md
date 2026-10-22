---
name: mac-disk-cleanup
description: Free disk space on a Mac (macOS "System Data" bloat, disk full, low storage, remove docker images). Use when the user says disk/storage is full, System Data is huge, or asks to clean up / delete caches / remove docker images. Lists where developer Macs usually lose space and which deletions are safe vs need confirmation.
---

# Mac disk cleanup

On a developer Mac, macOS "System Data" is mostly **`~/Library`**: Docker VM disks, caches,
Xcode/simulators. Measure first, then delete in tiers. Show the user the full list of what
will be deleted and get **one explicit confirmation** before deleting anything.

## 1. Measure

```bash
df -h /System/Volumes/Data
du -xhd1 ~ 2>/dev/null | sort -rh | head -15
du -xhd1 ~/Library ~/Library/Caches ~/Library/Developer 2>/dev/null | sort -rh | head -25
docker context ls; docker system df
colima list 2>/dev/null
ls -la ~/Library/Containers/com.docker.docker/Data/vms/0/data/ 2>/dev/null   # Docker Desktop disk
tmutil listlocalsnapshots /
```

VM disk files are **sparse**: compare `ls -la` (apparent size) with `du -sh` (real size).
A `Docker.raw` can show 460 GB apparent but use 140 GB.

## 2. Usual hotspots

| Path | What it is |
|---|---|
| `~/Library/Containers/com.docker.docker/Data/vms/0/data/Docker.raw` | Docker Desktop VM disk. Often 50–150 GB, even when Docker Desktop is unused and another runtime is active. The most common culprit. |
| `~/.colima/_lima` | Colima VM disk |
| `~/Library/Group Containers/*.dev.orbstack` | OrbStack data |
| `~/Library/Developer/CoreSimulator` | iOS simulators |
| `~/Library/Developer/Xcode/{DerivedData,iOS DeviceSupport,Archives}` | Xcode build output, device symbols, archives |
| `~/Library/Caches/{Yarn,go-build,pip,CocoaPods,com.microsoft.VSCode.ShipIt}` | Rebuildable caches |
| `~/Library/Caches/Google` | Chrome cache |
| `~/.npm`, `~/.gradle`, `~/.m2`, `~/go/pkg/mod`, `~/.bun`, `~/Library/pnpm` | Package caches |
| `~/.android`, `~/Library/Android` | Android SDK / emulators |

## 3. Tiers

**Safe: rebuildable caches** (still include them in the one confirmation):
```bash
rm -rf ~/Library/Caches/{Yarn,go-build,pip,CocoaPods,com.microsoft.VSCode.ShipIt}
rm -rf ~/Library/Developer/Xcode/DerivedData/*
npm cache clean --force
xcrun simctl delete unavailable
go clean -modcache
rm -rf ~/.gradle/caches
```

**Docker: "remove all docker images".** First find out which runtime is active (`docker context ls`)
and whether anything is running (`docker ps`):
- **Inactive Docker Desktop:** `rm -f ~/Library/Containers/com.docker.docker/Data/vms/0/data/Docker.raw`.
  This also wipes its containers and **volumes**, so tell the user.
- **Active runtime:** `docker image prune -af && docker builder prune -af && docker container prune -f`
  removes only **unused** images. If containers are running (e.g. a local k3s/kind cluster or
  databases), ask before removing their images or the VM (`colima delete`, OrbStack reset).
  These can't be undone, and the safe default is to keep them.
- Docker Desktop's disk image may not shrink after a prune. Use "Troubleshoot → Clean / Purge data",
  or delete `Docker.raw` while Docker Desktop is stopped.

**Ask first: these aren't caches:**
- VM disks for active runtimes, working simulators (`xcrun simctl delete <udid>`), older
  `iOS DeviceSupport` versions, Android emulators, browser caches, source/project folders, `node_modules`.
- Anything outside `~`: it needs sudo, and it's usually the OS and apps, so leave it alone.
- Time Machine local snapshots (`tmutil deletelocalsnapshots <date>`).

## 4. Verify

Re-run `df -h /System/Volumes/Data` and report free space before → after, plus what was kept and why.
