# Nytra Examples

Public distribution catalog for optional Nytra examples. Plugin source remains in
its own repositories. No example packages are published yet.

Nytra Examples reads `catalog.json` from the stable `main` branch. Each enabled
plugin registers its provider and this catalog URL; entries for other providers
are not displayed. Opening the browser checks for changes, with a 15-minute
in-session interval. Refresh bypasses that interval. A valid local cache remains
available when the network or a newer manifest fails.

## Versioning

- `schemaVersion` versions the catalog format, currently `1`.
- Each example has a stable provider-prefixed ID and its own `major.minor.patch`.
- Tag releases independently, e.g. `quest-essentials-v1.0.0`.
- Attach engine-specific ZIPs, e.g. `QuestEssentials-1.0.0-UE5.8.zip`.
- Keep previous catalog entries and ZIPs while supported. Never replace the bytes of a published
  version. The browser chooses the newest compatible build for the installed
  engine, provider plugin, and dependencies. If no build is compatible, it shows
  the newest entry with an explanation.
- Builds of the same ID/version must not overlap in both engine and provider
  plugin compatibility ranges. A stable catalog URL does not require UE branches.

## Publishing an example

1. Author and validate the example using the supported public plugin APIs.
2. Keep every imported asset at its final `/Game/NytraExamples/...` package path.
3. Package the assets with ZIP paths beginning `Content/NytraExamples/...`.
   Include dependencies not supplied by a declared plugin. Check references in a
   clean project. Content imports accept `.uasset`, `.umap`, `.uexp`, and `.ubulk`.
4. Create a GitHub Release and attach the versioned ZIP. Publish the release
   before updating the catalog, so users cannot see an unavailable download.
5. Copy an entry from `catalog.example.json`, replacing all illustrative values.
   `bytes` and `sha256` must describe the exact uploaded ZIP. Plugin version
   ranges refer to the integer `Version` in `.uplugin`, not `VersionName`.
6. Commit the entry to `catalog.json` on `main`. No Fab plugin update is needed
   unless the example requires new plugin functionality.

The template is documentation only and is deliberately not a valid live package.
The editor fetches only `catalog.json`.

For a complete standalone project, set `delivery` to `project` and omit
`contentRoot` and `openAsset`. The browser saves its verified ZIP to a user-chosen
location. It does not merge configuration, install plugins, or launch the project.
Gameplay dependencies are shown but need not be enabled just to download it.

## Thumbnails

Optional `thumbnailUrl`, `thumbnailBytes`, and `thumbnailSha256` fields describe
a public HTTPS PNG. Use a versioned release URL or immutable commit URL. Maximum
size is 2 MiB and dimensions are limited to 2048 x 2048. Images are verified and
cached locally. A missing or invalid image leaves the fallback icon visible.

## Updates and trust

Catalogs contain metadata only. Package downloads require exact byte counts and
SHA-256. Catalog responses are limited to 2 MiB and 1000 release entries.
Only HTTPS URLs are accepted. The configured repository and its maintainers are
trusted: catalogs are not cryptographically signed in this implementation.
Protect publishing access and use immutable release assets.

Here, immutable means publishing changed bytes under a new version, rather than
overwriting a previous ZIP. It does not require GitHub's optional immutable-release
setting, which would restrict editing individual attachments.

## Storage and withdrawal

Commit only catalog metadata and publishing documentation to this repository.
Never commit example ZIPs, Unreal assets, or project source here, including through
Git LFS. Upload ZIPs as GitHub Release attachments instead. Attachments are outside
Git history; checking out an old catalog commit cannot recreate a deleted ZIP.
GitHub's automatically generated source archives contain only the repository
files, not the downloadable example attachments.

Each GitHub Release attachment must be under 2 GiB. The current Nytra downloader
accepts one ZIP per entry; multipart archives are not supported yet. Larger
examples need an HTTPS host that supports the entire ZIP, or a smaller package.

To withdraw a version:

1. Remove all matching version/build entries from `catalog.json` and commit it.
2. Delete the ZIP attachments (and any duplicate hosted copies), or their release.
3. Verify that each old download URL no longer serves the archive.

Old catalog commits retain metadata and URLs, not the ZIP bytes. A cached catalog
may briefly show a withdrawn entry. HTTP 404/410 downloads report that the file is
no longer available and refresh the catalog; there is no mirror fallback. Removal
cannot recall copies users already downloaded or stop every in-flight download.

Installation always requires a user action. Existing content is never overwritten,
including modified examples. A newer compatible version is labelled as available;
install it into a separate project to compare or migrate changes deliberately.

## Licensing

This repository does not relicense Nytra plugins or third-party content. Each
published package must state its applicable licenses and third-party notices.
No packages are published until their distribution permissions are established.

