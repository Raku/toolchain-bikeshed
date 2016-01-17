# toolchain-bikeshed
Discussion area for the Perl 6 toolchain

# packaging issues
* precomp files and the .deps files contain absolute paths which during rpm package build contain $RPM_BUILD_ROOT.
* rev-deps have to be re-compiled on installing a package. This needs root and changing of installed files.

# open tickets related to this discussion
* https://rt.perl.org/Ticket/Display.html?id=127031
* https://rt.perl.org/Ticket/Display.html?id=127058

# nice to have
  * Non-SHA-256 filenames for installed modules (this doesn't have to be the case on disk, as long as backtraces include the original filenames)
  * CUR tools to list installed dists, uninstall dists, list dist contents (basically `any(rpm|dpkg|pacman)` for CUR)
  * (for panda) don't install to `~/.perl6` by default if you don't have access to site installation; that, or at least make this configurable.  A really nice-to-have would be `--site`, `--user`, and `--lib` (which would install to a custom CUR, much like P5 `local::lib`).
  * `META6.json` could include a `doc_requires` for modules used only in `DOC` blocks.
