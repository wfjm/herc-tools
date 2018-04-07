## hercjis: Hercules Job Input System 

### Table of content

- [Synopsis](#user-content-synopsis)
- [Description](#user-content-description)
  - [Overview](#user-content-overview)
  - [Preprocessor directives and variable substitution](#user-content-dirsub)
  - [Job Template File](#user-content-jesi)
  - [Job Description File](#user-content-jes)
- [Options](#user-content-options)
- [Usage](#user-content-usage)

### Synopsis <a name="synopsis"></a>
```
  hercjis [OPTIONS]... [FILE]...
```

### Description <a name="description"></a>
#### Overview <a name="overview"></a>
hercjis is a preprocessor which allows to generate the complete JCL of a batch
job from
- a [job description file](#user-content-jes) holding the information
  specific for a job like name of template, file names and parameters.
- a [job template file](#user-content-jesi) which contains the JCL for a
  specific job type, e.g. an assembler compile-load-go job.
- source code and data files

The generated JCL is by default submitted directly to a running
[Hercules](https://en.wikipedia.org/wiki/Hercules_(emulator)) system
via a `sockdev reader` (see [-a option](#user-content-opt-a)),
but can alternatively be written to file (see [-o option](#user-content-opt-o)).
hercjis is usually called with one or more job description file, but ready
to run JCL files can of course also be submitted via hercjis.

#### Preprocessor directives and variable substitution <a name="dirsub"></a>
The hercjis preprocessor supports the functions
- inclusion of files
- definition of variables
- substitution of variables

Lines starting with the `//** ##` are considered to be preprocessor directives:
- __//**##define name value__: the variable `name` is set to `value`.
- __//**##include  fname__: include file `fname`, the file name is interpreted
  as relative to the current working directory. Is usually used in
  job templates. 
- __//**##rinclude fname__: include file `fname`, the file name is interpreted
  as relative to the directory of the including file. Is usually used in
  source files to include further source code components.
- __//**## /*__: lines starting with this prefix are ignored as comments.

The substitution syntax is similar to that of the Unix shell `sh`:
- __${name}__: substitutes variable `name`, fails if undefined.
- __${name:=def}__: if variable `name` is undefined set it to `def`,
    than substitute.
- __${name:-def}__: substitutes variable `name`. In case `name` is undefined
    use `def` for substitution, but leave `name` undefined.

Variables can be defined via five mechanisms, which are listed by precedence
(highest to lowest):
- [-D option](#user-content-opt-dd), overrides all other definition sources
- environment variables of the form `HERCJIS_xxx` define variable `xxx`
- `//**##define` directive
- [-d option](#user-content-opt-d)
- `${name:=def}`

#### Job Template File <a name="jesi"></a>
A job template file holds the JCL statements for a given job type.
Many JCL parameters are represented by hercjis variables with an appropriate
default so that they can be overridden, if needed, by the
[job description file](#user-content-jes).
Source or data files are represented by `**##include` directives with a
hercjis variable as file name, which again will be setup in the
job description file. A minimalist template for an assembler
compile-link-go job looks like
```
//${JOB} JOB 'S322-0C4','WFJM'},CLASS=${CLASS:-A},MSGCLASS=A,
//      MSGLEVEL=(1,1),REGION=${REGION:-128K},TIME=${TIME:-(1,0)}
//CLG EXEC ASMFCLG,
//      MAC1='SYS2.MACLIB',
//      PARM.ASM=${PARMC:-'NODECK,LOAD'},
//      PARM.LKED=${PARML:-'MAP,LIST,LET,NCAL'}
//ASM.SYSIN  DD *
//** ##include ${DDSRC}
/*
//GO.SYSIN DD *
//** ##include ${DDDAT}
/*
//
```

with the variables
- `JOB`: job name
- `CLASS`: job class (default = A)
- `REGION`: job memory (default = 128K)
- `TIME`: job time limit (default = (1,0))
- `PARMC`: compile step PARM (default = 'NODECK,LOAD')
- `PARML`: linker step PARM (default = 'MAP,LIST,LET,NCAL')
- `DDSRC`: source file
- `DDDAT`: data file

Template files have usually the file type `.JESI`.
A full-fledged and well documented example of an `ASMFCLG` job template is under
[job_asm_clg.JESI](https://github.com/wfjm/mvs38j-langtest/blob/master/jcl/job_asm_clg.JESI)
in the [mvs38j-langtest](https://github.com/wfjm/mvs38j-langtest) project.
This project contains a full collection of compiler job templates in the
[jcl](https://github.com/wfjm/mvs38j-langtest/tree/master/jcl) directory,
for a description consult the
[README](https://github.com/wfjm/mvs38j-langtest/blob/master/jcl/README.md).

#### Job Description File <a name="jes"></a>
A job description file contains the information for a specific job.
Usually it defines some variables which specify file names or parameters,
and as last action includes the
[job template](#user-content-jesi)
which steers the actual processing. A simple example is
```
//** ##define  JOB    HEWO#ASM
//** ##define  CLASS  B
//** ##define  DDSRC  ../codes/hewo_asm.asm
//** ##define  DDDAT  /dev/null
//** ##include ../jcl/job_asm_clg.JESI
```

Description files have usually the file type `.JES`.
A collection of jobs which nicely demonstrates the usage of hercjis
is part of the
[mvs38j-langtest](https://github.com/wfjm/mvs38j-langtest) project,
located in the
[jobs](https://github.com/wfjm/mvs38j-langtest/tree/master/jobs)
directory and described in the
[README](https://github.com/wfjm/mvs38j-langtest/blob/master/jobs/README.md).
Hercjis was in fact initially developed as part of the mvs38j-langtest
project and later factored out into the separate project
[herc-tools](https://github.com/wfjm/herc-tools).

### Options <a name="options"></a>

| Option | Description |
| ------ | :---------- |
| [-a [nam:]port](#user-content-opt-a) | sockdev address, default localhost:3505 |
| [-o file](#user-content-opt-o)     | build job(s) and write to `file` |
| [-d nam=val](#user-content-opt-d)  | define default for ${nam} substitution |
| [-D nam=val](#user-content-opt-dd) | define override for ${nam} substitution |
| [-c x](#user-content-opt-c)        | short cut for `-D CLASS=x` |
| [-r nn](#user-content-opt-r)       | repeat jobs nn times (only for FILE input) |
| [-f](#user-content-opt-f)          | run as filter (read STDIN, write STDOUT) |
| [-v](#user-content-opt-v)          | verbose, trace files, print statistics |
| [-M](#user-content-opt-mm)         | output Makefile dependencies to `STDOUT` |
| [-w ns](#user-content-opt-w)       | wait ns seconds after command |
| [-h](#user-content-opt-h)          | print help text |

#### -a [nam:]port <a name="opt-a"></a>
hercjis uses as default socket address for job submission `localhost:3505`,
which is the default configuration in [tk4-](http://wotho.ethz.ch/tk4-/).
The `-a` option allows to specify a different socket address. If only a
port number is specified `localhost` will be used as host name.

#### -o file <a name="opt-o"></a>
The job output is written to the file `file`. If `file` is specified as `-`
the output is written to `STDOUT`. If multiple input files are specified
or the [-r option](#user-content-opt-r) is used all jobs will be
concatenated into one file.

#### -d nam=val <a name="opt-d"></a>
Sets the start-up default for variable `nam` to `val`. The variable can later
of over-written with `##define` statements, `HERCJIS_nam` environment
variables, or a [-D option](#user-content-opt-dd).

#### -D nam=val <a name="opt-dd"></a>
Will set the variable `nam` to `val` and override all other definitions
from [-d options](#user-content-opt-dd), `##define` statements, or
`HERCJIS_nam` environment variables.

#### -c x <a name="opt-c"></a>
Is a shortcut for `-D CLASS=x` and useful to setup the job class.

#### -r nn <a name="opt-r"></a>
Will repeat all input files `nn` times.

#### -f <a name="opt-f"></a>
hercjis runs as a filter, reads input from `STDIN` and send output to `STDOUT`.
No [-o option](#user-content-opt-o) and no file names are allowed.
Is equivalent to `hercjis -o - -`.

#### -v <a name="opt-v"></a>
Will create addition output, written to `STDERR`, which traces all file
open and close operations and some statistics, like
```
hercjis-I: open  ifile 's370_perf_ff.JES'
hercjis-I: open  ifile '../jcl/job_asm_clg.JESI'
hercjis-I: open  ifile 's370_perf.asm'
hercjis-I: open  ifile '../sios/otxtdsc.asm'
hercjis-I: close ifile '../sios/otxtdsc.asm'
...
hercjis-I: open  ifile '../sios/sis_iint10.asm'
hercjis-I: close ifile '../sios/sis_iint10.asm'
hercjis-I: close ifile 's370_perf.asm'
hercjis-I: open  ifile '/dev/null'
hercjis-I: close ifile '../jcl/job_asm_clg.JESI'
hercjis-I: close ifile 's370_perf_ff.JES'
hercjis-I send  6833 cards with   20 substitutions
```

#### -M <a name="opt-mm"></a>
Instead of creating jobs hercjis will output rules suitable for `make`
describing the dependencies of the input file.
A [-o option](#user-content-opt-o) must be specified, the file name will
be used as the name of the make target. Works similar to the `-M` option of
`cc` and can be used in the implementation of auto-dependency make rules.
An example can be found in this
[Makefile](https://github.com/wfjm/mvs38j-langtest/blob/master/jobs/Makefile).

#### -w ns <a name="opt-w"></a>
hercjis will wait `ns` seconds before exiting. The implementation of
sockdev devices in Hercules (as distributed with
[tk4-](http://wotho.ethz.ch/tk4-/)) can exhibit a race condition when
back-to-back connections to a `sockdev` reader are made, e.g. by a
sequence of hercjis commands in a script. This options adds some cool-down
time and avoids these race conditions.

#### -h <a name="opt-h"></a>
Print a brief help text and exit.
All other options and arguments will be ignored.

### Usage <a name="usage"></a>
#### Submit job(s)
To directly submit a job to a running Hercules system simply call hercjis
with the name of the job description file as argument. If multiple file name
are specified the jobs are submitted in this order.
```
  hercjis s370_perf_ff.JES
```

#### Create JCL files
To capture the generated job for further usage simply use the
[-o option](#user-content-opt-o) with an appropriate file name.
```
  hercjis -o s370_perf_ff.jcl s370_perf_ff.JES
```
This can be automatized with `make`, consult this
[Makefile](https://github.com/wfjm/mvs38j-langtest/blob/master/jobs/Makefile)
as example.
