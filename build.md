# Declarative Build System
We want to replace Build.pm by a declarative build system driven by entries in the META6.json.
From systemd-init we can learn that this will probably need a lot of directives to cover all use cases but that this is still better than each dist implementing it new in a (badly maintained) build script.
From Perl 5 we can learn, that we won't get this right the first time or even the second time.

# Suggested structure in the META data
The top level "builder" key specifies the name of the module used to build this distribution.
This module must consume the Distribution::Builder role.
As a start, Distribution::Builder::MakeFromJSON - a JSON driven Makefile generator is proposed.

The top level "build" key is the entry point for MakeFromJSON's directives. It points to a hash possibly containing the following keys:
For example:
```json
{
    "build": {
        "makefile-variables": {
            "p5helper": {"library": "p5helper"},
            "perlopts": {"run": "perl -MExtUtils::Embed -e ccopts -e ldopts"}
        }
    }
}
```

## "makefile-variables"
A hash containing names and values of variables that may be used in a Makefile.in to generate a Makefile.
The key "p5helper" will cause the string "%p5helper%" in Makefile.in to be replaced by the value given by the META file.
Values may be literals or hashes which further specify how to generate the value.

### library
library values specify the names of resources which are native libraries. They will be expanded using $*VM.platform-library-name to match what NativeCall does.
E.g. "p5helper" will be expanded to "resources/libraries/libp5helper.so" on Linux or "resources\libraries\p5helper.dll" on Windows.
Rationale: use as a Makefile target or an output option for a compiler.

### run
Runs the given command on a shell and uses the output as value for the Makefile.in variable.
The command can be a literal or again a hash for selection by platform specifics.
The value hash would consist of a single key and a single value which is again a hash.

Keys may be:
* "by-kernel.name"
* "by-kernel.version"
* ...
These refer to the $*KERNEL Perl 6 variable and its properties. Access to other, yet to be determined variables is expected to be useful.

The values hash contains the different cases. E.g. "windows" or "linux" for kernel.name or an empty string as fallback if no other key matches.

The platform specifics hashes may be nested for building a selection tree.

For example:
```json
{
  "perlopts": {
    "run": {
      "by-kernel.name": {
        "windows": "perlopts.bat",
        "linux": {
          "by-kernel.version": {
            4: "perlopts-new-kernel.sh",
            2.6: "perlopts-old-kernel.sh",
            "": "perlopts-ancient-kernel.sh",
          }
        }
      }
    }
  }
}
```
# Survey of the existing ecosystem
Looking through the 41 Build.pm files currently in use in the Perl 6 ecosystem, the following use cases were identified:
* supporting only certain platforms (Linux::Fuser)
* checking external dependencies
* build a native lib
* call some shell command to get build vars
* git clone some test data
* download dlls for Windows (IOW ensuring external requirements are met) (Gtk::Simple)
* platform dependent make command? (Native::LibC)
* run some Perl 6 script to actually create the Perl 6 code (Math--ThreeD)
* building an external library from source (could just be moved to the Makefile) (p6-fcgi)
* defaults for build variables (Inline::Scheme::Gambit)
* re-creating make's dependency management in Perl 6 (v5)
