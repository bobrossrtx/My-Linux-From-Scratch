# Software-Building-HOWTO
## 1. Introduction
Many software packages for the various flavors of UNIX and Linux come as compressed archives of source files. The same package may be "built" to run on different target machines, and this saves the author of the software from having to produce multiple versions. A single distribution of a software package may thus end up running, in various incarnations, on an intel box, a DEC Alpha, a RISC workstation, or even a mainframe. Unfortunately, this puts the responsibility of actually "building" and installing the software on the end-user, the de facto "system administrator", the fellow sitting at the keyboard -- you. Take heart, though, the process is not nearly as terrifying or mysterious as it seems, as this guide will demonstrate.

## 2. Unpacking the files
You have downloaded or otherwise acquired a software packed. Most likely it is archived (tarred) and compressed (gzipped), in .tar.gz or .tgz form (familiarly known as a "tarball"). First copy it to a working directory. Then untar and gunzip it. The appropriate command for this is `tar xzvf filename`, where filename is the name of the software file, of course. The de-archiving process will usually install the appropriate files in subdirectories it will create. Note that if the package name has a .Z suffix, the above procedure will serve just as well, though running uncompresses, followed by `tar xvf` also works. You may preview this process by a `tar tzvf filename`, which lists the files in the archive without actually unpacking them.

The above method of unpacking "tarballs" is equivalent to either of the following:

 - gzip -cd filename | tar xvf -
 - gunzip -c filename | tar xvf -

(The '-' causes the tar command to take its input from `stdin`.)

Source files in the new bzip2 (.bz2) format can be unarchived by a `bzip2 -cd filename | tar xvf -`, or, more simply by a `tar xyvf filename`, assuming that tar has been appropriately [athced (refer to the [Bzip2 HOWTO](ftp://metalab.unc.edu/pub/Linux/docs/HOWTO/mini/Bzip) for details). Debian Linux uses a different patch for tar, one written by Hiroshi Takekawa, so that the -I, --bzip2, --bunzip2 options work with that particular tar version.

[Many thanks to R. Brock Lynn and Fabrizio Stefani for corrections and updates on the above information.]

Sometimes the archived file must be untarred and installed from the user's home directory or perhaps in certain other directories, such as /, /usr/src, or /opt, as specified in the package's config info. Should you get an error message attempting to untar it, this may be the reason. Read the package docs, especially the README and/or Install files, if present, and edit the config files and/or MakeFiles as necessary, consistent with the installation instructions, Note that you would not ordinarily alter the Imake file, since this could have unforeseen consequences. Most software packages permit automating this process by running make install to emplace the binaries in the appropriate system areas.

- You might encounter shar files, or shell archives, especially in the source code newsgroups on the internet. These remain in use because they are readable to humans, and these permit newgroup moderators to sort through them and reject unsuitable ones. They may be unpacked by the `unshar filename.shar` command. Otherwise, the procedure for dealing with them is the same as for "tarballs".

- Some source archives have been processed using nonstandard DOS, Mac, or even Amiga compression utilities such as SIP, arc, lha, arj, zoo, rar, and shk. Fortunately, [Sunsite](https://www.ibiblio.org/) and other places have Linux uncompression utilities that can deal with most or all of these.

Occasionally, you may need to update or incorporate bug fixes into the unarchived source files using a patch or diff file that lists the changes. The doc files and/or README file that lists the changes. The normal syntax for invoking Larry Wall's powerful patch utility is `patch < patchfile`.

You may now proceed to the build stage of the process.

## 3. Using Make
The Makefile is the key to the build process. In its simplest form, a Makefile is a script for compiling or building the "binaries", the executable portions of a package. The Makefile can also provide a means of updating a software package without having to recompile every single source file in it, but that is a different story (or a different article).

At some point, the Makefile launches cc or GCC. This is actually a preprocessor, a C (or C++) compiler, and a linker, invoked in that order. This process converts the source into the binaries, the actual executables.

Invoking make usually involves just typing make. This generally builds all the necessary executable files for the package in question. However, make can also do other tasks, such as installing the files in their proper directories (make install) and removing stale object files (make clean). Running `make -n` permits previewing the build process, as it prints out all the commands that would be triggered by a make, without actually executing them.

Only the simples software uses a generic Makefile. More complex installations require tailoring the Makefile according to the location of libraries, include files, and resources on your particular machine. This is especially the case when the build needs `X11` libraries to install Imake and xmkmf to accomplish this task.

An `Imakefile` is, to quote the man page, a "template" Makefile. The imake utility constructs a Makefile appropriate for your system from the I makefile. In almost all cases, however, you would run `xmkmf`, a shell script that invokes imake, a front end for it. Check the README or INSTALL file included in the software archive for specific instructions. (If after de-archiving the source files, there is an `Imake` file present in the base directory, this is a dead giveaway that xmkmf should be run.) Read the `Imake` and `xmkmf` man pages for a more detailed analysis of the procedure.

Be aware that `xmkmf` and `make` may need to be invoked as root, especially when doing a `make install` to move to binaries to the `/usr/bin` or `/usr/local/bin` directories. Using make as an ordinary user without root privileges will likely result in write access denied error messages because you lack write permissions to system directories. Check also that the binaries created have the proper execute permissions for you and any other appropriate users.

Invoking `xmkmf` uses the `Imake` file to build a new Makefile appropriate for your system. You would normally invoke `xmkmf` with the `-a` argument, to automatically do a make Makefiles, make includes, and make depend. This sets the variables and defines the library location for the compiler and linker. Sometimes there will be no `Imake` file, instead, there will be an `INSTALL` or `configure` script that will accomplish this purpose. Note that if you run `configure`, it should be invoked as `./configure` to ensure that the correct `configure` script in the current directory is called. In most cases, the `README` file included with the distribution will explain the install procedure.

It is usually a good idea to visually inspect the `Makefile` that `xmkmf` or one of the install scripts builds, The Makefile will normally be correct for your system, but you may occasionally be required to "tweak" it or correct errors manually.

Installing the freshly built binaries into the appropriate system directories is usually a matter of running `make install` as root. The usual directories for system0wide binaries on modern Linux distributions are `/usr/bin`, `/usr/X11R6/bin`, and `/usr/local/bin`. The preferred directory for new packages is `/usr/local/bin`, as this will keep separate binaries not part of the original Linux installation.

Packages originally targeted for commercial versions of UNIX may attempt to install in the `/opt` or another unfamiliar directory. This will, of course, result in an installation error if the intended installation directory does not exist. The simplest way to deal with this is to create, as root, an `/opt` directory, let the package install there, then add that directory to the `PATH` environmental variable. Alternatively, you may create symbolic links to the `/usr/local/bin` directory.

Your general installation procedure will therefore be:

- Read the `README` file and other applicable docs.
- Run `xmkmf -a`, or the `INSTALL` or `configure` script.
- Check the Makefile.
- If necessary, run `make clean`,  `make Makefiles`, `make includes`, and `make depend`.
- Run `make`.
- Check the file permissions.
- If necessary run `make install`.

Notes:

- You would not normally build a package as root. Doing an `su` to root is only necessary for installing the compiled binaries into system directories.
- After becoming familiar with make and its uses, you may wish to add additional optimization options passed to `GCC` in the standard `Makefile` included or created in the package you are installing. Some of these common options are `-O2`, `-formit-frame-pointer`, `-funroll-loops`, and `-mpentium` (if you are running a Pentium CPU). Use caution and good sense when modifying a `Makefile`!
- After the make creates the binaries, you may wish to `strip` them. The strip command removes the symbolic debugging information from the binaries and reduces their size, often drastically. This also disables debugging, of course.
- The [Pack Distribution Project](http://sunsite.auc.dk/pack/) offers a different approach to creating archived software packages, based on a set of Python scripting tools for managing symbolic links to files installed in a separate collection directory. These archives are ordinary tarballs, but they install `/coll` and `/pack` directories. You may find it necessary to download the Pack-Collection from the above site should you ever run across one of these distributions.


## 4. Prepackaged binaries
### 4.1 What's wrong with RPMs?
Manually building and installing packages from source is apparently so daunting a task for some Linux users that they have embraced the popular rpm and deb or the newer Stampede slp package formats. While it may be the case an rpm install normally runs as smoothly and as fast as software installs in a certain another notorious operating system, some thought should certainly be given to the disadvantages of self-installing, prepackaged binaries.

First, be aware that software packages are normally released first as "tarballs", and that prepackaged binaries follow days, weeks, even months later. A current rpm package is typically at least a couple of minor versions behind the latest "tarball". So, if you wish to keep up with all the bleeding edge' software you might not wish to wait for an rpm or deb to appear. Some less popular packages may never be rpm'ed.

Second, the "tarball" package may well be more complete, have more options, and lent itself better to customization and tweaking. The binary rpm version may be missing some of the functionality of the full release. Source rpm's contain the full source code and are equivalent to the corresponding "tarballs", and they likewise need to be built and installed using either of the `rpm --recompile packagename.rpm` or `rpm --rebuild packagename.rpm` options.

Third, some prepackaged binaries will not properly install, and even if they do install, they could crash and core-dump. They may depend on different library versions than are present in your system, or they may be improperly prepared or just plain broken. In any case, when installing an rpm or deb you necessarily trust the expertise of the person who has packaged it.

Finally, it helps to have the source code on hand, to be able to tinker with and learn from it. It is much more straightforward to have the source in the archive you are building the binaries from, and not in a separate source rpm.

Installing an rpm package is not necessarily a no-brainer. If there is a dependency conflict, an rpm install will fail. Likewise, should the rpm require a different version of libraries than the ones present on your system, then install may not work, or even if you create symbolic links to the missing libraries from the ones in place. Despite their convenience, rpm installs often fail for the same reasons "tarball" ones do.

You must install rpm's and deb's as root, to have the necessary write permissions, and this opens a potentially serious security hole, as you may inadvertently clobber system binaries and libraries, or even install a Trojan horse that might wreak havoc upon your system. It is therefore important to obtain rpm and deb packages from a "trusted source". In any case, you should run  a 'signature check' (against the MD5 checksum) on the package, `rpm --checksig packagename.rpm`, before installing. Likewise highly recommended is running `rpm -K --nopgp packagename.rpm`. The corresponding commands for deb packages are `dpkg -I | --info packagename.deb` and `dpkg -e | --control packagename.deb`.

- `rpm --checksig gnucash-1.1.23-4.i386.rpm`<br>gnucash-1.1.23-4.i386.rpm: size md5 OK

- `-K --nopgp gnucash-1.1.23-4.i386.rpm`<br>gnucash-1.1.23-4.i386.rpm: size md5 OK

For the truly paranoid (and, in this case, there is much to be said for paranoia), there are the unrpm and rpmunpack utilities available from the [Sunsite utils/package directory](http://www.ibiblio.org/pub/Linux/utils/package/) for unpacking and checking the individual components of the packages.

In their most simple form, the commands `rpm -i packagename.rpm` and `dpkg --install packagename.deb` automatically unpack and install the software. Exercise caution, though, since using these commands blindly may be dangerous to your system's health!

Note that the above warnings also apply, though to a lesser extent, to Slackware's pkgtool installation utility. All "automatic" software installations require caution.

The alien program allows conversion between the rpm, deb, Stampede slp, and tar.gz package formats. This makes these packages accessible to all Linux distributions.

Carefully read the man pages for the rpm and dpkg commands, and refer to the TFUG's [Quick Guide to Red Hat's Package Manager](http://www.tfug.org/helpdesk/linux/rpm.html), and the [Debian Package manager](https://www.dpkg.org/) for more detailed information.

### 4.2 Problems with RPMs: an example
Jan Hubicka wrote a very nice fractal package called xaos. At his [home page](http://www.paru.cas.cz/~hubicka/XaoS), both `.tar.gz` and `rpm` packages are available. For the sake of convenience, let us try the rpm version, rather than the "tarball".

Unfortunately, the rpm of xaos fails to install. Two separate rpm versions misbehave.

```bash
rpm -i --test Xaos-3.0-1.i386.rpm
    error: failed dependencies:
        libslang.so.0 is needed by XaoS-3.0-1
        libpng.so.0 is needed by XaoS-3.0-1
        libaa.so.1 is needed by XaoS-3.0-1
```
```bash
rpm -i --test Xaos-3.0-8.i386.rpm
    error: failed dependencies:
        libaa.so.1 is needed by xaos-3.0-8
```

The strange thing is that linslang.so.0, libpng.so.0, and libaa.10.1 are all present in `/usr/lib` on the system tested. The RPMs of xaos mush have been built with slightly different versions of those libraries, even if the release numbers are identical.

As a test, let us try installing `xaos-3.0-8.i386.rpm` with the --nodeps option to force the install. A trial run of xaos crashes.

> xaos: error in loading shared libraries:

Let us stubbornly try to get to the bottom of this, Running lld on the xaos binary to find its library dependencies shows all the necessary shared libraries present, Running nm on the `/usr/lib/libaa.so.1` library to list its symbolic references shows that is indeed missing __fabsl. Of course, the absent reference could be missing from one of the other libraries... There is nothing to be done about that short of replacing one or more libraries.

Enough! Download the "tarball", `XaoS-s.0.tar.gz`, available from the home page. Try running it, Running `./configure`, `make`, and finally (as root) `make install`, works flawlessly.

This is one of a number of examples of prepackaged binaries being more trouble than they are worth.

### 5. Termcap and Terminfo Issues
According to its man page, "terminfo is a database describing terminals, used by screen-oriented programs...". It defines a generic set of control sequences (escape codes) used to display text on terminals, and makes possible support for different terminal hardware without the need for special drivers. The terminfo libraries are located in `/usr/share/terminfo` on modern Linux distributions.

The terminfo database has largely supplanted the older termcap and the totally obsolete termlib ones. This is usually of no concern for program installations except when dealing with a package that requires termcap.

Most Linux distributions now use terminfo, but still retain the older termcap libraries for compatibility with legacy applications (see `/etc/termcap`). Sometimes there is a special compatibility package that needs to be installed to facilitate the use of termcap linked binaries. Very occasionally, an #define termcap statement might need to be commented out of a source file. Check the appropriate doc files for your particular distribution for definitive information on this.

## 6. Backward Compatibility With a.out Binaries
In very few cases, it is necessary to use a.out binaries, either because the source code is not available or because it is not possible to build new ELF binaries from the source for some reason.

As it happens, ELF installations almost always have a complete set of a.out libraries in the `/usr/i486-linuxaout/lib` directory. The numbering scheme for a.out libraries differs from that of ELF ones, cleverly avoiding conflicts that could cause confusion. The a.out binaries should therefore be able to find the correct libraries at runtime, but this might not always be the case.

Note that the kernel needs to have a.out support built into it, either directory or as a loadable module. It may be necessary to rebuild the kernel to enable this. Moreover, some Linux distributions require the installation of a special compatibility package, such as Debian's xcompat for executing a.out X applications.

### 6.1 An Example
Jerry Smith wrote a very handy rolodex program some years back. It uses the Motif libraries but fortunately is available as a statically linked binary in a.out format. Unfortunately, the source requires numerous tweaks to rebuild using the lesstif libraries. Even more, unfortunately, the a.out binary bombs on an ELF system with the following error message.

```
	xrolodex: can't load library '//lib/libX11.so.3'
	No such library
```

As it happens, there is such a library in `/usr/i486-linuxaout/lib`, but xrolodex is unable to locate it at runtime, The simple solution is to provide a symbolic link to the `/lib` directory:

```bash
ln -s /usr/i486-linuxaout/lib/X11.so.3.1.0 libX11.so.3
```

it turns out to be necessary to provide similar links for the libXt.so.3 and libc.so.4 libraries. This needs to be done as root, of course. Note that you should make absolutely certain you will not overwrite or cause version number conflicts with pre-existing libraries. Fortunately, the new ELF libraries have higher version numbers than the older a.out ones, to anticipate and forestall just such problems.

After creating the three links, xrolodex runs fine.

The xrolodex package was originally posted on [Spectro](http://www.spectro.com/), but seems to vanish from there. It may currently be downloaded from [Sunsite](http://www.ibiblio.org/pub/Linux/apps/reminder/rolodex/!INDEX.short.html) as a tar.Z format source file [512k].

## 7. Troubleshooting
If xmkmf and/or make succeeded without errors, you may proceed to the next section. However, in "real life", few things work right the first time. This is when your resourcefulness is put to the test.

### 7.1 Link Errors
- Supposes make fails with a Link error: -lX11: No such file or directory, even after xmkmf has been invoked. This may mean the Imake file was not set up properly. Check the first part of the Makefile for lines such as:

```make
LIB=		-L/usr/X11/lib
INCLUDE=	-I/usr/X11/include/X11
LIBS=		-lX!! -lc -lm
```

T -L and -I switches tell the compiler and linker where to look for the library and include files, respectively. In this example, the X11 libraries should be in the `/usr/X11/lib` directory, and the X11 include files should be in the `/usr/X11/include/X11` directory. If this is incorrect for your machine, make the necessary changes to the Makefile and try the make it again.

- Undefined references to math library functions, such as the following:<br>```		/tmp/cca011551.o(.text+0x11): undefined reference to `cos'```<br>The fix for this is to explicitly link the math library, by adding an -lm to the LIB or LIBS flags in the Makefile (see the previous example).

- Yet another thing to try if xmkmf fails is the following script:<br>```		make -DUseInstalled -I/usr/ZX386/lib/X11/config```<br>
This is a sort of bare-bones equivalent of xmkmf.

- In a very few cases, running ldconfig as root may be the solution:<br>`# ldconfig` updates the shared library symbolic links. This may not be necessary.

- Some Makefiles use unrecognized aliases for libraries present in your system. For example, the build may require `libX11.so.6`, but there exists no such file or link in `/usr/X11R6/lib`. Yet, there is a `libX11.so.6.1`. The solution is to do a `ln -s /usr/X11R6/lib/libX11.so.6.1 /usr/X11R6/lib/libX11.so.6`, as root. This may need to be followed by a ldconfig.

- Sometimes the source needs the older release X11R5 libraries to build. If you have the R5 libs in `/usr/X11R6/lib` (you were given the option of having them when first installing Linux), then you need only ensure that you have the links that the software needs to build. The R5 libs are named `libX11.so.3.1.0`, `libXaw.so.3.1.0`, and `libXt.so.3.1.0`. You generally need links, such as `libX11.so.3 -> libX11.so.3.1.0`. Possibly the software will also need a link of the form `libX11.so -> libX11.so.3.1.0`.Of course, to create a "missing" link, use the command `ln -s libX11.so.3.1.0 libX11.so`, as root.

- Some packages will require you to install an updated version of one or more libraries. For example, the 4.x versions of the StarOffice suite from StarDivision GmbH were notorious for needing libc version 5.4.4 or greater. Even the more recent StarOffice 5.0 will not run after installation with the new glibc 2.1 libs. Fortunately, the newer StarOffice 5.1 solves problems. If running an older version of StarOffice you would, as root, need to copy one or more libraries to the appropriate directories, remove the old libraries, then reset the symbolic links (check the latest version of the StarOffice mini howto for more information on this_). <strong>Caution: Exercise extreme care in this, as you can render your system nonfunctional if you screw up.</strong> You can usually find the latest updated libraries at [Sunsite](http://www.ibiblio.org/pub/Linux/libs/).

### 7.2 Other Problems
- An installed Perl or shell script gives you `No such file or directory` error message. In this case, check the file permissions to make sure the file is executable and check the file header to ascertain whether the shell or program invoked by the script is in the place specified. For example, the script may begin with:
```perl
#!/usr/local/bin/perl
```
> If perl is in fact installed in your `/usr/bin` directory instead of `/usr/local/bin` one, then the script will not run. There are two methods of correcting this. The script file header may be changed to `#!/usr/bin/perl`, or a symbolic link to the correct directory may be added, `ln 0s /usr/bin/perl /usr/local/bin/perl`.

- Some X11 software requires the Motif libraries to build. The standard Linux distributions do not have Motif libraries installed, and at present Motif costs an extra $100-$200 (through the freeware [Lesstif](http://lesstif.org/) also works in may cases). If you need Motif to build a certain package, but lack the Motif libraries, it may be possible to obtain statically linked binaries. Static linking incorporates the library routines in the binaries themselves. This results in much larger binary files, but the code will run on systems lacking the libraries.<br>When a package requires libraries not present on your system for the build, it will result in link errors ( undefined reference errors). The libraries may be expensive proprietary ones or difficult to find for some other reason. In that case, obtaining a statically linked binary either from the author of the package or from a Linux user group may be the easiest to implement the fix.

- Running a configure script creates a strange Makefile, one seemingly unrelated to the package you are attempting to build. This means the wrong configure ran, one found somewhere else in your path. Always invoke configure as `./configure` to prevent this.

- Most Linux distributions have changed over to the libc 6 / glibc 2 libraries from the older libc 5. Precompiled binaries that worked with the older library may bomb if you have upgraded the library. The solution is to either recompile the applications from the source or to obtain newer precompiled binaries.<br>Note that there are some minor incompatibilities between glibc versions, so a binary built with glibc 2.1 may not work with glibc 2.0, and vice versa.

- Sometimes it is necessary to remove the -asni option from the compile flags in the Makefile. This enables gcc's extra, non-ANSI features, and allows building packages that require these extensions.

- Some programs require having a setuid root, in order to run with root privileges. The command to implement this is `chmod u+s filename`, as root (note that the program must already be owned by root). This has the effect of setting the setuid bit in the file permissions. This issue comes up when the program accesses the system hardware, such as a modem or CD ROM drive, or when the SVGA libs are invoked from console mode, as in one particularly notorious emulation package. If a program works when run by root, but gives access to denied error messages to an ordinary user, suspect this as the cause.

> **Warning:** A program with a setuid as root may pose a security risk to your system. The program runs with root privileges and thus has the potential for doing significant damage, Make certain that you know what the program does, by looking at the source if possible, before setting the setuid bit.

### 7.3 Tweaking and fine-tuning
You may wish to examine the Makefile to make certain that the best compilation options for your system are invoked. For example, setting the -O2 flag chooses the highest level of optimization, and the -fomit-frame-pointer flag results in a smaller binary (though debugging will then be disabled). **Do not play around with this unless you know what you are doing, and in any case, not until after the trial build works.**

### 7.4 Where to go for more help
In my experience, perhaps 25% of applications build "right out of the box". Another 50% or so can be "persuaded" to build with an effort ranging from trivial to herculean. That still means a significant number of packages will not build no matter what. Even then, the Intel ELF and/or a.out binaries for these might possibly be found at [Sunsite](http://metalab.unc.edu/). [Red Hat](https://www.redhat.com/en) and [Debian](http://www.debian.org/) have extensive archives of prepackaged binaries of most of the popular Linux software., Perhaps the author of the software can supply the binaries compiled for your particular flavor of the machine.

**Note that if you obtain precompiled binaries, you will need to check for compatibility with your system:**
- **The binaries must run on your hardware (i.e., Intel X86).**
- **The binaries must be compatible with your kernel (i.e., a.out or ELF).**
- **Your libraries must be up to date.**
- **Your system must have the appropriate installation utility (rpm or deb).**

If all else fails, you may find help in the appropriate newsgroups, such as (comp.os.linux.x)[news://comp.os.linux.x] or (comp.os.linux.development)[news://comp.os.linux.development].

If nothing at all works, at least you gave it your best effort, and you learned a lot.