# Software-Building-HOWTO
## 1. Introduction
Many software packages for the various flavors of UNIX and Linux come as compressed archives of source files. The same package may be "built" to run on different target machines, and this saves the author of the software from having to produce multiple versions. A single distribution of a software package may thus end up running, in various incarnations, on an intel box, a DEC Alpha, a RISC workstation, or even a mainframe. Unfortunately, this puts the responsibility of actually "building" and installing the software on the end user, the de facto "system administator", the fellow sitting at the keyboard -- you. Take heart, though, the process is not nearly as terrifying or mysterious as it seems, as this guide will demonstrate.

## 2. Unpacking the files
You have downloaded or otherwise acquired a software packed. Most likely it is archived (tarred) and compressed (gzipped), in .tar.gz or .tgz form (familiarly known as a "tarball"). First copy it to a working directory. Then untar and gunzip it. The appropriate command for this is `tar xzvf filename`, where filename is the name of the software file, of course. The de-archiving process will usually install the appropriate files in subdirectories it will create. Note that is the package name has a .Z suffix, the the above provedure will serve just as well, though running uncompress, followed by `tar xvf` also works. You may preview this process by a `tar tzvf filename`, whick lists the files in the archive withoutactually unpacking them.

The above method of unpacking "tarballs" is equivalent to either of the following:

 - gzip -cd filename | tar xvf -
 - gunzip -c filename | tar xvf -

(The '-' causes the tar command to take its input from `stdin`.)

Source files in the new bzip2 (.bz2) format can be unarchived by a `bzip2 -cd filename | tar xvf -`, or, more simply by a `tar xyvf filename`, assuming that tar has been appropiately [athced (refer to the [Bzip2 HOWTO](ftp://metalab.unc.edu/pub/Linux/docs/HOWTO/mini/Bzip) for details). Debian Linux uses a different patch for tar, one written by Hiroshi Takekawa, so that the -I, --bzip2, --bunzip2 options work with that particular tar version.

[Many thanks to R. Brock Lynn and Fabrizio Stefani for corrections and updates on the aboce information.]

Sometimes the archived file must be untarred and installed from user's home directorym or perhaps in a certain other directory, such as /, /usr/src, or /opt, as specified in the package's config info. Should you get an error message attempting to untar it, this may be the reason. Read the package docs, especially the README and/or Install files, if present, and edit the config files and/or MakeFiles as necessary, consistent with the installation instructions, Note that you would not ordinarily alter the Imake file, since this could have unforseen consequenses. Most software packages permit automating this process my running make install to emplace the binaries the appropriate system areas.

- You might encounter shar files, or shell archives, especvially in the source code newsgroups on the internet. These remain in use because they are readable to humans, nd this permits newgroup moderators to sort through them and reject unsuitable ones. The may be unpacked by the `unshar filename.shar` command. Otherwise the procedure for dealing with them is the same as for "tarballs".

- Some source archives have been processed using nonstandard DOS, Mac, or even Amiga compression utilities such as SIP, arc, lha, arj, zoo, rar, and shk. Fortunately, [Sunsite](https://www.ibiblio.org/) and other places have Linux uncompression utilites that can deal with most or all of these.

Occasionally, you may need to update or incorporate bug fixes into the unarchived source files using a patch or diff file that lists the changes. The doc files and/or README file that lists the changes. The normal syntax for invoking Larry Wall's powerfull patch utility is `patch < patchfile`.

You may now proceed to th ebuild stage of the process.

## 3. Using Make
The Makefile is the key to the build process. In its simplest form, a Makefile is a script for compiling or building the "binaries", the executable portions of a package. The Makefile can also provide a means of updating a software package without having to recompile every single source file in it, but that is a different story (or a different article).

At some point, the Makefile launchess cc or gcc. This is actually a preprocessor, a C (or C++) compiler, and a linker, invoked in that order. This process converts the source into the binaries, the actual executables.

Invoking make usually involves just yping make. This generally builds all the necessary executable files for the package in question. However, make can also do other tasks, such as installing the files in their proper directories (make install) and removing stale object files (make clean). Running `make -n` permits previewing the build process, as it prints out all the commands that would be triggered by a make, without catually executing them.

Only the simples software uses a generic Makefile. More complex installations require tailoring the Makefile according to the location of libraries, include files, and resources on yourparticular machine. This is especially the case when the build needs `X11` libraries to install Imake and xmkmf accomplishthis task.

An `Imakefile` is, to quote the man page, a "template" Makefile. The imake utility constructs a Makefile appropriate for your system from the I makefile. In almost all cases, howeverm you would run `xmkmf`, a shell script that invokes imake, a front end for it. Check the README or INSTALL file included in the software archive for specific instructions. (Ifm after dearchiving the source files, there is an `Imake` file present in the base directory, this is a dead giveaway that xmkmf should be run.) Read the `Imake` and `xmkmf` man pages for a more detailed analysis of the procedure.

Be aware that `xmkmf` and `make` may need to be invoked as root, especially when doing a `make install` to move to binaries to the `/usr/bin` or `/usr/local/bin` directories. Using make as an ordinary user without root privileges will likely result in write access denied error messages because you lack write permisions to system directories. Check also that the binaries created have the proper execute persmisions for you and any other appropriate users.

Invoking `xmkmf` uses the `Imake` file to build a new Makefile appropriate for your system. You would normally invoke `xmkmf` with the `-a` argument, to automatically do a make Makefiles, make includes, and make dpend. This sets the variables and defines the library location for the compiler and linker. Sometimes there will be no `Imake` file, instead there will be an `INSTALL` or `configure` script that will accomplish this purpose. Note that if you run `configure`, it should be invoked as `./configure` to ensure that the correct `configure` script in the current directory is called. In most cases, the `README` file included with the distriburion will explain the install procedure.

It is usually a good idea to visually inspect the `Makefile` that `xmkmf` or one of the install scripts builds, The Makefile will normally be correct for you system, but you may occasionally be required to "tweak" it or correct errors manually.

Installing the freshly built binaries into the appropriate system directories is usually a matter of running `make install` as root. The usual directories for system0wide binaries on modern Linux distributions are `/usr/bin`, `/usr/X11R6/bin`, and `/usr/local/bin`. The preferred directory for new packages is `/usr/local/bin`, as this will keep separate binaries not part of the original Linux installation.

Packages originally targeted for commercial versions of UNIX may attempt to install in the `/opt` or other unfamiliar directory. This will, of course, result in an installation error if the intended installation directiody does not exist. The simplest way to deal with this is to create, as root, an `/opt` directiory, let the package install there, then add that directory to the `PATH` environmental variable. Alternatively, you may create symbolic links to the `/usr/local/bin` directory.

Your general installation procedure will therefore be:

- Read the `README` file and other applicable docs.
- Run `xmkmf -a`, or the `INSTALL` or `configure` script.
- Check the Makefile.
- If necessary, run `make clean`,  `make Makefiles`, `make includes`, and `make depend`.
- Run `make`.
- Check the file permissions.
- If necessarym run `make install`.

Notes:

- You would not normally build a package as root. Doing an `su` to root is only necessary for installing the compiled binaries into system directories.
- After becoming familiar with make and its uses, you may wish to add additional optimization options passed to `gcc` in the standard `Makefile` included or created in the package you are installing. Some of these common options are `-O2`, `-formit-frame-pointer`, `-funroll-loops`, and `-mpentium` (if you are running a Pentium cpu). Use caution and good sense when modifying a `Makefile`!
- After the make creates the binaries, you may wish to `strip` them. The strip command removes the symbolic debugging information from the binaries and reduces their size, often drastically. This also disables debugging, of course.
- The [Pack Distribution Project](http://sunsite.auc.dk/pack/) offers a different approach to reating archived software packages, based on a set of Python scripting tools for managing symbolic links to files installed in a separate collection directiories. These archives are orinary tarballs, but they install `/coll` and `/pack` directories. You may find it necessary to downlad the Pack-Collection from the above site should you ever run across one of these distributions.


## 4. Prepackeged binaries
### 4.1 Whats wrong with rpms?
Manually building and installing packages from source is apparently so daunting a task for some Linux users that they have embroaced the popular rpm and deb or the newer Stampede slp package formats. While it may be the case an rpm install normally runs as smoothly and as fast as a software install in a certain other notorious operating system, some though should certanly be givern to the disadvantages of self-installing, prepackaged binaries.

First, be aware that software packages are normally release first as "tarballs", and that prepackaged binaries follow days, weeks, even months later. A current rpm package is typically at least a couple of miner version behind the latest "tarball". So, if you wish to keep up with all the 'bleeding edge' softwarem you might not wish to wait for an rpm or deb to appear. Some less popular packages may never be rpm'ed.

Second, the "tarball" package may well be more complete, have more options, and lent itself better to customization and tweaking. The binary rpm version may be missing some of the functionality of the full release. Source rpm's contain the full source code and are equivalent to the corresponding "tarballs", and they likewase need to be built and installed using either of the `rpm --recompile packagename.rpm` or `rpm --rebuild packagename.rpm` options.

Third, some prepackaged binaries will not properly install, and even if they do install, they could crash and core-dump. They may depend on different library versions than are present in your system, or they may be imporperly prepared or just plain broken. In any case, when installing an rpm or deb you necessarily trust the expertise of the person who have packaged it.

Finally, it helps to have the source code on hand, to be able to tinker with and learn from it. It is much more straightforward to have the sourc in the archive you are building the binaries from, and not in a separate source rpm.

Installing an rpm package is not necessarialy a no-brainer. If there is a dependency conflict, an rpm install will fail. Likewise, should the rpm require a different version of libraries than the ones present on your system, the install may not work, or even if you create symbolic links to the missing lbraries from the ones in place. Despite their convenience, rpm installs often fail for the same reasons "tarball" ones do.

You must install rpm's and deb's as root, in order to have the necessary write permsissions, and this opens a potentially serious ecurity hole, as you may inadvertenly clobber system binaries and libraries, or even install a Trojan horse that might wreak havoc upon your system. It is therefore important to obtain rpm and deb packages from a "trusted source". In any case, you should run  a 'signature check' (against the MD5 checksum) on the package, `rpm --checksig packagename.rpm`, before installing. Likewise highly recommended is running `rpm -K --nopgp packagename.rpm`. The corresponding commands for deb packages are `dpkg -I | --info packagename.deb` and `dpkg -e | --control packagename.deb`.

- `rpm --checksig gnucash-1.1.23-4.i386.rpm`<br>gnucash-1.1.23-4.i386.rpm: size md5 OK

- `-K --nopgp gnucash-1.1.23-4.i386.rpm`<br>gnucash-1.1.23-4.i386.rpm: size md5 OK

For the truly paranoid (and, in this case there is much to be said for paranoia), there are the unrpm and rpmunpack utilites available from the [Sunsite utils/package directory](http://www.ibiblio.org/pub/Linux/utils/package/) for unpacking and checking the individual components of the packages.

In there most simple form, the commands `rpm -i packagename.rpm` and `dpkg --install packagename.deb` automatically unpack and install the software. Exercise caution, though, since using these commands blinly may be dangrous to your system's health!

Note that the above warnings also apply, though to a lesser extent, to Slackware's pkgtool installation utility. All "automatic" software installations require caution.

The alien program allows conversion between the rpm, deb, Stampede slp, and tar.gz package formats. This makes these packages accessible to all Linux distributions.

Carefully read the man pages for the rpm and dpkg commands, and refer the the TFUG's [Quick Guide to Red Hat's Package Manager](http://www.tfug.org/helpdesk/linux/rpm.html), and the [Debian Package manager](https://www.dpkg.org/) for more detailed information.

### 4.2 Problems with rpms: an example
Jan Hubicka wrote a very nice fractal package called xaos. At his [home page](http://www.paru.cas.cz/~hubicka/XaoS), both `.tar.gz` and `rpm` packages are available. For the sake of convenience, let us try the rpm version, rather than the "tarball".

Unfortunaly, the rpm of xaos fails to install. Two separate rpm versions misbehave.

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

The strange thing is that linslang.so.0, libpng.so.0, and libaa.10.1 are all present in `/usr/lib` on the system tested. The rpms of xaos mush have been built with slightly different versions of those libraries, even if the release numbers are identical.

As a test, let us try installing `xaos-3.0-8.i386.rpm` with the --nodeps option to force the install. A trial run of xaos crashes.

> xaos: error in loading shared libraries:

Let us stubbornly try to get to the bottom of this, Running lld on the xaos binary to find its library dependencies shows all the necessary shared libraries present, Running nm on the `/usr/lib/libaa.so.1` library to list its symbolic references shows that is is indeed missing __fabsl. Of course, the absent reference could be missing from one of the other libraries... There is nothing to be done abouth thetm short of replacing one or more libraries.

Enough! Download the "tarball", `XaoS-s.0.tar.gz`, available from the home page. Try running it, Running `./configure`, `make`, and finally (as root) `make install`, works flawlessly.

This is one of an number of examples of prepackaged binaries being more trouble than they are worth.
