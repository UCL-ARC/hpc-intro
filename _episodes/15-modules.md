---
title: "Accessing software via Modules"
teaching: 30
exercises: 15
questions:
- "How do we load and unload software packages?"
objectives:
- "Load and use a software package."
- "Explain how the shell environment changes when the module mechanism loads or unloads packages."
keypoints:
- "Load software with `module load softwareName`."
- "Unload software with `module unload`"
- "The module system handles software versioning and package conflicts for you
  automatically."
---

On a high-performance computing system, it is seldom the case that the software
we want to use is available when we log in. It is installed, but we will need
to "load" it before it can run.

Before we start using individual software packages, however, we should
understand the reasoning behind this approach. The three biggest factors are:

- software incompatibilities
- versioning
- dependencies

Software incompatibility is a major headache for programmers. Sometimes the
presence (or absence) of a software package will break others that depend on
it. Two well known examples are Python and C compiler versions.
Python 3 famously provides a `python` command that conflicts with that provided
by Python 2. Software compiled against a newer version of the C libraries and
then run on a machine that has older C libraries installed will result in a
nasty `'GLIBCXX_3.4.20' not found` error.

Software versioning is another common issue. A team might depend on a certain
package version for their research project - if the software version was to
change (for instance, if a package was updated), it might affect their results.
Having access to multiple software versions allows a set of researchers to
prevent software versioning issues from affecting their results.

Dependencies are where a particular software package (or even a particular
version) depends on having access to another software package (or even a
particular version of another software package). For example, the VASP
materials science software may depend on having a particular version of the
FFTW (Fastest Fourier Transform in the West) software library available for it
to work.

## Environment Modules

Environment modules are the solution to these problems. A _module_ is a
self-contained description of a software package -- it contains the
settings required to run a software package and, usually, encodes required
dependencies on other software packages.

There are a number of different environment module implementations commonly
used on HPC systems: the two most common are _TCL modules_ and _Lmod_. Both of
these use similar syntax and the concepts are the same so learning to use one
will allow you to use whichever is installed on the system you are using. In
both implementations the `module` command is used to interact with environment
modules. An additional subcommand is usually added to the command to specify
what you want to do. For a list of subcommands you can use `module -h` or
`module help`. As for all commands, you can access the full help on the _man_
pages with `man module`.

On login you may start out with a default set of modules loaded or you may
start out with an empty environment; this depends on the setup of the system
you are using.

### Listing Available Modules

To see available software modules, use `module avail`:

```
{{ site.remote.prompt }} module avail
```
{: .language-bash}

{% include {{ site.snippets }}/modules/available-modules.snip %}

### Listing Currently Loaded Modules

You can use the `module list` command to see which modules you currently have
loaded in your environment. If you have no modules loaded, you will see a
message telling you so

```
{{ site.remote.prompt }} module list
```
{: .language-bash}

{% include {{ site.snippets }}/modules/default-modules.snip %}

## Loading and Unloading Software

To load a software module, use `module load`. In this example we will use
Python 3.

Initially, Python 3 is not loaded. We can test this by using the `which`
command. `which` looks for programs the same way that Bash does, so we can use
it to tell us where a particular piece of software is stored.

```
{{ site.remote.prompt }} which python3
```
{: .language-bash}

{% include {{ site.snippets }}/modules/missing-python.snip %}

We can load the `python3` command with `module load`:

{% include {{ site.snippets }}/modules/module-load-python.snip %}

{% include {{ site.snippets }}/modules/python-executable-dir.snip %}

So, what just happened?

To understand the output, first we need to understand the nature of the `$PATH`
environment variable. `$PATH` is a special environment variable that controls
where a UNIX system looks for software. Specifically `$PATH` is a list of
directories (separated by `:`) that the OS searches through for a command
before giving up and telling us it can't find it. As with all environment
variables we can print it out using `echo`.

```
{{ site.remote.prompt }} echo $PATH
```
{: .language-bash}

{% include {{ site.snippets }}/modules/python-module-path.snip %}

You'll notice a similarity to the output of the `which` command. In this case,
there's only one difference: the different directory at the beginning. When we
ran the `module load` command, it added a directory to the beginning of our
`$PATH`. Let's examine what's there:

{% include {{ site.snippets }}/modules/python-ls-dir-command.snip %}

{% include {{ site.snippets }}/modules/python-ls-dir-output.snip %}

Taking this to its conclusion, `module load` will add software to your `$PATH`.
It "loads" software. A special note on this - depending on which version of the
`module` program that is installed at your site, `module load` will also load
required software dependencies.

{% include {{ site.snippets }}/modules/software-dependencies.snip %}

Note that this module loading process happens principally through
the manipulation of environment variables like `$PATH`. There
is usually little or no data transfer involved.

The module loading process manipulates other special environment
variables as well, including variables that influence where the
system looks for software libraries, and sometimes variables which
tell commercial software packages where to find license servers.

The module command also restores these shell environment variables
to their previous state when a module is unloaded.

## Software Versioning

So far, we've learned how to load and unload software packages. This is very
useful. However, we have not yet addressed the issue of software versioning. At
some point or other, you will run into issues where only one particular version
of some software will be suitable. Perhaps a key bugfix only happened in a
certain version, or version X broke compatibility with a file format you use.
In either of these example cases, it helps to be very specific about what
software is loaded.

Let's examine the output of `module avail` more closely.

```
{{ site.remote.prompt }} module avail
```
{: .language-bash}

{% include {{ site.snippets }}/modules/available-modules.snip %}

{% include {{ site.snippets }}/modules/wrong-gcc-version.snip %}

> ## Using Software Modules in Scripts
>
> Create a job that is able to run `python3 --version`. Remember, no software
> is loaded by default! Running a job is just like logging on to the system
> (you should not assume a module loaded on the login node is loaded on a
> compute node).
>
> > ## Solution
> >
> > ```
> > {{ site.remote.prompt }} nano python-module.sh
> > {{ site.remote.prompt }} cat python-module.sh
> > ```
> > {: .language-bash}
> >
> > ```
> > {{ site.remote.bash_shebang }}
> > {% if site.sched.flag.qos %}
> > {{ site.sched.comment }} {{ site.sched.flag.qos }}
> > {% endif %}{{ site.sched.comment }} {{ site.sched.flag.time }}00:00:30
> > 
> > module load {{ site.remote.module_python3 }}
> >
> > python3 --version
> > ```
> > {: .output}
> >
> > ```
> > {{ site.remote.prompt }} {{ site.sched.submit.name }} {% if site.sched.submit.options != '' %}{{ site.sched.submit.options }} {% endif %}python-module.sh
> > ```
> > {: .language-bash}
> {: .solution}
{: .challenge}

{% include links.md %}
