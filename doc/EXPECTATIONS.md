# Expectations of Perl 6 deployment tools

## Original source for this document

See http://pad.hackeriet.no/p/p6-deploy for initial discussions and definitions.
See http://pad.hackeriet.no/p/p6-deploy-spec for ideas about metadata formats and expected files and fields.
See http://pad.hackeriet.no/p/p6-deploy-guidelines for more in-depth thoughts of the deployment process.

## Initial considerations when doing deployment of software

	* Keep track of deployment authority. Which tools have the authority to delete or update something once it has been deployed? How do you ensure that one deployment tool/scheme doesn't muck around with another's files?
	* What kind of files may be installed? A software package doesn't consist only of library files and executables. You may have to install templates, configuration files, documentation, init scripts, database content, third-party client libraries, static data, user interface instructions, and lots more.
	* Should files be filtered or modified before installation? You may want to minify your CSS and JS files, generate bitmaps from your SVG originals or build your PDF files from the source documentation.
	* What does your application depend upon? What deployment authorities can supply these dependencies? How about non-deployable dependencies like available disk space, port numbers and network services?
	* Some files may also have to be fixed or even generated before deployment. How do you handle local patches? Do you want to be able to produce files that contain host- or OS-specific data?
	* Installation context is also important. Is your deployment tool used to prepare packages used for other deployment authorities? While a "system install" may put your files in /usr/local/lib, and a "local install" installs into $HOME/lib, where and how should you install your files when they are intended to be made into an RPM package? How about a Docker image? How about a jail or other chroot-like environment?
	* How about management of configurable resources? Let's say your Nagios installation has a directory where you can drop monitoring scripts. How can the deployment system identify that directory? If any configuration files need to be modified or added, how should this be done? Whose responsibility is it to do this?
	* How about specifying services offered or configurable resources shared by your own software, so that other projects may correctly "drop off" configuration files to set up your software? How can you specify that your software offers such a configurable service that other software can depend upon and use?
	* Feedback loops also need some love! Each of these files may need improvement, so it makes sense to keep them easy to understand, and to make it simple to offer improvements (e.g. with a pull request).

### Pre-publish steps

	1. Some software comes with pre-generated data files. Make sure to update the MANIFEST.json file to reflect this! (This may mean that software checked out from a repository may not be in a "complete" state as far as the MANIFEST is concerned.)
	1. Do pre-publish sanity checks
		2. Check if all files have a type classification in the MANIFEST.
		2. Run the build/test/install/install-tests/post-install-test steps in a clean environmet to check if everything seems ok.
	1. Publish! :)

### The build/test/install steps

	1. Download & unpack – manually or with a build tool. Source may be a tarball or zip file at a website, a source code repository or a directory tree on some physical medium. When successfully completed, you should have a directory tree containing everything necessary to proceed to the next step.
		2. Optionally, check if directory content matches with the MANIFEST.json.
	1. Determine the build, test and installation type you are about to begin.
		2. Do you intend to do a local, or a system install?
		2. If you're doing a system install, do you need it to be installed in a staging area? E.g. so you can use some other packaging tool to create system packages like .rpm or .deb. If you're doing a local install, the staging area doesn't matter since you're installing in your own home directory.
			3. Optionally, specify which installation target index to use when installing files. This is especially important when preparing a staging area with files intended for a different OS or system distribution (e.g. FreeBSD or Debian) than the current build system.
		2. If you are installing into a staging area, do you intend to make a self-contained installation? (Meaning, you should end up with a staging area that contains software with no unresolved external dependencies). This is useful and/or necessary when installing into Jails, Zones, Containers or similar software encapsulation environments. Make sure to supply the staging area (directory name) you intend to use.
		2. Do you intend to install the test suite too, so users later can run the tests on the system they are installed on? (e.g. before and/or after doing an upgrade, to make sure everything is working as expected.)
		2. Do you have all necessary installation locations configured for the system you intend to install on? (Optionally, can you specify these no the fly right now?)
	1. Build the software! Let's call our software "myapp" for now. Building is done by going through a series of sub-steps:
		2. Check if your build dependencies are met.
			3. If they are not, then do a separate download and install for each of these dependencies, but ensure that these are installed locally.
			3. Optionally, you may specify if you want to do an additional system install of the build dependencies (perhaps in a staging area). This may be useful if you want to make it easy to create a build environment usable by others.
			3. If an error happens during determining the build dependencies (e.g. some packages aren't available) then stop the build process and offer the bugtracker URL (and some useful copypastable info) for giving feedback to upstream.
		2. Check if your test dependencies are met.
			3. If they are not, then do a separate download and install for each of these dependencies. Install the test suites locally.
			3. If point 2.4. is true (installing the test suite), then build and install these test dependencies in the same way the original package is configured to be built and installed. If point 2C is set (make a self-contained install) make sure you reuse the staging area from the original install config.
			3. If an error happens during determining the test dependencies (e.g. some packages aren't available) then stop the build process and offer the bugtracker URL (and some useful copypastable info) for giving feedback to upstream.
		2. Check if your runtime dependencies are met.
			3. If they are not, then do a separate download and install for each of these dependencies. Install these in the same way as the original package is configured to be built and installed. If point 2.3. is true (make a self-contained install) make sure you reuse the staging area from the original installation.
			3. If an error happens during determining the run dependencies (e.g. some packages aren't available) then stop the build process and offer the bugtracker URL (and some useful copypastable info) for giving feedback to upstream.
		2. Create a directory specifically for placing the resulting files of the build (e.g. "/tmp/$USER-build-myapp-$UUID").
			3. If point 2.3. is true (make a self-contained install) don't create a new directory, instead use the staging area supplied.
		2. Copy any static source files to the build directory. The build directory tree should look similar to what is expected to be the final installation locations. E.g. if a file is expected to be installed in "/etc/myapp/sites.d" then this file should be found in "/tmp/$USER-build-myapp-$UUID/etc/myapp/sites.d".
		2. If 2.4. is true (installing the test suite), copy the test suite into the build directory, taking care to keep it separate from other installations of the same software, their versions and authorities.
		2. Optionally, apply patches to source files (e.g. local fixes to bugs while you wait for upstream to publish their fixes). If patches are applied, consider changing the source authority of the package to yourself!
		2. Optionally, generate any source files that need pre-build generating. If appropriate, consider changing the source authority of the package to yourself.
		2. Optionally, apply filters to source files (e.g. minify CSS and JS files or create man files from POD).
		2. Copy all source files to the build dir.
			3. Each source file must have a type classification (presumably specified in the MANIFEST file). If a file isn't classified, then this is an error. Stop the build process and offer the bugtracker URL (and some useful copypastable info) for giving feedback to upstream.
			3. Look up in the system-specific installation target index where each file is expected to be installed.
			3. Copy each source file into target directory i the build tree.
	1. Run pre-install tests
	1. If 2.2 is true, install into the staging area.


