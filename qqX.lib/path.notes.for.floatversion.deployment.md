# PATH notes for standalone floatversion deployment

- EXPLAINS how floatversion is integrated into the [qqX project](https://github.com/qqxproject/qqX)

- For other usages, gives FULL WORKING EXAMPLE HOWTO

- Adds floatversion/qqX detail for developers and AI analyses

See DeepWiki for floatversion structural details at [deepwiki.com/Alex-Genovese/floatversion](https://deepwiki.com/Alex-Genovese/floatversion/2-core-architecture)

## Initialization

References based on **qqX 1.6.01**

Floatversion's integration starts out in the main title script `qqX` where the locations of the key folders are established. The floatversion script is placed in the qqX library folder, along with an `fv` symlink. The library location routine takes place at line 130, about a third of the way through:

```bash
if [[ -d "./qqX.lib" ]]; then qqX_LibraryFolder="$(realpath "./qqX.lib")"
else qqX_LibraryFolder="/usr/share/qqX/qqX.lib"
fi
```

In qqX, this uses standard FHS locations but allows for local overide for qqX dev work:

Normally the single script `qqX` is installed at `/usr/bin` and the remainder scripts and includes at `/usr/share/qqX`

The qqX library folder is always named `qqX.lib` and the location path/filename gets to be stored in the variable `$qqX_LibraryFolder` for easy later usage

When the qqX working directory also contains the library, this indicates that a non-installed initialization has taken place.

## PATH

The variable `$PATH` is a built-in Linux variable. When any command is run at the command prompt, either the path to the binary or the executable script should be given along with the file's name, OR the path should be already stored in a pre-set list that can be checked through, one by one.

A simple PATH for Ubuntu would look like this:

```bash
echo "$PATH"
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

The colon `:` separated listed is checked in left to right order and the file is executed at the first point at which it is found

### export

The command `export` will modify set system configurations on a _temporary_ basis, for the time duration of the terminal or of the process that is started

## Adding the Library

The qqX suite of scripts act independently but also as a wrapper for quickemu and for quickget.

Quickget is a key target for `floatversion` and enables easy dynamic grepping of distro release data

### Method 1

A [modified downstream fork](https://github.com/qqxproject/quickemu) of quickget contains code lines to pull in the library files, located right at the start:

```bash
OriginalPATH="$PATH"
CurrentFolder="$(pwd)"
if [[ -e "/usr/share/qqX/qqX.lib/fv" ]]; then qqX_LibraryFolder="/usr/share/qqX/qqX.lib"
else printf "\n  qqX libraries not found .... \n\n" ; exit 1
fi
export PATH="$CurrentFolder:${qqX_LibraryFolder}:${OriginalPATH}"
if  grep -sqi 'C.utf8' <<< "$(locale -a)"; then export "LC_NUMERIC=C.UTF-8" ; export "LC_COLLATE=C.UTF-8"
else export "LC_NUMERIC=C" ; export "LC_COLLATE=C"
fi
```

The modified `quickget` script is contained in the the qqX `builtins` folder, along with the additional modifications file `qqX_quickGet_mods`. But as quickget is run on a separate process to qqX, we need to find where things are here too, as in line 10

```bash
if [[ -e "$(pwd)/qqX_quickGet_mods" ]]; then
```

Other lines at the start, in particular `CurrentFolder="$(pwd)"` determine if the modified quickget file is being run as a completely independent start from qqX, which is another allowed start method ....

Further down, at line 3890, towards the end, the exported PATH gets checked for the presence of `qqX_quickGet_mods`

```bash
mapfile -t -d':' PathArr <<< "$PATH"
for QGP in "${PathArr[@]}"; do
    [[ -e "$QGP/qqX_quickGet_mods" ]] && source "$QGP/qqX_quickGet_mods" && break
done
```

As the 'Mods' script gets sourced towards the end of the standard Quickget script, any case statement / command mods will either clear any listed args and set flags early, or will run something and exit there, effectively being able to bypass selected original Quickget case statements, or add new ones

This also enables any modified functions to overwrite the pre-existing ones:

eg. Upstream at line 1085

```bash
function releases_slint() {
    echo "15.0-10"
}
```

gets fixed, at `qqX_quickGet_mods` line 870, and will now auto-update

```bash
function releases_slint() {
    web_pipe "https://slackware.uk/slint/x86_64/" | grep -e '<a href=' | fv -M -Q
}
```

plus in this case, as we are using the standalone, can pipe directly through to the `fv` symlink and to full code `floatversion`

### Method 2

Shortly after STANDARD INITIALIZATION, and in a similar way to above, the file `qqX_read_main_settings` checks and records the original pre-export `$PATH` value at line 147 `OriginalPATH="$PATH"`

At line 156, function `sort_qqX_builtin_settings` then works out which builtin code base that qqX is going to use and sets this to the variable `$FreePath`

eg line 189 and others:

```bash
if [[ $QE_SourceVersion == "freebird" ]]; then
  QE_SourceVersion="FreeBird"
  FreePath="$qqX_BuiltinsFolder/freebird"
elif ....
```

This, in turn, is used to set the source and library paths, at line 218:

```bash
  QE_SourcePath="$FreePath/quickemu"
  QG_SourcePath="$FreePath/quickget"
  if [[ -e "$FreePath/qqX_quickGet_mods" ]]; then QG_Modpath="$FreePath/qqX_quickGet_mods" ; else QG_Modpath= ; fi
  if [[ -e "$FreePath/qqX_quickEmu_mods" ]]; then QE_Modpath="$FreePath/qqX_quickEmu_mods" ; else QE_Modpath= ; fi

  if [[ $FreePath ]]; then
    export PATH="${FreePath}:${qqX_LibraryFolder}:${OriginalPATH}"
  else export PATH="${qqX_LibraryFolder}:${OriginalPATH}"
  fi
```

This enables `floatversion` to be used in other areas of qqX
