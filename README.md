# 4d-plugin-file-information

Get DLL/EXE file information.

### Platform

| carbon | cocoa | win32 | win64 |
|:------:|:-----:|:---------:|:---------:|
|||<img src="https://cloud.githubusercontent.com/assets/1725068/22371562/1b091f0a-e4db-11e6-8458-8653954a7cce.png" width="24" height="24" />|<img src="https://cloud.githubusercontent.com/assets/1725068/22371562/1b091f0a-e4db-11e6-8458-8653954a7cce.png" width="24" height="24" />|

### Version

<img width="32" height="32" src="https://user-images.githubusercontent.com/1725068/73986501-15964580-4981-11ea-9ac1-73c5cee50aae.png"> <img src="https://user-images.githubusercontent.com/1725068/73987971-db2ea780-4984-11ea-8ada-e25fb9c3cf4e.png" width="32" height="32" />

### Releases

[Releases](https://github.com/miyako/4d-plugin-file-information/releases)

Original article: [miyako.github.io](https://miyako.github.io/2019/12/13/4d-plugin-file-information.html)

## Manual source version and publication

**Bump version** and **Build** are independent manual Actions workflows. Select
the source branch in the UI. Bump offers `patch`, `minor`, and `major`, and
commits only source version metadata to that branch. It never tags, publishes,
or dispatches Build. **Build has zero inputs** (only the Actions ref selector);
it derives the version, `vX.Y.Z` tag, and asset names from the selected source.
Neither workflow runs on push. Every checkout pins `${{ github.sha }}`.
No new branch restriction is introduced; Bump requires a selected branch.

### Authority and intake repair

The original `02309b6` bump header explicitly promises coordinated
`MARKETING_VERSION`/`CURRENT_PROJECT_VERSION` and
`CFBundleShortVersionString`/`CFBundleVersion`, overriding its buggy
marketing-first/current-fallback implementation. The subsequent `f29be0c`
bump advanced all three marketing fields and `Info.plist` to **2.0.2**, but
left all three current/build fields at 2.0.1. Rename-follow history of the
project, plist, and English strings reaches `4ca7641` (`2.0`) and `f0f6e1b`;
the Xcode project actually copies `English.lproj/InfoPlist.strings` through its
Resources phase and uses the root `Info.plist` in all three target configurations.
The source English strings still displayed 2.0.

This migration repairs only the three current/build fields and the two
localized displays to the **existing 2.0.2**, without an intake increment.
It preserves the absent optional `CFBundleVersion`, CR-only localization,
copyright, file modes, and unrelated bytes. The old plist and UTF-16 English
strings inside the committed test bundle are **prebuilt outputs**, not extra
source inputs; they remain untouched. Build invalidates these stale copied
metadata products before Xcode runs and verifies the newly built values.

All three assignments of each project version key must agree with the source
plist and both localized displays as strict `X.Y.Z`. Supported Xcode references
are preserved; unknown/cyclic references, drift, duplicate plist or localized
keys (including quoted/inline duplicates), and unsupported serialization fail
before writes. Rewrites preserve BOMs, CR/LF/CRLF/mixed endings, whitespace,
modes, and unrelated bytes. Prebuilt bundles, Windows projects, dependencies,
4D data/catalogs, manifests, schema and localization counters are excluded.
The existing remote `v2.0.2` intentionally blocks publication of intake:
run Bump before a future Build needs a fresh publication version.

The full exact committed prior art was compared and adapted:
[`native-encoding-detection@c42ca56`](https://github.com/miyako/4d-plugin-native-encoding-detection/tree/c42ca565857f674bde5ea6ed3435c87375eda4a7)
supplies the coordinated contract, strict localized tokenizer, advertised-SHA
guard, lifecycle and closed-PATH native tests;
[`codec@72c6748`](https://github.com/miyako/4d-plugin-codec/tree/72c67486d8f8ca88c1d72935137502b3d7ac4465)
demonstrates two actual source plists, unlike this project's single source plist;
[`common-string-algorithm@2606a68`](https://github.com/miyako/4d-plugin-common-string-algorithm/tree/2606a68201866ec9fcbb38b51462d79215f504ef)
and corrected
[`window-button@3f14fac`](https://github.com/miyako/4d-plugin-window-button/tree/3f14facb9aaaaff0e22eeb8e07531163680c3a8a)
preserve independent build metadata under marketing-only/fallback headers.
That independent policy is not imported into this explicitly coordinated project.

### Publication safeguards and recovery

Bump requires a clean checkout and the exact selected remote branch SHA.
Its generated commit must be a source-only, single-parent direct child with
the Copilot coauthor trailer. A mandatory executable pre-push guard is checked
initially and immediately before push; it requires exactly one advertised
update with the expected old SHA, literal new SHA/HEAD, selected target and
parent. Advance, rewind, deletion, and same-result races fail. Push is
nonforce with both `push.followTags=false` and `--no-follow-tags`.

Build preserves Windows x64, MSBuild v143/SDK 10.0, macOS universal
arm64+x86_64, Xcode Default/deployment 11.5, resources, dependencies,
`windows-binaries`, all six existing signing/notary secret names, and
`File Information-X.Y.Z.zip`/`.dmg`. Exact fresh output paths, source-matching
bundle metadata, both architectures with no legacy i386 slice, signature verification, both notary
statuses `Accepted`, and DMG stapling are required before publication.

All branches share constant `release-publication` concurrency with cancellation
disabled. Early and final checks require exact remote tag absence and authenticated,
fully paginated release lists including drafts. Invalid data, 404, authentication,
network and later-page errors fail closed. Publication uses separate create-only
POSTs: tag ref pinned to selected SHA, remote verification, new public
nonprerelease release with explicit tag/target/name and original empty body
(no generated notes), then assets only to the newly returned release ID.
No reuse, updates, deletion, clobber, rollback or automatic retries occur.

**Failures retain partial state**, including a tag, public release and earlier
successful uploads. Inspect the selected SHA/tag, remote refs, drafts/public
releases and assets before deciding on manual recovery or a fresh source bump.
Do not blindly rerun; an existing tag/release deliberately blocks publication.
A branch race requires review and a fresh Bump dispatch, never a forced push.

### Local checks

Run `actionlint .github/workflows/*.yml` and
`PYTHONDONTWRITEBYTECODE=1 python3 -m unittest discover -s .github/tests -v`.
The full suite also runs in a disposable post-bump source copy. Fixtures cover
live and stable versions, byte/mode preservation, exclusions, duplicates and
references, real local bare-Git races/guard loss/multiparent commits, mocked
paginated APIs/races/retained state/payloads, and closed-PATH native stubs.
PowerShell (`pwsh`) enables the Windows shell harness. Tests do not execute
native builds, signing, notarization, 4D, Actions dispatch or GitHub publication.

---

# File Information

File Information reads the version resource (`VS_VERSIONINFO`) of a Windows executable, DLL or other PE file and returns its string fields, such as `FileVersion`, `ProductVersion` and `CompanyName`, as two parallel `Text` arrays. It uses the Win32 version API (`GetFileVersionInfoSize`, `GetFileVersionInfo`, `VerQueryValue`). The plugin is built for macOS too, so a cross-platform database can load it, but on macOS the command does nothing.

| Command | Returns | Purpose |
|---|---|---|
| [FILE Get information](#file-get-information) | `Longint` | Read the version strings of a Windows EXE/DLL into a key array and a value array |

**Platforms:** Windows (64-bit). macOS (Apple silicon and Intel) loads the plugin, but the command is a no-op there.

---

## Requirements & platform notes

- **Windows only.** **On macOS** the command doesn't read the path, returns `0`, and resizes both arrays to zero elements. A `0` result therefore does not mean "success" on macOS; check the platform before relying on the arrays.
- **All three parameters are mandatory.** The manifest declares `FILE Get information(&T;&Y;&Y):L`: a path, then two `Text` arrays the command fills.
- **The path is passed as is to Windows.** Give a full Windows path, such as the one `Application file` returns. The plugin does no conversion.
- **Only files with a version resource return data.** Executables and DLLs normally have one. Other files, missing files and files that can't be opened return a negative error code (see [Error handling & troubleshooting](#error-handling--troubleshooting)); no 4D error is raised.
- **Preemptive processes:** the manifest marks the command `threadSafe`, and it uses only local state, so it can be called from a preemptive process.

---

## FILE Get information

### Syntax

```
FILE Get information ( path ; keys ; values ) → error
```

| Parameter | Type | Description |
|---|---|---|
| `path` | Text | Full Windows path of the EXE, DLL or other file with a version resource |
| `keys` | Text array | Receives the field names (output) |
| `values` | Text array | Receives the field values, in the same order as `keys` (output) |
| Result | Longint | `0`, or a negative error code: `-2` the file has no readable version resource, `-3` the version resource couldn't be read |

### Description

The command reads the file's version block, looks up the list of language/code-page pairs in `\VarFileInfo\Translation`, and for each pair queries these string fields, in this order:

`Comments`, `CompanyName`, `FileDescription`, `FileVersion`, `InternalName`, `LegalCopyright`, `LegalTrademarks`, `OriginalFilename`, `ProductVersion`, `PrivateBuild`

For each field it appends the field name to `keys` and its value to `values`. A field the file doesn't define gets an empty value, so `keys` and `values` always have the same size. The data starts at element `1`; element `0` is empty.

- **`SpecialBuild` is not returned.** The plugin's field list contains it, but the loop stops after `PrivateBuild`.
- **One block of 10 fields per translation.** Most files declare a single translation, so you get 10 elements. A file that declares several languages returns one block of 10 per language, with the same key names repeated and no indication of which language a block belongs to.
- **Since the review fix that follows release 2.0.1**, the number of blocks is exactly the number of translations. Earlier builds could return extra blocks with empty or garbage values, because the loop bound was overwritten while the values were read.
- **On Windows**, if the file has a version resource but no `\VarFileInfo\Translation` entry, the command returns `0` and both arrays have no elements (`Size of array` is `0`).
- **On macOS**, nothing is read: the result is `0` and both arrays end up with zero elements.

The values are returned as stored in the resource. `FileVersion` and `ProductVersion` are the free-form strings (e.g. `"10.0.19041.1 (WinBuild.160101.0800)"`), not the binary `VS_FIXEDFILEINFO` numbers, which the plugin doesn't return.

### Example

From the plugin's own test method (`test/Project/Sources/Methods/TEST.4dm`):

```4d
ARRAY TEXT:C222($keys;0)
ARRAY TEXT:C222($values;0)

$error:=FILE Get information (Application file:C491;$keys;$values)
```

Look up one field by name:

```4d
ARRAY TEXT($keys;0)
ARRAY TEXT($values;0)

$error:=FILE Get information (Application file;$keys;$values)

If ($error=0)
	$pos:=Find in array($keys;"FileVersion")
	If ($pos>0)
		$version:=$values{$pos}
	End if 
End if 
```

Copy the first block into an object (later blocks, for other languages, are skipped):

```4d
ARRAY TEXT($keys;0)
ARRAY TEXT($values;0)

$info:=New object
If (FILE Get information ($path;$keys;$values)=0)
	For ($i;1;Size of array($keys))
		If ($info[$keys{$i}]=Null)
			$info[$keys{$i}]:=$values{$i}
		End if 
	End for 
End if 
```

---

## Error handling & troubleshooting

- **`-2`: no version resource.** `GetFileVersionInfoSize` failed: the file doesn't exist, can't be opened, isn't a PE file, or has no version resource (many DLLs built without a `.rc` file). Both arrays end up with no elements.
- **`-3`: the version resource couldn't be read.** `GetFileVersionInfo` failed after the size query succeeded. Rare; both arrays end up with no elements.
- **`0` with empty arrays on Windows.** The file has a version resource but no translation table, so the plugin doesn't know which language block to read. It doesn't fall back to common code pages such as `040904B0`.
- **All values empty.** The file's translation table names a language/code-page pair that has no matching `StringFileInfo` block. The keys are still returned.
- **`0` with empty arrays on macOS.** Expected: the command is Windows-only.
- **Repeated key names.** The file declares more than one translation; see the [FILE Get information](#file-get-information) description.
- **Error code `-1` is never returned.** It is defined in the source but not used.

---

## Quick reference

```4d
ARRAY TEXT($keys;0)
ARRAY TEXT($values;0)
$error:=FILE Get information (Application file;$keys;$values)
// 0 ok, -2 no version resource, -3 read failed; always 0 on macOS (no-op)
$pos:=Find in array($keys;"FileVersion")
If ($pos>0)
	$version:=$values{$pos}
End if 
```
