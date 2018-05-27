# herc-tools:  Tools for Hercules and MVS 3.8J Turnkey Systems

### <a id="overview">Overview</a>

This projects contains tools for the handling of batch jobs in a
[Hercules](https://en.wikipedia.org/wiki/Hercules_(emulator)) 
plus MVS 3.8J Turnkey System (e.g. [tk4-](http://wotho.ethz.ch/tk4-/))
environment
- [hercjis](doc/hercjis.md) - batch job preprocessor and submission tool,
    supports variable substitution and file inclusion and allows to build
    jobs from modular building blocks. Can submit directly to a
    `sockdev reader` and write the created JCL to a file.
- [hercjos](doc/hercjos.md) - yet another sockdev printer.
- [hercjsu](doc/hercjsu.md) - analyses batch job output files and generates
    compact summaries. Very helpful when a larger number of job outputs
    have to be checked.

`hercjis` should work with all OS running on Hercules. `hercjos` and `hercjsu`
handle, at this point, only output from MVS 3.8J systems with the JES2
configuration distributed with the Turnkey Systems,
[tk3](http://www.bsp-gmbh.com/turnkey/) and
[tk4-](http://wotho.ethz.ch/tk4-/).

The tools were initially developed in the context of the
[wfjm/mvs38j-langtest](https://github.com/wfjm/mvs38j-langtest) and
[wfjm/s370-perf](https://github.com/wfjm/s370-perf) GitHub projects
which now include to herc-tools as submodules.

### Directory organization
The project files are organized in directories as

| Directory  | Content |
| ---------- | ------- |
| [bin](bin) | scripts |
| [doc](doc) | documentation |

### License
This project is released under the 
[GPL V3 license](https://www.gnu.org/licenses/gpl-3.0.html),
all files contain the disclaimer:

    This program is free software; you may redistribute and/or modify
    it under the terms of the GNU General Public License version 3.
    See License.txt in distribition directory for further details.

The full text of the GPL license is in this directory as
[License.txt](License.txt).
