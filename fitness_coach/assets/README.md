# Fitness Coach — immutable, deduplicated assets (v1)

**Scope:** Fitness Coach APK `fitnesscoach.workoutplanner.weightloss` v1.1.31 and all resources recovered from `PART001` / `FC_SMALL_00001`–`00027` **only**. `PART002`–`PART010` are **not yet included**. This is a source-derived asset repository, not a claim that all screens/resources of the complete app are recovered.

## Layout

```text
fitness_coach/assets/
  catalog/
    actions.json            # action_id -> mode -> variant -> version -> resources
    remote_assets.json      # exact remote file sources, URLs, hashes, provenance
    apk_local_assets.json   # original APK path -> ready object
    apk_unresolved.json     # original APK binary paths not yet decoded (NO blobs here)
    other_downloads.json    # direct downloaded assets not mappable to action URL
    objects.json            # global SHA-256 -> one immutable asset object path
    coverage.json           # which PARTs, counts and limitations
    issues.json             # parser discrepancies; should be [] for this build
  objects/
    media/ab/<sha256>.mp4|png|webp|jpg|ttf|mp3|ogg
    data/ab/<sha256>.json|xml|properties
  README.md
```

All object paths are **content-addressed**. The same bytes are stored exactly once, even when referenced by multiple actions or APK paths. `objects.json` is authoritative for the file location and SHA-256.

### Resolve a workout resource

For instance, take action `229`, mode `live`, variant `women_white`, version `v12`. Find an entry in `catalog/actions.json`, then append its `object` path to your repository raw asset root:

```javascript
const root = 'https://raw.githubusercontent.com/votaminh/009_file_mini_app/main/fitness_coach/assets/';
const actions = await fetch(root + 'catalog/actions.json').then(r => r.json());
const entries = actions['229']?.live?.women_white?.v12 ?? [];
const video = entries.find(e => e.name === 'videos/video.mp4');
const videoUrl = video ? root + video.object : null;
```

For APK-bundled local images/music, look up the original `apk_path`, e.g. `assets/cover/img_music_01.jpg`, in `catalog/apk_local_assets.json` and resolve `object` the same way. **Do not infer asset paths from action IDs or filenames**; always consult the catalog.

`catalog/remote_assets.json` includes the exact source URL and archive (`data.zip` vs `data_720.zip`), original action ID, mode, variant and version, whenever the source actually provided them. Multiple quality variants stay distinguishable.

## What is intentionally NOT duplicated

- The original `FC_SMALL` split archives and decrypted archive containers.
- Inner `*.zip` resources whose JSON content has already been extracted byte-for-byte.
- Duplicate byte-identical media and JSON files; they share a SHA-256 object.
- 0-byte language placeholders, audit HTTP traces and downloader cache.
- Third-party ad SDK binaries and unrelated APK libraries.

`catalog/apk_unresolved.json` lists APK-embedded binary/compiled XML/encrypted assets that **haven't yet been decoded**. Those bytes have been preserved separately in `APK_UNRESOLVED_EVIDENCE_NOT_FOR_GITHUB.zip`, which does NOT belong in the public assets folder. A manifest entry is **not** proof of a playable or usable asset.

## Updates from PART002–PART010

Each later upload uses **exactly this same path structure**. The update ZIP contains:

1. `fitness_coach/assets/objects/...` — only new SHA-256 content.
2. `fitness_coach/assets/catalog/...` — refreshed index files at the existing paths (overwrite).

Extract the update ZIP into the repository root and commit its `fitness_coach/assets` changes. Never upload the update ZIP file itself to the repository, and never delete existing objects unless an explicit clean-up report proves they are unreferenced. Stable paths are important for the PhonePilot Mini App cache.

The `FC_ASSETS_BUILDER.py` tool (separate tooling ZIP) can build an overlay with `--base` pointing at the already-published repository and `--out` to a new empty output directory. It requires Python 3 and uses only the Python standard library. It **does not decrypt** the input archives; supply previously decoded `FC_DECRYPTED_#####.zip` plus their index CSVs.

**Licensing:** These bytes were extracted from the user-provided APK/CDN capture. Before publishing a public GitHub repository, verify you have permission to redistribute third-party artwork, video and audio.
