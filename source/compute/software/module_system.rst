.. _module_system:

#############
Module system
#############

Many software packages are installed as modules. These packages range from
compilers, interpreters and mathematical libraries to the actual scientific
software applications.

All VSC sites use a Lua based implementation called `Lmod`_. Interacting
with the module system happens via the ``module`` command  or its shorter
equivalent ``ml``. To get a list of all available module commands, type:

::

   $ module help


Available modules
=================

To view the list of all available software packages, use the command
``module av``. The output will look similar to this:

.. code-block:: shell

   $ module av

   ---------------- /apps/leuven/rocky8/icelake/2023a/modules/all -----------------
   ATK/2.38.0-GCCcore-12.3.0                               (D)
   Armadillo/12.6.2-foss-2023a                             (D)
   Bison/3.8.2                                             (D)
   ...
   CP2K/2023.1-foss-2023a                                  (D)
   CP2K/2023.1-intel-2023a
   ...
   zlib/1.2.13                                             (D)

   ---------------- /apps/leuven/rocky8/icelake/2022b/modules/all -----------------
   ATK/2.38.0-GCCcore-12.2.0                               (D)
   ...

As such lists tend to be rather long, it is common to use more specific queries
(see :ref:`Searching for modules <module_search>` below).


Module names
============

In general, the anatomy of a module name is ``<package>/<version>-<toolchain>[-<extra>]``.

For example, in the module name ``GROMACS/2023.3-foss-2023a-PLUMED-2.9.0`` we have:

* package: ``GROMACS``, the name of the software package
* version: ``2023.3``, the GROMACS Version
* toolchain: ``foss-2023a``, the toolchain GROMACS was built with
* extra: ``PLUMED-2.9.0``, the version of PLUMED used by this GROMACS

.. note::

  The ``extra`` part of the module name is usually only used to distinguish
  variants of the same modules. In the example with GROMACS, you might find
  another GROMACS v2023.3 without PLUMED or using a different version of PLUMED.

Toolchains such as ``intel-2023a`` or ``foss-2023a`` refer to the 2023a
versions of the :ref:`toolchains <toolchains>` based on the Intel and GNU
compilers respectively. Modules that do not belong to a particular toolchain
are called *system* modules. The module name of *system* modules is simpler
with a structure ``<package>/<version>[-<extra>]``.

When different modules exist for the same package, for example because it has
been compiled with two different toolchains, please consider trying out the
different modules so as to choose the one that performs best for your workloads.

.. _module_search:

Searching for modules
=====================

Often, when looking for some specific software, you will want to filter
the list of available modules, since it tends to be rather large.
For a (case-insensitive) search for modules containing the word ``python``,
you can either try

.. code-block:: shell

   $ module av python

or, for a more comprehensive search

.. code-block:: shell

   $ module spider python


To restrict the search to modules where the package name ends with ``python``,
add a trailing slash (e.g. ``module av python/``).

.. note::

   The module command writes its output to standard error, rather than standard
   output. If you want to use pipes for filtering, consider using ``2>&1``
   or ``|&`` (e.g. ``module av |& grep -i python``).


Information about modules
=========================

The ``spider`` sub-command can also be used to provide information on a specific
module, e.g.

.. code-block:: shell

   $ module spider Python/3.11.3-GCCcore-12.3.0

   ----------------------------------------------------------------------------
   Python: Python/3.11.3-GCCcore-12.3.0
   ----------------------------------------------------------------------------
    Description:
      Python is a programming language that lets you work more quickly and
      integrate your systems more effectively.

      ...

      More information
      ================
       - Homepage: https://python.org/

      Included extensions
      ===================
      flit_core-3.9.0, packaging-23.1, pip-23.1.2, setuptools-67.7.2,
      setuptools_scm-7.1.0, tomli-2.0.1, typing_extensions-4.6.3, wheel-0.40.0


More technical information can be obtained using the ``show`` sub-command.
It will show which other modules will get loaded and in which ways various
environment variables (``PATH``, ``LD_LIBRARY_PATH``, ...) will be modified,
e.g.:

.. code-block:: shell

   $ module show Python/3.11.3-GCCcore-12.3.0


Loading modules
===============

A module is loaded using the ``module load`` command, e.g.:

.. code-block:: shell

   $ module load CP2K

will load the default ``CP2K`` module (``CP2K/2023.1-foss-2023a`` in this
example).

If multiple versions are installed; the ``module load`` command will
automatically choose the default version, which is typically, but not always,
the most recent version. If, in this example, you would prefer to use same
version of CP2K but built with the ``intel-2023a`` toolchain, you would need
to specify:

.. code-block:: shell

   $ module load CP2K/2023.1-intel-2023a

.. note::

   Loading modules with explicit versions is considered as best practice. It ensures
   that your scripts will use the expected version of the software, regardless of
   newly installed software. Failing to do this may jeopardize the reproducibility
   of your results!

Modules need not be loaded one by one; two ``load`` sub-commands
can for example be combined as follows:

.. code-block:: shell

   $ module load CP2K/2023.1-foss-2023a GROMACS/2023.3-foss-2023a-PLUMED-2.9.0

.. warning::

   Do *not* load modules in your ``.bashrc``, ``.bash_profile`` or ``.profile``,
   you *will* shoot yourself in the foot at some point. If you want to avoid
   typing the same module load commands over and over, we instead recommend to
   define aliases or functions in your ``.bashrc``.


Conflicting modules
-------------------

It is important to note that only modules that are compatible with
each other should be loaded together. The loaded modules should all
be associated with either the same toolchain or compatible (sub)toolchains
(see also https://docs.easybuild.io/common-toolchains/#toolchains_diagram)
or be ``system`` modules.

For example, once you have loaded a module that uses the ``foss/2023a``
toolchain, all other modules that you load next should have been installed
with the same toolchain or with compatible (sub)toolchains such as
``GCCcore/12.3.0``, ``GCC/12.3.0`` and ``gompi/2023a``. Subtoolchains
compatible with e.g. ``intel/2023a`` include ``GCCcore/12.3.0``,
``intel-compilers/2023.1.0`` and ``iimpi/2023a``.

Additionally, two versions of the same software packages can not be loaded
together. If you e.g. loaded a ``Python/3.11.3-GCCcore-12.3.0`` module, then
also loading another ``Python`` module (either directly or as a dependency of
another module) will cause ``Python/3.11.3-GCCcore-12.3.0`` to be unloaded and
replaced by the new module (the same will happen to the modules which both
``Python`` modules load as dependencies).


List loaded modules
-------------------

Obviously, the user needs to keep track of the modules that are
currently loaded. After executing the above load command, the list
of loaded modules will look similar to:

.. code-block:: shell

   $ module list
   Currently Loaded Modulefiles:
     1) cluster/wice/batch
     2) GCCcore/10.3.0
     ...
     16) OpenMPI/4.1.1-GCC-10.3.0
     17) OpenBLAS/0.3.15-GCC-10.3.0
     ...
     46) PLUMED/2.9.0-foss-2023a
     47) CP2K/2023.1-foss-2023a
     48) GROMACS/2023.3-foss-2023a-PLUMED-2.9.0

Note that this does not just show the two requested modules, but also all
the modules that got loaded automatically in order to satisfy (runtime)
dependencies of the explicitly loaded ``CP2K`` and ``GROMACS`` installations
(``PLUMED``, ``OpenMPI``, ``OpenBLAS``, etcetera).


Unloading modules
=================

To unload a specific module, use the ``module unload`` command, e.g.:

::

   $ module unload CP2K

Notice that the version was not specified: the module system is
sufficiently clever to figure out what the user intends. However,
checking the list of currently loaded modules is always a good idea,
just to make sure.

Keep in mind that ``module unload`` only unloads the chosen module.
It will not unload other modules which were automatically loaded
as dependencies of the chosen module.


.. _module_purge:

Purging modules
---------------

In order to unload all modules at once and start with a clean slate, use:

.. code-block:: shell

   $ module purge

This will not unload so-called `sticky modules
<https://lmod.readthedocs.io/en/latest/240_sticky_modules.html>`__, which
are special modules that do not normally need to be unloaded (for example
because they define the appropriate module paths and possibly other environment
variables). If really needed, sticky modules can be unloaded with
``module --force purge``.


.. _collections of modules:

Collections of modules
======================

It is also possible to bundle different modules together as a collection:

   #. Be sure to start with a clean environment:

      .. code-block:: shell

         $ module purge

   #. Load the modules you want in your collection, e.g.,

      .. code-block:: shell

         $ module load matplotlib/3.7.2-gfbf-2023a
         $ module load MATLAB/2023b

   #. Save your collection, e.g., as ``data_analysis``

      .. code-block:: shell

         $ module save data_analysis

   #. At a later point, you can load the module collection via:

      .. code-block:: shell

         $ module restore data_analysis

   #. To list all your collections:

      .. code-block:: shell

         $ module savelist

   #. To remove the collection:

      .. code-block:: shell

         $ rm ~/.lmod.d/data_analysis


.. warning::

   Be aware that module collections stop working when one of the
   modules in the collection is reinstalled. In such cases the
   collection needs to be removed and then redefined.


.. _module_hierarchy:

Module organization
===================

When a cluster is not completely homogeneous (for instance there are
differences regarding architecture or operating system between nodes), it is
important to use the appropriate modules in your jobs. Using a module that is
not suited for the node on which the job is running, can give suboptimal
performance or may even completely fail to work.

This is why modules are organized in different *software stacks*,
differentiated by the operating system, the architecture, and the toolchain
version. The good news is that in general you do not need to worry too much
about this, as in most cases the correct software stack will be made available
in a job automatically thanks to the *cluster* modules, as explained in the
:ref:`next section <cluster_modules>`.

If you are interested in more technical details, you can read the section on
:ref:`manually modifying the modulepath <manually_modifying_modulepath>`,
which is oriented towards advanced users.


.. _cluster_modules:

The cluster modules
-------------------

Background: a given module will only be available if it is located inside a
directory contained in the ``$MODULEPATH`` environment variable.
This ``$MODULEPATH`` environment variable is a colon-separated list of
directories, and you can list all modules located inside those directories
with the ``module avail`` command. The different software stacks mentioned
earlier are located in different directories (see the
:ref:`next section <manually_modifying_modulepath>` for more details), so in
order to make sure you are loading modules from the appropriate software stack,
the ``$MODULEPATH`` variable needs to contain the appropriate paths for the
node where you want to use a module.

Because working with the different directories containing different software
stacks is cumbersome, we advise users to rely on the cluster module to set
the ``$MODULEPATH`` variable. The cluster module can be handled identically
as other modules, but instead of making executables or libraries available,
its only purpose is to set up your environment to make the correct modules
available. The cluster module is always available and you can see which
versions can be loaded by executing ``module avail cluster``.

On the login nodes and inside a job environment, the correct version of the
cluster module will be loaded automatically. This means that for these cases,
you do not need to take any special action: the modules from the appropriate
software stack will be the only ones available to you. There is hence no need
for ``module use/unuse`` commands in your jobscripts (unless you deal with an
exceptional case).

.. note::

   The appropriate cluster module is only loaded automatically inside a job
   environment and on the login nodes. In other cases (for example when you
   ssh directly into a node), you will need to first load a cluster module
   yourself in order to make the correct modules available.

.. warning::

   If your jobscript contains the command ``module --force purge``, the
   cluster module will be unloaded and your ``$MODULEPATH`` will not contain
   the directory with the appropriate software stack. It will be necessary to
   load the correct cluster module or set your ``$MODULEPATH`` in another way.
   This is why we advise to not use ``module --force purge`` in your jobs,
   unless you are well aware of the consequences. Note that it is ok to
   execute ``module purge``, since the cluster modules are
   :ref:`sticky <module_purge>`.

A common scenario is that you want to search through the installed modules for
a software package you need, while you are on a login node. There are two ways
this can be done. In the example below we assume the commands are executed on
a Genius login node. The software package that is used as an example is
called ``CP2K``.

The first option is to load the cluster module corresponding to the node where
you eventually want to use a certain software package. If you are planning to
run jobs on the wICE batch partition, the commmand is:

.. code-block:: shell

   $ module load cluster/wice/batch

Note that the previously loaded cluster module will be automatically unloaded:
at most one cluster module can be loaded at a time. Now you can search for
modules containing ``CP2K`` by executing (the search is not case sensitive):

.. code-block:: shell

   $ module avail CP2K
   -- /apps/leuven/rocky8/icelake/2021a/modules/all --
      CP2K/8.2-foss-2021a         Libint/2.6.0-GCC-10.3.0-lmax-6-cp2k
      CP2K/8.2-intel-2021a (D)    Libint/2.6.0-iimpi-2021a-lmax-6-cp2k
      Libint/2.6.0-intel-compilers-2021.2.0-lmax-6-cp2k (D)

A second approach to search for installed software, is to use the
``module spider`` command. In contrast to the ``module avail`` command, with
``module spider`` Lmod will not only search for available modules (meaning
modules inside directories included in the current value of ``$MODULEPATH``),
but additionally will take into account additional entries that would be added
to ``$MODULEPATH`` in case a cluster module would be loaded. An example is:

.. code-block::

   $ module spider CP2K
   -------------------------------------
     CP2K:
   -------------------------------------
   Description:
         CP2K is a freely available (GPL) program, ...
   Versions:
           CP2K/5.1-intel-2018a
           CP2K/6.1-foss-2018a
           CP2K/6.1-intel-2018a
           CP2K/7.1-foss-2019b
           CP2K/7.1-intel-2019b
           CP2K/8.2-foss-2021a
           CP2K/8.2-intel-2021a
   -------------------------------------
     For detailed information about a specific "CP2K" package (including how
     to load the modules) use the module's full name.
     Note that names that have a trailing (E) are extensions provided by other
     modules. For example:
        $ module spider CP2K/8.2-intel-2021a
   -------------------------------------

As suggested by the output, you can obtain more information about one
of the available versions of the ``CP2K`` module by executing:

.. code-block:: shell

   $ module spider CP2K/8.2-intel-2021a

   -------------------------------------
     CP2K: CP2K/8.2-intel-2021a
   -------------------------------------
       Description:
         CP2K is a freely available (GPL) program, ...


    You will need to load all module(s) on any one of the lines below before
    the "CP2K/8.2-intel-2021a" module is available to load

      cluster/genius/amd
      cluster/genius/amd_long
      cluster/genius/batch
      ...
      cluster/wice/batch
      ...

This command shows which cluster modules will make the ``CP2K/8.2-intel-2021a``
module available. As discussed earlier, loading ``cluster/wice/batch`` is one
example of a cluster module that suffices to make ``CP2K/8.2-intel-2021a``
available. For more information about ``module spider``, have a look at the
`Lmod documentation page <https://lmod.readthedocs.io/en/latest/135_module_spider.html>`__

.. note::

   In contrast to previous behavior, modules from different toolchain versions
   are now available automatically. On Genius, all modules since 2018a
   are available, and on wICE, all modules starting from 2021a. For a few
   legacy modules, installation is impossible on a recent operating system. In
   such a case, it is recommended to use a replacement module from a newer
   toolchain version. Alternatively you can consider to run your legacy
   software inside a container, but this is only the best option in some
   specific cases.


.. _manually_modifying_modulepath:

Manually modifying the modulepath
---------------------------------

As discussed in the previous section, the recommended approach to set your
``$MODULEPATH`` environment variable, is by using the cluster module. This
will make modules from the correct software stack available. It is however
also possible to manually modify the path where modules are searched.

Each software stack is located in a directory with the following hierarchical
structure::

   /apps/leuven/${VSC_OS_LOCAL}/${VSC_ARCH_LOCAL}${VSC_ARCH_SUFFIX}/TOOLCHAIN_VERSION/modules/all

e.g.:

.. code-block:: shell

   /apps/leuven/rocky8/skylake/2018a/modules/all

This convention is in line with other VSC sites and will also be used on wICE
and future clusters. In order to add such a directory to your modulepath, the
following command can be used:

.. code-block:: shell

   module use /apps/leuven/rocky8/skylake/2018a/modules/all

To remove the entry again:

.. code-block:: shell

   module unuse /apps/leuven/rocky8/skylake/2018a/modules/all

Keep in mind that also ``/apps/leuven/common/modules/all`` is part of your
default ``$MODULEPATH``. This module collection is intended for packages which
have no operating system or toolchain dependencies. Typical examples are
packages which are distributed as precompiled binaries such as FLUENT.

.. _specialized software stacks:

Specialized software stacks
---------------------------

The list of software available on a particular cluster can be
unwieldingly long and the information that ``module av`` produces
overwhelming. Therefore the administrators may have chosen to only show
the most relevant packages by default, and not show, e.g., packages that
aim at a different cluster, a particular node type or a less complete
toolchain. Those additional packages can then be enabled by loading
another module first. E.g., to get access to the modules in
the (at the time of writing) incomplete 2019a toolchain on UAntwerpen's
leibniz cluster, one should first enter

   ::

      $ module load leibniz/2019a-experimental

