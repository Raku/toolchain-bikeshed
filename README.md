# toolchain-bikeshed
Discussion area for the Perl 6 toolchain

# packaging issues
For packaging you will want to build in a temporary directory with an unprivileged user, package the resulting files and make installation mostly just extracting of the resulting archive.
* FIXED: precomp files and the .deps files contain absolute paths which during rpm package build contain $RPM_BUILD_ROOT.
* FIXED: rev-deps have to be re-compiled on installing a package. This needs root and changing of installed files.
* FIXED: .rev-deps files of installed modules have to be changed when installing new modules.
* FIXED: existing files in the short/ directory have to be changed when installing new modules.

# possible solutions
* IMPLEMENTED: We're gonna need a way to provide the source file and the path to use as $?FILE separately to the precomp process (requires changes to rather straight forward code in nqp)
* REMOVED: The .rev-deps files should be replaced by directories where you can simply drop in the additions.
* IMPLEMENTED: The files in short/ should probably also be replaced by directories which unfortunatly will incure some runtime cost. This cost may be offset by adding enough information (ver, auth, api) to these files so we only need to load the dist file of the final candidate.

# open tickets related to this discussion
* https://rt.perl.org/Ticket/Display.html?id=127031
* CLOSED: https://rt.perl.org/Ticket/Display.html?id=127058

# nice to have
  * FIXED: Non-SHA filenames for installed modules (this doesn't have to be the case on disk, as long as backtraces include the original filenames)
  * IMPLEMENTED: CUR tools to list installed dists, uninstall dists, list dist contents (basically `any(rpm|dpkg|pacman)` for CUR)
  * (for panda) don't install to `~/.perl6` by default if you don't have access to site installation; that, or at least make this configurable.  A really nice-to-have would be `--site`, `--user`, and `--lib` (which would install to a custom CUR, much like P5 `local::lib`).
  * `META6.json` could include a `doc_requires` for modules used only in `DOC` blocks.
  * The [Elm](http://elm-lang.org/) package manager/ecosystem has some notable features I think it would be cool to adopt:
    * The package manager has a `diff` function that will tell you the differences between two versions of a package (which functions were added or removed, and which functions' signatures changed)
    * The Elm package ecosystem will refuse to accept a package update that changes the interface (removes a function or changes its signature) without bumping the semantic version
