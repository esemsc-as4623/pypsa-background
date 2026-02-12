## FAIR Research Software
- Findable
	- software metadata (DOI, citation.cff, version, metadata)
- Accesible
	- public GitHub, git clone, zenodo
- Interoperable
	- standard formats, documented dependencies, stable releases
- Reusable
	- LICENSE, README, environment files

Licenses:
1. permissive
	- e.g. BSD 3-clause, MIT, Apache 2.0
	- no strings attached
	- licenses allow users to distribute software commercially
	- Apache has something to do with patent (try to avoid)
3. copyleft / reciprocal
	- e.g. GNU General Public License 2.0
	- next release required to be under the same license
	- protects openness across lifecycle of project

Environments:
- exact versions
- required packages
- dependency set needed to run the software
- instructions to reproduce execution environment
- pixi = fast environment manager that works with Python, R on MacOS, Linux, Windows
	- automatically creates a lock file that contains exact versions of all packages installed in environment
```bash
cd project
pixi init
pixi add python # and/or R
pixi add numpy # and/or r-dplyr
pixi run python script.py
# and/or Rscript script.R
```
- pixi.toml  = reproducible record of all dependencies
- every command is executed inside environment

CITATION.cff
- use cffinit to set up
- DOI = QR code for software project
	- never breaks
	- points to archived snapshot (Zenodo)
	- includes metadata
	- citable in academic publications
- Mint a DOI
	- log in to Zenodo with GitHub
	- enable repository
	- create GitHub release
	- Zenodo auto-archives and mints DOI
	- add DOI badge to README
	- update CFF with DOI

README
- software front door
- in 30s:
	- understand what it does
	- how to install
	- how to use
- FAIR
	- findable (e.g. clear description to aid discovery)
	- accessible (e.g. easy install)
	- reusable (e.g. add examples)
- essential sections
	- about
	- features
	- getting started
	- usage
	- citation
	- license
	- contact
- tips
	- clear description
	- show, don't tell
	- link metadata (e.g. DOI badge, link to CITATION.cff)
	- keep updated
	- use template

Other GitHub automations
- CONTRIBUTING.md = how to contribute
- CODE_OF_CONDUCT.md = behavioral rules
- CHANGELOG.md = version history
- template: https://github.com/UC-OSPO-Network/templates
## 