# Generating build dependencies automatically

As we start updating packages for the next Red Hat distro, I'd like to see
packages start to make use of build dependencies. Basically build
dependencies are just like install dependencies, but they are resolved
against the build system just after parsing the spec file. Syntactically,
build dependencies look just like install dependencies in a spec file with
"Build" prefixed:

```
	BuildPreReq:
	BuildRequires:
	BuildConflicts:
```

All the above dependencies include versions, files, existence/range tests, etc.
The build dependency checking can also be turned off with --nodeps if necessary
just like install dependency checking can. Eventually, build dependencies will
be automated in rpm, but the major impediment to that effort is the engineering
required to maintain the pretense that src rpm's are "noarch".

Meanwhile, I've added a package called "InDependence-1.0" to powertools-6.2
that may be of use in detecting build dependencies that can be added to
spec files as part of rebuilding packages for Red Hat 6.2.

Here's a short example of how to generate the package/file names that were used
while building gnorpm using InDependence:

```
	rpm -U /mnt/redhat/comps/powertools/6.2/i386/InDependence-1.0-3.i386.rpm
	rpm -i /mnt/redhat/comps/dist/6.2/SRPMS/gnorpm-0.9-11.src.rpm
	cd /usr/src/redhat/SPECS
	dep -detail rpmbuild -ba gnorpm.spec >& xxx
	...
	(the build will take longer since both dep and strace are pigs)
	...
	grep -- '::' xxx > yyy
```

```
Aside:	The dep perl wrapper is a "pig" only because it's exec'ing
		rpm -qf <filename>
        in order to turn filenames into package names. There are easier/faster
	ways to get this information...

	There's no way to speed up the
		/sbin/strace -q -etrace=open,execve -o ...
	command itself. The eventual implementation in rpm will snatch the
	same open/execve syscalls using LD_PRELOAD.

	Patches cheerfully accepted :-)
```

Here's what's in yyy (\<packagename\>::\<filename\> format):

```
	ORBit-devel-0.4.95-2::/usr/bin/orbit-config 
	XFree86-libs-3.3.5-6::/usr/X11R6/lib/libICE.so.6 
	XFree86-libs-3.3.5-6::/usr/X11R6/lib/libSM.so.6 
	XFree86-libs-3.3.5-6::/usr/X11R6/lib/libX11.so.6 
	XFree86-libs-3.3.5-6::/usr/X11R6/lib/libXext.so.6 
	audiofile-0.1.9-1::/usr/lib/libaudiofile.so.0 
	autoconf-2.13-5::/usr/bin/autoconf 
	autoconf-2.13-5::/usr/bin/autoheader 
	autoconf-2.13-5::/usr/share/autoconf/acgeneral.m4 
	autoconf-2.13-5::/usr/share/autoconf/autoconf.m4f 
	automake-1.4-5::/usr/bin/aclocal 
	automake-1.4-5::/usr/bin/automake 
	bash-1.14.7-16::/bin/sh 
	bash-1.14.7-16::/etc/bashrc 
	binutils-2.9.1.0.23-7::/usr/bin/strip 
	binutils-2.9.1.0.23-7::/usr/lib/libbfd-2.9.1.0.24.so 
	binutils-2.9.1.0.23-7::/usr/lib/libopcodes-2.9.1.0.24.so 
	bzip2-0.9.5c-1::/usr/lib/libbz2.so.0 
	dev-2.7.10-2::/dev/null 
	diffutils-2.7-16::/usr/bin/cmp 
	egcs-1.1.2-25::/usr/bin/gcc 
	egcs-1.1.2-25::/usr/lib/gcc-lib/i386-redhat-linux/egcs-2.91.66/specs 
	esound-0.2.14-1::/usr/lib/libesd.so.0 
	file-3.27-3::/usr/bin/file 
	file-3.27-3::/usr/share/magic 
	fileutils-4.0-8::/bin/chgrp 
	fileutils-4.0-8::/bin/chmod 
	fileutils-4.0-8::/bin/chown 
	fileutils-4.0-8::/bin/cp 
	fileutils-4.0-8::/bin/ln 
	fileutils-4.0-8::/bin/ls 
	fileutils-4.0-8::/bin/mkdir 
	fileutils-4.0-8::/bin/mv 
	fileutils-4.0-8::/bin/rm 
	fileutils-4.0-8::/usr/bin/install 
	findutils-4.1-32::/usr/bin/xargs 
	gawk-3.0.4-1::/bin/awk 
	gawk-3.0.4-1::/bin/gawk 
	gettext-0.10.35-13::/usr/bin/xgettext 
	glib-1.2.5-1::/usr/lib/libglib-1.2.so.0 
	glib-1.2.5-1::/usr/lib/libgmodule-1.2.so.0 
	glib-devel-1.2.5-1::/usr/bin/glib-config 
	glibc-2.1.2-13::/etc/localtime 
	glibc-2.1.2-13::/etc/nsswitch.conf 
	glibc-2.1.2-13::/lib/ld-linux.so.2 
	glibc-2.1.2-13::/lib/libc.so.6 
	glibc-2.1.2-13::/lib/libcrypt.so.1 
	glibc-2.1.2-13::/lib/libdb.so.2 
	glibc-2.1.2-13::/lib/libdl.so.2 
	glibc-2.1.2-13::/lib/libm.so.6 
	glibc-2.1.2-13::/lib/libnsl.so.1 
	glibc-2.1.2-13::/lib/libnss_dns.so.2 
	glibc-2.1.2-13::/lib/libnss_files.so.2 
	glibc-2.1.2-13::/lib/libnss_nis.so.2 
	glibc-2.1.2-13::/lib/libnss_nisplus.so.2 
	glibc-2.1.2-13::/lib/libresolv.so.2 
	glibc-2.1.2-13::/usr/bin/ldd 
	gnome-libs-1.0.54-1::/usr/lib/libart_lgpl.so.2 
	gnome-libs-1.0.54-1::/usr/lib/libgnome.so.32 
	gnome-libs-1.0.54-1::/usr/lib/libgnomesupport.so.0 
	gnome-libs-1.0.54-1::/usr/lib/libgnomeui.so.32 
	gnome-libs-devel-1.0.54-1::/usr/bin/gnome-config 
	grep-2.3-2::/bin/egrep 
	grep-2.3-2::/bin/fgrep 
	grep-2.3-2::/bin/grep 
	gtk+-1.2.5-2::/usr/lib/libgdk-1.2.so.0 
	gtk+-1.2.5-2::/usr/lib/libgtk-1.2.so.0 
	imlib-1.9.7-1::/usr/lib/libgdk_imlib.so.1 
	libghttp-1.0.4-1::/usr/lib/libghttp.so.1 
	libtool-1.3.3-1::/usr/bin/libtoolize 
	libtool-1.3.3-1::/usr/share/libtool/config.guess 
	libtool-1.3.3-1::/usr/share/libtool/config.sub 
	libtool-1.3.3-1::/usr/share/libtool/ltconfig 
	libtool-1.3.3-1::/usr/share/libtool/ltmain.sh 
	libxml-1.4.0-1::/usr/lib/libxml.so.1 
	libxml-devel-1.4.0-1::/usr/bin/xml-config 
	m4-1.4-12::/usr/bin/m4 
	make-3.77-6::/usr/bin/make 
	mktemp-1.5-1::/bin/mktemp 
	net-tools-1.53-1::/bin/hostname 
	patch-2.5-9::/usr/bin/patch 
	rootfiles-5.2-5::/root/.bashrc 
	rpm-3.0.4-0.16::/bin/rpm 
	rpm-3.0.4-0.16::/usr/lib/librpm.so.0 
	rpm-3.0.4-0.16::/usr/lib/rpm/find-provides 
	rpm-3.0.4-0.16::/usr/lib/rpm/find-requires 
	rpm-3.0.4-0.16::/usr/lib/rpm/macros 
	rpm-3.0.4-0.16::/usr/lib/rpm/rpmpopt 
	rpm-3.0.4-0.16::/usr/lib/rpm/rpmrc 
	sed-3.02-4::/bin/sed 
	setup-2.0.5-1::/etc/group 
	setup-2.0.5-1::/etc/host.conf 
	setup-2.0.5-1::/etc/passwd 
	sh-utils-2.0-1::/bin/basename 
	sh-utils-2.0-1::/bin/false 
	sh-utils-2.0-1::/bin/sleep 
	sh-utils-2.0-1::/bin/true 
	sh-utils-2.0-1::/usr/bin/expr 
	sh-utils-2.0-1::/usr/bin/id 
	texinfo-3.12h-2::/usr/bin/makeinfo 
	textutils-2.0-2::/bin/cat 
	textutils-2.0-2::/bin/sort 
	textutils-2.0-2::/usr/bin/cut 
	textutils-2.0-2::/usr/bin/tr 
	zlib-1.1.3-5::/usr/lib/libz.so.1 
```

The information can be used to generate build prerequisites. What is still
needed is a sensible approach on

```
	1) eliminating obvious common dependencies (e.g. libtool, egcs).
	2) identifying (and removing for now) per-platform build dependencies.
	3) deciding on whether to add the build dependency on a file or on the
	package that contains the file.
	4) if adding a dependency on a package, choosing version ranges as
	appropriate.
```

but that's up to individual packagers.
