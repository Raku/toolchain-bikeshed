# Installing Perl 6 packages, modules and applications, and installing the Perl 6 executable and accompaning virtual machines

This document contains thoughts on installing all kinds of things! Please read thoroughly and give feedback (to sjn on irc.freenode.org). Thanks! :)

## Original sources

See http://pad.hackeriet.no/p/p6-deploy for initial discussions and definitions.
See http://pad.hackeriet.no/p/p6-deploy-spec for ideas about metadata formats and expected files and fields.
See http://pad.hackeriet.no/p/p6-deploy-guidelines for more in-depth thoughts of the deployment process.

## What can be a dependency?

    - Perl 6 Modules
    - Non-Perl 6 Modules (e.g. when using Inline::Perl5, to depend on Perl 5's DBI)
    - Binary shared libraries (libmysql.so)
    - Binary executables (/usr/sbin/ps)
    - Services (ntpd, pop3, imaps)
    - System resources (disk storage, unused port 80, minimum RAM)

## What types of dependencies do we have?

    - run-depends - Perl 6 dependencies for regular use of the software, during the build step and after it has been installed (in META.info: "depends")
    - build-depends - deps only used during the build step of the installation
    - test-depends - deps only used during the test step of the installation
    - Recommended
    - Suggested (is this necessary?)
    - Notes: 
        - run-depends - deps only used during the runtime (e.g. to install rpm you need only these deps)
        - To run tests during build/install, both build-depends and test-depends have to be satisfied

## General - principles, marketing, memes

- Define "The Perl 6 module deployment stanza"
	- Equivalent to "perl Makefile.PL; make; make test; make install"
	- Common tasks should be easy to understand and trivial to do
	- Make it explicit, make it configurable, make it easy to figure out what's going on
	- CPANfile -- deployment with just configuration (worth checking out)
- Make %*CUSTOM_LIB<site/vendor> configurable so perl6 packagers can adjust them
- Customize installation locations
	- Per-distro defaults (/etc/perl6/locations.conf)
	- Per-system package-specific locations (/etc/perl6/locations.d/*.conf)
	- Per-user overrides ($HOME/.perl6locations.conf or File::Homedir->my_data('.perl6locations.conf') )
	- Also, support config directories (e.g. /etc/perl6/locations.d/myapp.conf)

## Dependency management

	- Bundling dependencies with your code, with an easy way to update them if needed
		- Can this be made possible/optional?
	- Including "rules" with your code that make it easy to build dependencies wherever necessary (Carton-way?)
	- Perl 5 dependencies
	- Non-perl deps (debian packages, whatnot)
		- Compiled/shared object libraries (/usr/lib/libfoo.so)
		- Dependencies are handled nicely by rpm/deb. Use their facilities! (requires a canonical way of specifying names, that work across packaging systems and their different conventions)
		- Supporting services that a Perl 6 application may need. (apache, nginx, nagios, ntpd, etc.)
	- OS-specific deps
		- the same depencencies can have different names on different OSes
		- this can be solved by using the most-upstream name of the dependency (e.g. )
	- Dev environment recommendations

## Versioning

	- of the compiler/runtime (rakudo-parrot, version x)
	- of the dependencies used
		- version-locking is common for production installations, see cpanfile.snapshot of Carton
	- of the software provided
		- Separate API/protocol/file format version numbers?
	- !!! RPM versioning and upgrading is not easily compatible with Perl 6 (as Perl 6 allows to install more than one version of module)

# Software Lifecycle management

- Upgrading
	- Handling different application versions living side-by-side on a system
	- Do NOT replace existing files
- Downgrading
- Patching (temporary fixes)
	- Detecting temporary fixes to production code, and giving a warning about it
- Upstream communication (bug reporting/patch submisison/crash reports, etc.)
- Restarting and other service managment
- Monitoring, log management, etc.
- Post-install testing
	- Needs a way to override the version/authority that the test file requests
	- Where to install tests?
		- Suggestion: Somewhere "close to, but next to an installation"
	- How to enforce that tests don't write files in the test directory?
		- Offer a tempfile() function, or equivalent, as part of the Test module? #notchecked There's File::Temp already (which should really be in core)

## Building

- The build step is for preparing a local installation tree where all files used by the software is to be copied from
    - If any of the files to be installed are generated, this must happen during the build step, and the resulting file(s) must be placed in the build directory
    - When the build step is done, everything in the build directory should be ready for the semifinal and/or final installation step(s).

## Testing

- Tests for different purposes
	- Author tests: thorough/long-running test-suite to run before publishing/releasing the software)
	- Integration tests: quick test-suite to do basic full-stack functionality tests ("smoke testing") in order to get quick feedback if something important broke
	- Acceptance tests: instructions for testing customer-specified acceptance criteria. Can be automated (see the Cucumber project for a way to do this), but is often done manually
- Specifying long-running tests
- Interactive tests (turn on/off as necessary, w/sane defaults)

## Deploying

- Applications
	- Local (system-independent) full-stack deploy: Put everything inside one directory (perhaps owned by an application user)
		- System deploy (expected to be installed with other system tools/libraries/services)
	- Configuration
		- system-wide (/etc/... or /usr/local/etc/...)
		- application-specific (/opt/vendor/product-1.0/etc/...)
		- user-specific ($HOME/.config/product-1.0/config)
	- Libraries
		- vendor (distribution-supplied) libs
		- local (sysadmin-supplied) libs
		- application (/opt/software-vendor/product-1.0/) libs
		- local / user-installed libs ($HOME/perl6/lib/...)
		- Commandline tools (/usr/bin/product-query)
		- system tools (e.g. init scripts, monitoring scripts for Nagios; /etc/init.d/product; )
		- application-specific tools (application cleanup scripts, sanity checks, etc. etc.)
- Data files
- Example configuration files
- Database setup/teardown/upgrade/downgrade/backup preparation
- Provisioning awareness?
	- Puppet/chef/cfengine/Docker etc.
- Undo/rollback support?

## Packaging

- Custom packaging-relevant target dir (may be used in the process of creating RPM's, DEB's etc.)
  - pre-install scripts
  - post-install scripts
  - pre-uninstall scripts
  - post-uninstall scripts
- Code-like behaviour during pre-deployment configuration
  - e.g. handling custom platform specific dependencies (MSWin32 vs. Unix-like vs. MacOS X etc.)
- Managing package namespace differences between Perl space and $package_manager space in order to automatically resolve dependencies

## Metadata availability

- What information should be exposed to indexers/installation tools/etc.?

Scenarios
    - What do we need from them? (what does rpm need? Docker? Copying? post-install scripts?)
    - 4 types of config files for each scenario: OS, vendor, packager, developer

## Relevant software

    - Panda https://github.com/tadzik/panda
    - Zef https://github.com/ugexe/zef
    - Rakudo build and installation files https://github.com/rakudo/rakudo/
    - The CompUnitRepo MANIFEST
	- Make it use SHA-1 hashes like git, so you don't have to read MANIFEST or anything to figure out which 'file number' is free

# Related links
- Synopsis 22: Distributions, Recommendations, Delivery and Installation: http://design.perl6.org/S22.html

# Useful resources

- Example Perl 6 distro: https://github.com/sjn/perl6-example-dist 
