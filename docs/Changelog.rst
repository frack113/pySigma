Changelog
#########

This page lists important changes between versions.

For a complete list of changes, see the `GitHub releases <https://github.com/SigmaHQ/pySigma/releases>`_.

Version 1.4.0
=============

Released: 2026-06-27

New Features
------------

* Add external data source placeholder transformations (File, HTTP, Command)
* Implement `neq` modifier for boolean NOT match

Bug Fixes
---------

* Fix filter rules with wildcard conditions (`1/any/all of pattern_*`)
* Fix `SigmaRule.to_dict()` fails after field_name_mapping transformation
* Harden external data source placeholder transformations
* Fix Dependabot dependency parsing for date-versioned packages
* Update dependencies and fix mypy type errors

Maintenance
-----------

* Enforce black formatting and mypy checks in Copilot agentic sessions
* Bump mypy from 1.20.2 to 2.1.0
* Bump pytest from 9.0.3 to 9.1.1
* Bump pylint from 4.0.5 to 4.0.6
* Bump actions/checkout from 6 to 7

Documentation
-------------

* Improved documentation structure with Furo theme
* Added Quickstart guide
* Added YAML Pipeline Tutorial
* Added Changelog page

Version 1.3.3
=============

Released: 2026-04-21

Bug Fixes
---------

* Various bug fixes and improvements

Version 1.3.2
=============

Released: 2026-03-15

Bug Fixes
---------

* Various bug fixes and improvements

Version 1.3.1
=============

Released: 2026-02-10

Bug Fixes
---------

* Various bug fixes and improvements

Version 1.3.0
=============

Released: 2026-01-20

New Features
------------

* Initial stable release with core functionality
