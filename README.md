# toolchain-bikeshed
Discussion area for the Perl 6 toolchain

# packaging issues
* precomp files and the .deps files contain absolute paths which during rpm package build contain $RPM_BUILD_ROOT.
* rev-deps have to be re-compiled on installing a package. This needs root and changing of installed files.

# open tickets related to this discussion
* https://rt.perl.org/Ticket/Display.html?id=127031
* https://rt.perl.org/Ticket/Display.html?id=127058
