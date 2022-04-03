---
section: 32
x-masysma-name: masysmaci/build
title: Ma_Sys.ma Build -- Ant Build Template
date: 2019/12/06 13:16:32
lang: en-US
author: ["Linux-Fan, Ma_Sys.ma (Ma_Sys.ma@web.de)"]
keywords: ["ant", "mdpc2", "make", "build", "masysmaci", "template", "ant_build_template", "ant-build-template.xml", "ci"]
x-masysma-version: 1.0.0
x-masysma-repository: https://www.github.com/m7a/co-build
x-masysma-website: https://masysma.lima-city.de/32/masysmaci_build.xhtml
x-masysma-owned: 1
x-masysma-copyright: |
  Copyright (c) 2019, 2020 Ma_Sys.ma.
  For further info send an e-mail to Ma_Sys.ma@web.de.
---
Overview
========

The _Ma_Sys.ma Build_ is an attempt to provide a single unifying `build.xml` for
the Ant build system which allows compiling most Ma_Sys.ma programs with only
minor modifications and which provides extended functionality to generate Debian
packages. This replaces large parts of the formerly used MDPC scripts and the
previously used Makefile templates.

In addition to being a template, this is designed to facilitate _code reuse_ by
not being copied upon use, but rather _imported_. E.g. this XML can be loaded
directly from its repository or alternatively from the file system. See section
_Using the Template_ for details.

The template is designed to allow building programs in the following programming
languages:

 * Ada
 * C
 * Java

Additionally, it can compile _LaTeX_ source code to PDF and act as a top-level
template for recursive invocation.

How to compile Projects using this Template
===========================================

Similar to Makefiles, `ant` provides a set of targets.
The following targets/functionalities are generally available.

`build`
:   This target compiles the project.
`jar`
:   If it is a Java project, `jar` builds a jarfile if possible.
`clean`
:   Deletes object files i.e. everything generated except for the linked build
    result.
`dist-clean`
:   Deletes all generated files including the linked build result.

If a project is intended to be built in form of a Debian package, this
additional target can be invoked:

`package`
:   Build a Debian package for this project.

For development purposes, the following targets allow managing the state of
a package:

`init`
:   Initializes a changelog.
`incver`
:   Interactively invoke the VIM editor to add a changelog entry
    (for an incremented version).
`trigger`
:   Ma_Sys.ma CI Integration: Trigger invoking CI targets by connecting to a
    locally running Ma_Sys.ma CI instance (`http://127.0.0.1:9030`).

Using the Template
==================

Two ways of including this template are recommended.

The include is a little obscure because it implements a routine for searching
the template's XML file and retrieving it from a local filesystem if possible.
The logic tries two distinct approaches:

 1. _Attempt to load from a local file_. This searches for a copy of the
    repository by the name `co-build` in the directory given by
    environment variable `MDVL_CI_PHOENIX_ROOT` or `..` (i.e. one level up the
    present working directory) if `MDVL_CI_PHOENIX_ROOT` is not set.
 2. If the file was not found this way, _attempt to load from Internet_. To
    do this, it tries to download the XML from GitHub, i.e. URL
    <https://raw.githubusercontent.com/m7a/co-build/master/ant-build-template.xml>.

## Variant A: Block at the end of a file

This is the preferred default variant which consists of a block entitled
`CONSTANT TPL IMPORT` to indicate that what follows is not intended to
normally be modified.

~~~{.xml}
<?xml version="1.0" encoding="UTF-8"?>
<project default="build">

<!-- APPLICATION METADATA -->
<property name="masysma.target"  value="hello"/> 
<!-- ... -->

<!-- CI INTEGRATION -->
<!-- ... -->

<!-- CONSTANT TPL IMPORT -->
<property environment="env"/>
<condition property="masysma.internal.includepath.rel" value="${env.MDVL_CI_PHOENIX_ROOT}" else=".."><isset property="env.MDVL_CI_PHOENIX_ROOT"/></condition>
<property name="masysma.internal.includepath" location="${masysma.internal.includepath.rel}"/>
<property name="masysma.internal.loadpath" value="${masysma.internal.includepath}/co-build/ant-build-template.xml"/>
<condition property="masysma.internal.load" value="file://${masysma.internal.loadpath}" else="https://raw.githubusercontent.com/m7a/co-build/master/ant-build-template.xml"><resourceexists><file file="${masysma.internal.loadpath}"/></resourceexists></condition>
<import><url url="${masysma.internal.load}"/></import>

</project>
~~~

## Variant B: Block split into two parts

This variant is useful if access to `MDVL_CI_PHOENIX_ROOT` is also needed by
parts of the build targets. They can read its absolute location from property
`masysma.internal.includepath`.

~~~{.xml}
<?xml version="1.0" encoding="UTF-8"?>
<project default="build">

<!-- (metadata etc) -->

<!-- PART OF TPL HERE FOR USE IN BUILD -->
<property environment="env"/>
<condition property="masysma.internal.includepath.rel" value="${env.MDVL_CI_PHOENIX_ROOT}" else=".."><isset property="env.MDVL_CI_PHOENIX_ROOT"/></condition>
<property name="masysma.internal.includepath" location="${masysma.internal.includepath.rel}"/>

<!-- (customized build instructions -->

<!-- TPL IMPORT (PARTIALLY GIVEN ABOVE) -->
<property name="masysma.internal.loadpath" value="${masysma.internal.includepath}/co-build/ant-build-template.xml"/>
<condition property="masysma.internal.load" value="file://${masysma.internal.loadpath}" else="https://raw.githubusercontent.com/m7a/co-build/master/ant-build-template.xml"><resourceexists><file file="${masysma.internal.loadpath}"/></resourceexists></condition>
<import><url url="${masysma.internal.load}"/></import>

</project>
~~~

Use in Projects with a single Programming Language
==================================================

The following sections show how the template can be instantiated for a given
directory with the source code in a single programming language.

To allow automatic compilation, at least a `masysma.target` needs to be given.
This is usually set to the name of the output file without extension.

## Ada

Detection
:   A project compiled as Ada source code, if a file with name
    `${masysma.target}.adb` can be found.

Compilation
:   Ada compilation is performed by invoking the `gnatmake` utility.

Example
:   See [ma_capsblinker(11)](../11/ma_capsblinker.xhtml).

No language-specific properties defined. `masysma.target` refers to the output
executable file to produce.

## C

Detection
:   C compilation is detected by the existence of any `.c` file in the
    `${basedir}`.

Compilation
:   Compilation for C projects consists of two separate invocations:

 1. Compilation of the individual `.c` files with
    `gcc -Wall -pedantic -std=c89 -O3 FILE.c`
 2. Linking of all `.o` files with
    `gcc -o ${masysma.target} *.o`

Example
:   See [progress(32)](progress.xhtml), especially the _Progress 2 C_ variant.

Properties to influence compilation are described in the following:

`masysma.c.standard`
:   Sets a different value for the `-std=` flag. Default: `c89`.
`masysma.c.compile.1` and `masysma.c.compile.2`
:   Sets additional arguments to pass to the compiler invocation.
    The use of this properties is similar to `masysma.c.link.X` explained
    below.
`masysma.c.link.1` and `masysma.c.link.2`
:   Sets additional arguments to pass to the final/linking `gcc` invocation.
    These are added as-is to the commandline if they have a value different
    from the empty string. Example: `-lpthread`. Currently, the number of
    such arguments is limted to 2, but of course, it is possible to increase
    that if more are needed in the future.

## Java

Detection
:   Java compilation is assumed if any `.java` files are found in the file
    tree (below the `build.xml`'s directory).

Compilation
:   All files are compiled including debug information and with
    `-Xlint:unchecked`. To invoke `javac`, ant's Javac task is instantiated.

To generate `.jar` files, target `jar` is provided. By default, all files from
directory `ma` are included in the jarfile. The name of the jarfile is derived
from `masysma.target` by appending suffix `.jar`.

Example
:   See [gamuhr(32)](gamuhr.xhtml) for basic compilation.
    See [progress(32)](progress.xhtml) (subdirectory `progress1`) for an example
    with classes provided in a directory other than `ma`.

Relevant properties for Java compilation are as follows:

`masysma.main`
:   Defines the main-class (only relevant in conjunction with target `jar`).

Aside from properties, there are further settings that can be tuned for
compilation.

`masysma.inccls` (fileset)
:   This fileset defines the list of files to include in the jarfile. By
    default, all files from below directory `ma` are included which means
    sources (`.java`), classes (`.class`) along with any other files the
    application might consider resources (e.g. `.txt`, `.png`, etc.).
    Change this setting in case classes are not taken from directory `ma`.
    Example: `<fileset id="masysma.inccls" dir="." includes="*.class"/>`
    includes all classfiles from the present working directory (useful for
    compiling programs which are using the default package).
    In order for this setting to take effect, it has to be specified after
    importing the template i.e. near the end of the `build.xml` file.

`masysma.classpath` (path)
:   This path defines the locations to look for classes (only relevant for
    compilation). By default, the `build.xml`'s directory and all `.jar` files
    from below a directory called `lib` are included.

## LaTeX

As LaTeX compilation may require multiple stages, the compilation is invoked
repeatedly until `Rerun to` no longer appears in the output. Additionally,
`bibtex8` is automatically invoked if file `literature.bib` exists. The `.tex`
file to compile is expected to have its name equal to the value of
property `masysma.target` followed by `.tex`. Task `clean` attempts to delete
most files created along with the `.aux` file for the respective source file.
Note that builds of multiple `.tex` files are only supported as far as they are
included with `\input`, because auxiliary files created by inclusion of other
`.tex` files are not cleaned up by default.

Detection
:   LaTeX compilation is invoked if a file called `${masysma.target}.tex`
    exists.

Compilation
:   Compilation currently uses a fixed set of flags and always invokes
    `pdflatex -halt-on-error -file-line-error -interaction batchmode`.
    If `literature.tex` exists, BibTeX is invoked with
    `bibtex8 ${masysma.target}`.

Use in Projects with multiple Parts
===================================

To build a project with multiple programming languages or parts, create
distinct subdirectories for each of the individual build configurations.
Inside them, provide individual `build.xml` files. These may use the template
as explained before except that the relative location of file
`ant-build-template.xml` may be one directory farther upwards. To apply this
change, edit the following line from the template:

~~~{.xml}
<condition property="masysma.internal.includepath.rel" value="${env.MDVL_CI_PHOENIX_ROOT}" else=".."><isset property="env.MDVL_CI_PHOENIX_ROOT"/></condition>
~~~

and change it to this:

~~~{.xml}
<condition property="masysma.internal.includepath.rel" value="${env.MDVL_CI_PHOENIX_ROOT}" else="../.."><isset property="env.MDVL_CI_PHOENIX_ROOT"/></condition>
~~~

The resulting structure behaves like a recursive makefile except it is using
`ant`. Note that the following targets can be invoked on the top-level
to apply them to all parts: `build`, `clean` and `dist-clean`.

As an example, consider Ma_Sys.ma DirStat [dirstat(32)](dirstat.xhtml), whose
`build.xml` files are organized as follows to build two distinct subprojects:
DirStat 1 and DirStat 2.

	/bo-dirstat
	 |
	 +-- dirstat1/
	 |    |
	 |    +-- build.xml
	 |
	 +-- dirstat2/
	 |    |
	 |    +-- build.xml
	 |
	 +-- build.xml

The top-level `build.xml` uses the template to declare an MDVL pacakge but does
not itself provide any `masysma.target` definition. The subordinate `build.xml`
files from the subdirectories are as follows (relevant excerpts shown):

For DirStat 1:

~~~{.xml}
<property name="masysma.target" value="dirstat1"/> 
<property name="masysma.main" value="ma.stat.Main"/> 
~~~

For DirStat 2:

~~~{.xml}
<property name="masysma.target" value="dirstat2"/> 
<property name="masysma.main" value="ma.dirstat.Main"/> 
~~~

Here, both parts are Java projects, but this need not be the case.
Also, it is not _required_ that the individual parts use the template -- they
may also consist of standalone Ant `build.xml` files.

MDPC 2.0
========

MDPC expands to _Ma_Sys.ma Developer Linux Packaging Control_ and provides a
means to build Debian-compatible packages in a simplified fashion.

The first version was published on the debian-user mailinglist
<https://lists.debian.org/debian-user/2013/08/msg00042.html> and consists of
a series of shell scripts. The new version is (partially) integrated with
Ma_Sys.ma Build and relies on Ant instead of shell scripts.

MDPC 2 sets defaults for files which would have otherwise to be provided
explicitly and declares Ant properties for relevant fields to fill in for a
given package. It provides the following targets: `package`, `init`, `incver`.
The properties to declare package metadata are explained in the following.

## Common Metadata

This section lists the most commonly needed MDPC 2 properties.

`mdpc.name` (required)
:   Declares the name of the package to build. This will later appear in
    the installation logs, file name and package search results if queried.

`mdpc.arch` (default: `all`)
:   Declares the package architecture.
    `all` means that the package is suited for all architectures (e.g. for
    portable scripts, this can be the case). `any` declares that the package
    can be compiled for any architecture, but the resulting binaries are
    architecture-specific. Apart from these two (very common) options, it is
    possible to specify an architecture identifier like `amd64`, `armhf`,
    `i386` etc. directly although this should only be necessary for packages,
    which rely on pre-built binaries.

`mdpc.descrs` (required)
:   One-line description of the package.

`mdpc.descrl` (required)
:   Multi-line description of the package. Note that all lines need to be
    indented by a single whitespace and “empty” lines are expected to be marked
    by a single whitespace followed by a dot (` .`).

`mdpc.depends` (default: `${shlibs:Depends}, ${misc:Depends}, ${java:Depends}`)
:   Declares the package's dependencies in Debian `control` file format
    (comma-separated). By default, a series of `${...}` attempt to auto-detect
    some dependencies, but this is known to be incomplete for scripts.

`mdpc.section` (required)
:   Associates the package with a _section_, which can be one of the following:
    `admin`, `devel`, `doc`, `editors`, `electronics`, `embedded`, `games`,
    `graphics`, `httpd`, `interpreters`, `java`, `libs`, `math`, `metapackages`,
    `misc`, `net`, `science`, `shells`, `tex`, `text`, `utils`, `web`, `x11`

`mdpc.maintainer` (default: `Linux-Fan, Ma_Sys.ma <Ma_Sys.ma@web.de>`)
:   Declares the package's maintainer.

## License Metadata

This section explains the licensing-related MDPC 2 properties. It is an intended
design goal that precise license specifications should be possible with MDPC 2,
thus there are multiple properties to influence the `copyright` file at
different levels of granularity.

`mdpc.cpyear` (default: `2022`)
:   Declares the copyright year range for the current package.
    For instance, if you develeoped a project from 2018 to 2020, you might
    set this to `2018, 2019, 2020` etc.
`mdpc.copyright`
:   Declares a copyright notice in single-line format.\
    Default value: `Copyright (c) ${mdpc.cpyear} Ma_Sys.ma <Ma_Sys.ma@web.de>`
`mdpc.lfiles` (Default: `*`)
:   Declares the set of files to apply the license inforamtion to.
`mdpc.lblock` (Default: GPL3+)
:   Declares a constant license block to use.\
    Defaults to the license block for _GPL 3 or higher_ which is used with
    most published Ma_Sys.ma projects.
`mdpc.copyright.file`
:   If this property is set, properties `mdpc.cpyear`, `mdpc.copyright` and
    `mdpc.lblock` are ignored unless explicitly referenced. This provides the
    raw contents of a Debian `copyright` file allowing for external projects and
    their licenses to be added or for a different choice of license from GPL3+:
    If this property is not set, a `copyright` file will be generated from
    `mdpc.cpyear` and `mdpc.copyright` properties and it will set the license to
    the value defined in `mdpc.lblock`

Ma_Sys.ma Note
:   Ma_Sys.ma code is usually released under _GPL 3 or higher_ and thus if you
    do not find any copyright information in a Ma_Sys.ma project, try to
    generate its package and refer to the generated `copyright` information. If
    you need the license declearation to be more explicit, just send an e-mail
    to `Ma_Sys.ma@web.de` and suitable annotations may be added to the project
    under consideration.

## Optional and Special Metadata

Here are some special-purpose and rarely needed properties.

`mdpc.bdep` (default: `debhelper (>= 8)`)
:   Declares the package's build-time dependencies.

`mdpc.priority` (default: `optional`)
:   Declares the package's priority. Most often, this need not be changed.

`mdpc.recommends` (optional)
:   Declares recommended dependencies. These need not be installed, but will
    be installed by default. For the exact meanings, refer to Debian Policy.

`mdpc.conflicts` (optional)
:   Declares `Conflicts`.

`mdpc.suggests` (optional)
:   Declares `Suggests`.

## File Properties

Some properties map directly to related Debian files:

 * `mdpc.file.install` -- contents for file `debian/install`.\
   Default value: `${masysma.target} /usr/bin`
 * `mdpc.file.displace` -- contents for file `debian/displace` (optional).
 * `mdpc.add.rules` -- contents to add to `debian/rules` (optional).

## Example

As an example, consider creating a simple _Hello World_ package.
Initially, it has just these two files:

	hello/
	 |
	 +-- hello.c
	 |
	 +-- build.xml

`hello.c` is a minimal C program:

~~~{.c}
#include <stdio.h>
int main(int argc, char** argv)
{
	printf("Hello world.\n");
	return 0;
}
~~~

`build.xml` is as follows:

~~~{.xml}
<?xml version="1.0" encoding="UTF-8"?>
<project default="build">

<!-- APPLICATION METADATA -->
<property name="masysma.target" value="hello"/> 
<property name="mdpc.name"      value="mdvl-hello"/>
<property name="mdpc.section"   value="misc"/>
<property name="mdpc.descrs"    value="Hello Package"/>
<property name="mdpc.arch"      value="any"/>
<property name="mdpc.descrl">
 This is the hello package.
</property>

<!-- CONSTANT TPL IMPORT -->
<property environment="env"/>
<condition property="masysma.internal.includepath.rel" value="${env.MDVL_CI_PHOENIX_ROOT}" else=".."><isset property="env.MDVL_CI_PHOENIX_ROOT"/></condition>
<property name="masysma.internal.includepath" location="${masysma.internal.includepath.rel}"/>
<property name="masysma.internal.loadpath" value="${masysma.internal.includepath}/co-build/ant-build-template.xml"/>
<condition property="masysma.internal.load" value="file://${masysma.internal.loadpath}" else="https://raw.githubusercontent.com/m7a/co-build/master/ant-build-template.xml"><resourceexists><file file="${masysma.internal.loadpath}"/></resourceexists></condition>
<import><url url="${masysma.internal.load}"/></import>

</project>
~~~

To perform the first build of a package it is necessary to provide a
`debian-changelog.txt`. To create this file, target `init` can be used as
follows: `ant init`. Afterwards, there are three files:

	hello/
	 |
	 +-- hello.c
	 |
	 +-- debian-changelog.txt
	 |
	 +-- build.xml

The new file's contents are as follows:

~~~
mdvl-hello (1.0.0) stable; urgency=medium

  * Package created with MDPC 2

 -- Linux-Fan, Ma_Sys.ma <Ma_Sys.ma@web.de>  Sat, 28 Mar 2020 16:51:28 +0100
~~~

See [documentation on `debian/changelog`](https://www.debian.org/doc/manuals/maint-guide/dreq.en.html#changelog)
for hints on the format of the file. Having established this directory
structure, one can call `ant package` to create the associated package file.
If the package is intended to be upgraded to a new version, target `incver`
can be invoked before performing the package build.

Ma_Sys.ma-supplied packages already contain a `debian-changelog.txt` which means
it is sufficient to invoke `ant package` only to generate their package files.

Downloading Artifacts
=====================

In case external dependencies are needed for a build, Ma_Sys.ma Build allows
downloading Git repositories and artifacts.

## Downloading Ma_Sys.ma Git Repositories

A downloaded Git repository is stored one next to the repository's directory.
The mechanism is invoked as follows (example):

~~~{.xml}
<masysma_require_repo masysmarepoid="co-maartifact"/>
~~~

The repository name is given in attribute `masysmarepoid`.

Note that this is specific to Ma_Sys.ma Repositories, i.e. prefix
`https://github.com/m7a`. To use this mechanism for your own repositories, you
could create a modified copy of the build template and change the attribute name
and base URL in `ant-build-template.xml`. Alternatively, consider using the more
versatile `masysma_require_artifact` element described in the next section.

An advanced example for using the mechanism to download a required library
can be found in `ma_inventory_barcodegensvg/build.xml` from the
[ma_inventory(32)](ma_inventory.xhtml) repository.

## Downloading Artifacts -- maartifact Integration

[maartifact(11)](../11/maartifact.xhtml) can be invoked from Ma_Sys.ma Build
targets by using the specifically provided `masysma_require_artifact` element.
If not already available, `maartifact` is downloaded using the Git download
mechanism explained before.

As an example, consider the following excerpt from `build.xml` of repository
`bp-jexer`:

~~~{.xml}
<target name="download">
	<masysma_require_artifact
			masysmaartifact="jexer.git"
			masysmaatarget="build"
			masysmaadef="https://gitlab.com/klamonte/jexer.git/"/>
</target>
~~~

Here, the repository specified under `masysmaadef` is downlaoded to directory
`build`. The name given under `masysmaartifact` serves to identify the
downloaded resource among all downloaded artifacts such that the download only
needs to happen on first use.

This XML specification of an artifact corresponds to the following `maartifact`
commandline invocation (`@...` refers to the value of attribute `...`):

	maartifact extract @masysmaartifact @masysmaartarget @masysmaadef

As an alternative to `extract`, a custom action can be specified by attribute
`masysmaaction`.

See [maartifact(11)](../11/maartifact.xhtml) for further documentation on the
use of `maartifact`.

Ma_Sys.ma CI Integration
========================

Ma_Sys.ma Build integrates with _Ma_Sys.ma CI_ in multiple ways. On the most
basic level, data from `build.xml` and `debian-changelog.txt` files is used to
register a repository for processing by the Ma_Sys.ma CI. Additionally, it is
possible to use target `trigger` to trigger a CI build in a Ma_Sys.ma CI
instance running on the local host port 9030.

Two modes of invocation are possible:

 1. `ant trigger` -- triggers all associated CI jobs.
 2. `ant trigger -Dmdpc.ttarget=...` --
    triggers a specific CI target as given after the `=` sign.

License
=======

	Ma_Sys.ma Build Ant Build Template,
	Copyright (c) 2019, 2020, 2022 Ma_Sys.ma.
	For further info send an e-mail to Ma_Sys.ma@web.de.
	
	This program is free software: you can redistribute it and/or modify
	it under the terms of the GNU General Public License as published by
	the Free Software Foundation, either version 3 of the License, or
	(at your option) any later version.
	
	This program is distributed in the hope that it will be useful,
	but WITHOUT ANY WARRANTY; without even the implied warranty of
	MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
	GNU General Public License for more details.
	
	You should have received a copy of the GNU General Public License
	along with this program.  If not, see <http://www.gnu.org/licenses/>.
