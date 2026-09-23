# scubadex-species-images

Species photos for the Scubadex app's catalogue. This repo holds content, not code. The app
downloads `manifest.json` from `raw.githubusercontent.com` and fetches only the images that
changed since its last sync.

> The five images currently in `/images` are **placeholders** for testing the sync pipeline, not
> final content.

## Layout

```
manifest.json           what the app syncs against
images/<speciesId>.webp one photo per species
```

`speciesId` is the species' numeric id in the app's catalogue (`seedSpeciesCatalogue` in
`AppDatabase.kt`). These ids are permanent, so a file named `52.webp` always means Common Octopus.

## Adding or replacing a photo

1. **Prepare the image.** Resize so the long edge is at most **1024 px**, then convert to WebP
   (quality ~80 is plenty):
   ```
   cwebp -q 80 -resize 1024 0 input.jpg -o 52.webp     # landscape: width capped
   cwebp -q 80 -resize 0 1024 input.jpg -o 52.webp     # portrait: height capped
   ```
   Keep files under ~1 MB. The app rejects anything over 5 MB or anything that isn't a WebP.
2. **Drop it into `/images`** as `<speciesId>.webp`, replacing any existing file.
3. **Update `manifest.json`:**
   - New species: add an entry (see below).
   - Replaced photo: bump that species' `imageVersion` by 1.
   - Fill in `credit` (`club` and `url` may be `null`) and `license`.
4. **Bump the manifest's top-level `version` by 1.** The app skips a sync entirely when this number
   hasn't changed, so if you forget it, nobody gets the new photo.
5. **Commit and push to `main`.** Devices pick it up on their next daily sync.

```json
{
  "id": "52",
  "imageFile": "52.webp",
  "imageVersion": 1,
  "credit": { "name": "Adam", "club": null, "url": null },
  "license": "All rights reserved — used by permission"
}
```

## Removing a photo

Delete the entry from `manifest.json` (you can leave or delete the file) and bump `version`. Devices
delete their cached copy on the next sync, so a withdrawn photo stops showing.

## Rules the app enforces

- `id` must be digits only, and `imageFile` a plain file name (letters, digits, `.`, `_`, `-`; no
  folders). Entries that break these rules are skipped.
- `imageVersion` must be 1 or higher. Any change, up or down, triggers a re-download, so reverting
  a photo is just a matter of setting the version back.
- If an id appears twice, the first entry wins.
- A malformed `manifest.json` doesn't break the app; devices just keep their current images. Still,
  validate before pushing: `python -m json.tool manifest.json`.
