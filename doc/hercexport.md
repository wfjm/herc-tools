## hercexport: list and unload to ASCII of a set of DASD

### Table of content

- [Synopsis](#user-content-synopsis)
- [Description](#user-content-description)
- [System Requirements](#user-content-sysreq)
- [Options](#user-content-options)
- [Environment](#user-content-env)
- [Usage](#user-content-usage)
- _source code:_ [bin/hercexport](../bin/hercexport)

### <a id="synopsis">Synopsis</a>
```
  hercexport [OPTIONS]... [FILE]...
```

### <a id="description">Description</a>
hercexport iterates over a set of Hercules
[DASD](https://en.wikipedia.org/wiki/Direct-access_storage_device)
volumes, searches for
[PDS](https://en.wikipedia.org/wiki/Data_set_(IBM_mainframe)#Partitioned_data_sets)
with record format `FB 80` and unloads all or a subset of them as ASCII
files into a directory structure. Listings of all available datasets for each
volumes can be generated optionally, see [--ls option](#user-content-opt-ls).
Filters allow to select the PDS to export, see
[--select option](#user-content-opt-select) and
[--exclude option](#user-content-opt-exclude).
The output path can be specified with the
[--dst option](#user-content-opt-dst). By default two sub-directory levels
are used, one for the DASD container name and one for the PDS name.
When the [--flat option](#user-content-opt-flat) is active only a single
sub-directory level indicating the PDS name is used.

The [--all option](#user-content-opt-all) enables listing generation and
selects all PDS for a full system export. The exported system can than be
easily inspected with standard Unix tools like `find`, `grep`, `xargs` etc.

### <a id="sysreq">System Requirements</a>
hercexport uses the `dasdls` and `dasdpdsu` utility programs and requires
a Hercules version which supports the extended listing options of `dasdls`,
especially `-hdr`, `caldt`, `-expdt`, `-dsnl`, and `-yroffs`.

hercexport **does work** with the `dasdls` coming with
- [Fish-Git/hyperion](https://github.com/fish-git/hyperion) after commit
  [8569600](https://github.com/Fish-Git/hyperion/commit/8569600)
  done 2016-12-15.
- [rbowler/spinhawk](https://github.com/rbowler/spinhawk) after commit
  [b4cb771](https://github.com/rbowler/spinhawk/commit/b4cb771)
  done 2013-04-13.

and **does not work** with
- [hercules-390/hyperion](https://github.com/hercules-390/hyperion) as of commit
  [d15e28f](https://github.com/hercules-390/hyperion/commit/d15e28f)
  done 2018-05-03.
- [tk4-](http://wotho.ethz.ch/tk4-/) as of update 08.

See [section environment](#user-content-env) for a mechanism to access `dasdls`
from a different Hercules installation.

### <a id="options">Options</a>

| Option | Description |
| ------ | :---------- |
| [--dst=odir](#user-content-opt-dst)  | output path for exported pds members |
| [--select=npat](#user-content-opt-select) | select pds with names matching npat |
| [--exclude=npat](#user-content-opt-exclude) | exclude pds with names matching npat |
| [--dsnl=n](#user-content-opt-dsnl)   | dataset name length (default 40) |
| [--yroffs=n](#user-content-opt-yroffs)  | year offset (default 0) |
| [--flat](#user-content-opt-flat)     | no per-dasd subfolders |
| [--all](#user-content-opt-all)       | short for --ls and --sel='.*' |
| [--ls](#user-content-opt-ls)         | generate dasd listing |
| [--trace](#user-content-opt-trace)   | trace dasd+pds names |
| [--v](#user-content-opt-v)           | show dasdpdsu output |
| [--help](#user-content-opt-help)     | print help text |

#### <a id="opt-dst">--dst=odir</a>
Defines the output path for exported PDS members and volume listings.
The directory `odir` must exists when hercexport is called, all
sub-directories are created as needed. When no `--dst` option is
given the current working directory is used.

#### <a id="opt-select">--select=npat</a>
`npat` is taken as
[PERL regular expression](https://perldoc.perl.org/perlre.html),
only PDS matching `npat` are exported, the match is case-blind. The 
Examples:
- `--sel=sys1\..*` selects PDS with names starting with `SYS1.`
- `--sel=.*\.cntl` selects PDS with names ending in `.CNTL`
- `--sel=.*` selects all PDS (see [--all option](#user-content-opt-all))

Multiple `--select` options can be specified, a PDS is selected when it
matches any of them. Note that `.` is a regular expression meta character
and must be escaped as `\.` when a `.` in the PDS name is to be matched.

#### <a id="opt-exclude">--exclude=npat</a>
`npat` is taken as
[PERL regular expression](https://perldoc.perl.org/perlre.html),
a PDS matching `npat` is excluded from the selection specified with
[--select](#user-content-opt-select) and [--all](#user-content-opt-all)
options. The match is case-blind.
Multiple `--exclude` options can be specified, a PDS is excluded when it
matches any of them.

#### <a id="opt-dsnl">--dsnl=n</a>
Defines the column width of the dataset name field in listings.
Is passed on to `dasdls`. The default for `n` is 40.

#### <a id="opt-yroffs">--yroffs=n</a>
Defines the year offset used in the time conversion of dataset
creation and expiration dates in listings.
Is passed on to `dasdls`. The default for `n` is 0.

#### <a id="opt-flat">--flat</a>
Only a single sub-directory layer is used which reflects the PDS name.

#### <a id="opt-all">--all</a>
Equivalent to `--ls` plus `--sel=.*`, triggers the generation of
volume listings and the export of all PDS.

#### <a id="opt-ls">--ls</a>
A dataset listing is created for each volume.

#### <a id="opt-trace">--trace</a>
The currently processed volume container names and PDS names are written
to `stdout`. Can be used to check PDS selection.

#### <a id="opt-v">--v</a>
Shows output of `dasdpdsu`. Best used together with `--trace`.

#### <a id="opt-help">--help</a>
Print a brief help text and exit.
All other options and arguments will be ignored.

### <a id="env">Environment</a>
hercexport requires a `dasdls` with extended listing capabilities, see
[section system requirements](#user-content-sysreq). In case the
Hercules installation used for normal operations does not provide this,
it is possible to use `dasdls` and `dasdpdsu` from a different path.
If the environment variable `DASDTOOLS_PATH` is defined, these two utility
programs will be taken from there and not searched via `PATH`.

### <a id="usage">Usage</a>

#### Get all PDS members with a given prefix
To get the content of all PDS with the prefix `SYS1.` into a flat
directory hierarchy (because it's not relevant to know where the
PDS is located) use
```
  cd <dasd-directory>
  hercexport -dst <dst-path> -trace -flat -sel='sys1\..*' *
```

#### Full export
To get a complete export of for example the [tk4-](http://wotho.ethz.ch/tk4-/)
system use
```
  export DASDTOOLS_PATH=<path-of-dasdls-capable-hercules-installation>
  cd <dasd-directory>
  hercexport -dst <dst-path> -all *
```
Setting up `DASDTOOLS_PATH` is required because because tk4- as of update 08
does not provide an adequate `dasdls`, see
[section system requirements](#user-content-sysreq).
The [--all option](#user-content-opt-all) triggers a full export into a
two-layer directory structure. `<dst-path>` will contain a dataset
listing for each volume (with file type `.log`) and a sub-directory
for each volume which has exported PDS. The PDS members are again
collected in sub-directories
```
  hasp00.152.log
  mvscat.191/
    jcc.ftpd-rac.asm/
      ftpauth.mac
      ftpdxctl.mac
      ...
    jcc.ftpd-rac.cntl/
      makeftpd.mac
      makexctl.mac
  mvscat.191.log
  mvsdlb.248/
    algol.f.level21.asm/
    ...
    sys1.amodgen/
      bldcpool.mac
      cvt.mac
  mvsdlb.248.log
  ...
```

The tk4- base configuration from `tk4-_v1.00_current.zip` yields about 11300
files with a total of 146 MB. The full tk4- configuration with `tk4-cbt.zip`
and `tk4-source.zip` volumes yields about 61000 files with 1249 MB.
