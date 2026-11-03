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
