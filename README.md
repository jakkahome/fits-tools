# fits-tools

Small command-line tools for astrophotography image files. They are plain Python 3
scripts with **no third-party dependencies** (no astropy, no numpy) — just copy them
somewhere on your `PATH` and run them.

## Files

| File | Description |
| --- | --- |
| `fitsparams` | Prints selected FITS header keywords of FITS and XISF files as a table. |
| `fits2xisf` | Converts FITS images to monolithic XISF files, preserving the header keywords. |
| `fitsmedian` | Reports the median pixel value of the central area of FITS and XISF images. |
| `files/` | Sample FITS and XISF frames used for testing (not tracked in git). |

## Installation

```sh
chmod +x fitsparams fits2xisf fitsmedian
ln -s "$PWD/fitsparams" /usr/local/bin/fitsparams
ln -s "$PWD/fits2xisf" /usr/local/bin/fits2xisf
ln -s "$PWD/fitsmedian" /usr/local/bin/fitsmedian
```

## `fitsparams`

Reads headers from `.fit`, `.fits`, `.fts` and `.xisf` files and prints them as an
aligned table. By default it shows `GAIN`, `OFFSET`, `EXPTIME` and `INSTRUME`.

```sh
fitsparams /path/to/directory
fitsparams /path/to/file.fit -p DATE
fitsparams /path/to/directory -p DATE-OBS,XPIXSZ
```

| Option | Description |
| --- | --- |
| `-p, --param KEY` | Extra keyword to show. Repeatable, or comma-separated. |
| `-o, --only` | Show only the keywords given with `-p`. |
| `-r, --recursive` | Search directories recursively. |
| `-c, --csv` | Output comma-separated values instead of an aligned table. |
| `-f, --full-path` | Print full paths instead of file names. |

Missing keywords are shown as `-`. For XISF files both `FITSKeyword` names and XISF
`Property` identifiers (e.g. `Instrument:Camera:Gain`) can be queried.

## `fits2xisf`

Converts a single FITS file or every FITS file in a directory into XISF. The output is
written next to the source file, or into the directory given with `-o`.

```sh
fits2xisf /path/to/file.fit
fits2xisf /path/to/directory -o /path/to/output -y
```

| Option | Description |
| --- | --- |
| `-o, --output DIR` | Output directory (default: next to the source file). |
| `-r, --recursive` | Search directories recursively. |
| `-y, --overwrite` | Overwrite existing XISF files. |
| `--flip` | Flip rows vertically (bottom-up FITS convention). |
| `-q, --quiet` | Suppress progress output. |

Details:

- Writes a monolithic XISF 1.0 file: XML header plus a 4096-byte aligned attached data
  block, with samples converted from FITS big-endian to little-endian.
- Supported sample formats: `UInt8`, `UInt16` (`BZERO=32768`), `UInt32` (`BZERO=2^31`),
  `Float32` and `Float64`; mono and 3-channel RGB images. Only the primary HDU is read,
  and `BSCALE` must be 1.
- All non-structural FITS cards are copied into the XISF header as `FITSKeyword`
  elements, and `IMAGETYP` is mapped to the XISF `imageType` attribute.
- Row order is preserved by default so images open the right way up in PixInsight; use
  `--flip` to mirror them vertically.

## `fitsmedian`

Measures the median pixel value of a centred region covering 10 % of the image area
(i.e. ~31.6 % of the width and height), which is handy for checking sky background or
flat levels.

```sh
fitsmedian /path/to/file.fit
fitsmedian /path/to/directory
fitsmedian /path/to/directory -a 25
```

| Option | Description |
| --- | --- |
| `-a, --area PERCENT` | Percentage of the image area to measure (default: 10). |
| `-r, --recursive` | Search directories recursively. |
| `-c, --csv` | Output comma-separated values instead of an aligned table. |
| `-f, --full-path` | Print full paths instead of file names. |

The `REGION` column shows the measured region in pixels. RGB images get one median per
channel (`MEDIAN_R`, `MEDIAN_G`, `MEDIAN_B`). FITS values are scaled with `BZERO`/`BSCALE`;
compressed XISF data blocks are not supported.
