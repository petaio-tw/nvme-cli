# nvme-cli
NVM-Express user space tooling for Linux.

When cloning from source, you will need the libnvme submodule, so use:

    $ git clone --recurse-submodules https://github.com/linux-nvme/nvme-cli

If you already cloned without recursing submodules, you will need to initialize
and update it:

    $ git submodule update --init

Or as two commands:

    $ git submodule init
    $ git submodule update

To install, run:

    $ make
    # make install

If not sure how to use, find the top-level documentation with:

    $ man nvme

Or find a short summary with:

    $ nvme help

## Distro Support

### Alpine Linux

nvme-cli is tested on Alpine Linux 3.3.  Install it using:

    # apk update && apk add nvme-cli nvme-cli-doc

if you just use the device you're after, it will work flawless.
```
# nvme smart-log /dev/nvme0
Smart Log for NVME device:/dev/nvme0 namespace-id:ffffffff
critical_warning                    : 0
temperature                         : 49 C
available_spare                     : 100%
```

### Arch Linux

nvme-cli is available in the `[community]` repository. It can be installed with:

    # pacman -S nvme-cli

The development version can be installed from AUR, e.g.:

    $ yay -S nvme-cli-git

### Debian

nvme-cli is available in Debian 9 and up.  Install it with your favorite
package manager.  For example:

    $ sudo apt install nvme-cli

### Fedora

nvme-cli is available in Fedora 23 and up.  Install it with your favorite
package manager.  For example:

    $ sudo dnf install nvme-cli

### FreeBSD

`nvme-cli` is available in the FreeBSD Ports Collection.  A prebuilt binary
package can be installed with:

```console
# pkg install nvme-cli
```

### Gentoo

nvme-cli is available and tested in portage:
```
$ emerge -av nvme-cli
```

### Nix(OS)

The attribute is named `nvme-cli` and can e.g. be installed with:
```
$ nix-env -f '<nixpkgs>' -iA nvme-cli
```

### openSUSE

nvme-cli is available in openSUSE Leap 42.2 or later and Tumbleweed. You can install it using zypper.
For example:

    $ sudo zypper install nvme-cli

### Ubuntu

nvme-cli is supported in the Universe package sources for Xenial for
many architectures. For a complete list try running:
  ```
  rmadison nvme-cli
   nvme-cli | 0.3-1 | xenial/universe | source, amd64, arm64, armhf, i386, powerpc, ppc64el, s390x
  ```
A Debian based package for nvme-cli is currently maintained as a
Ubuntu PPA. Right now there is support for Trusty, Vivid and Wiley. To
install nvme-cli using this approach please perform the following
steps:
   1. Add the sbates PPA to your sources. One way to do this is to run
   ```
   sudo add-apt-repository ppa:sbates
   ```
   2. Perform an update of your repository list:
   ```
   sudo apt-get update
   ```
   3. Get nvme-cli!
   ```
   sudo apt-get install nvme-cli
   ```
   4. Test the code.
   ```
   sudo nvme list
   ```
   In the case of no NVMe devices you will see
   ```
   No NVMe devices detected.
   ```
   otherwise you will see information about each NVMe device installed
   in the system.

### OpenEmbedded/Yocto

An [nvme-cli recipe](https://layers.openembedded.org/layerindex/recipe/88631/)
is available as part of the `meta-openembeded` layer collection.

### Buildroot

`nvme-cli` is available as [buildroot](https://buildroot.org) package. The
package is named `nvme`.

### Other Distros

TBD

## Developers

You may wish to add a new command or possibly an entirely new plug-in
for some special extension outside the spec.

This project provides macros that help generate the code for you. If
you're interested in how that works, it is very similar to how trace
events are created by Linux kernel's 'ftrace' component.

### Add command to existing built-in

The first thing to do is define a new command entry in the command
list. This is declared in nvme-builtin.h. Simply append a new "ENTRY" into
the list. The ENTRY normally takes three arguments: the "name" of the 
subcommand (this is what the user will type at the command line to invoke
your command), a short help description of what your command does, and the
name of the function callback that you're going to write. Additionally,
You can declare an alias name of subcommand with fourth argument, if needed.

After the ENTRY is defined, you need to implement the callback. It takes
four arguments: argc, argv, the command structure associated with the
callback, and the plug-in structure that contains that command. The
prototype looks like this:

  ```c
  int f(int argc, char **argv, struct command *cmd, struct plugin *plugin);
  ```

The argc and argv are adjusted from the command line arguments to start
after the sub-command. So if the command line is "nvme foo --option=bar",
the argc is 1 and argv starts at "--option".

You can then define argument parsing for your sub-command's specific
options then do some command specific action in your callback.

### Add a new plugin

The nvme-cli provides macros to make define a new plug-in simpler. You
can certainly do all this by hand if you want, but it should be easier
to get going using the macros. To start, first create a header file
to define your plugin. This is where you will give your plugin a name,
description, and define all the sub-commands your plugin implements.

There is a very important order on how to define the plugin. The following
is a basic example on how to start this:

File: foo-plugin.h
```c
#undef CMD_INC_FILE
#define CMD_INC_FILE plugins/foo/foo-plugin

#if !defined(FOO) || defined(CMD_HEADER_MULTI_READ)
#define FOO

#include "cmd.h"

PLUGIN(NAME("foo", "Foo plugin"),
	COMMAND_LIST(
		ENTRY("bar", "foo bar", bar)
		ENTRY("baz", "foo baz", baz)
		ENTRY("qux", "foo quz", qux)
	)
);

#endif

#include "define_cmd.h"
```

In order to have the compiler generate the plugin through the xmacro
expansion, you need to include this header in your source file, with
pre-defining macro directive to create the commands.

To get started from the above example, we just need to define "CREATE_CMD"
and include the header:

File: foo-plugin.c
```c
#include "nvme.h"

#define CREATE_CMD
#include "foo-plugin.h"
```

After that, you just need to implement the functions you defined in each
ENTRY, then append the object file name to the Makefile's "OBJS".
