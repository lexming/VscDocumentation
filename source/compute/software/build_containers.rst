.. _build_containers:

##############################
Building Containers on the HPC
##############################

.. seealso::

   Section :ref:`use_containers` shows how to use ready available containers
   on the VSC clustere

How can I create a Apptainer image?
===================================

You have three options to build images, locally on your  machine, in the
cloud or on the VSC infrastructure.


Building on VSC infrastructure
------------------------------

Given that most build procedures require superuser privileges, your options
on the VSC infrastructure are limited.  You can build an image from a Docker
container, e.g., to build an image that contains a version of TensorFlow 
and has Jupyter as well, use::

   $ export APPTAINER_TMPDIR=$VSC_SCRATCH/apptainer_tmp
   $ mkdir -p $APPTAINER_TMPDIR
   $ export APPTAINER_CACHEDIR=$VSC_SCRATCH/apptainer_cache
   $ mkdir -p $APPTAINER_CACHEDIR
   $ apptainer build tensorflow.sif docker://tensorflow/tensorflow:latest-jupyter

.. warning::

   Don't forget to define and create the ``$APPTAINER_TMPDIR`` and
   ``$APPTAINER_CACHEDIR`` since if you fail to do so, Apptainer
   will use directories in your home directory, and you will exceed
   the quota on that file system.

   Also, images tend to be very large, so store them in a directory
   where you have sufficient quota, e.g., ``$VSC_DATA``.


This approach will serve you well if you can use either prebuilt images
or Docker containers.  If you need to modify an existing image or
container, you should consider the alternatives.

.. note::

   Creating image files may take considerable time and resources. It is good
   practice to do this on a compute node, rather than on a login node.


Local builds
------------

The most convenient way to create an image is on your own machine, since
you will have superuser privileges, and hence the most options to chose
from.  At this point, Apptainer only runs under Linux, so you would
have to use a virtual machine when using Windows or macOS.  For detailed
instructions, see the `Apptainer Quick Start`_ guide.

Besides building images from Docker containers, you have the option to
create them from a definition file, which allows you to completely customize
your image.  We provide a brief :ref:`introduction to Apptainer definition files
<apptainer_definition_files>`, but for more details, we refer you to the
documentation on `Apptainer Definition Files`_.

When you have a Apptainer definition file, e.g., ``my_image.def``, you can
build your image file ``my_image.sif``::

   your_machine> apptainer build my_image.sif my_image.def

Once your image is built, you can :ref:`transfer <data transfer>`
it to the VSC infrastructure to use it.

.. warning::

   Since Apptainer images can be very large, transfer your image
   to a directory where you have sufficient quota, e.g.,
   ``$VSC_DATA``.


Remote builds
-------------

You can build images on the Apptainer website, and download
them to the VSC infrastructure.  You will have to create an account
at Sylabs.  Once this is done, you can use `Sylabs Remote Builder`_
to create an image based on an `Apptainer definition file
<apptainer_definition_files>`.  If the build succeeds, you can
pull the resulting image from the library::

   $ export APPTAINER_CACHEDIR=$VSC_SCRATCH/apptainer_cache
   $ mkdir -p $APPTAINER_CACHEDIR
   $ apptainer pull library://gjbex/remote-builds/rb-5d6cb2d65192faeb1a3f92c3:latest

.. warning::

   Don't forget to define and create the ``$APPTAINER_CACHEDIR``
   since if you fail to do so, Apptainer will use directories in
   your home directory, and you will exceed the quota on that file
   system.

   Also, images tend to be very large, so store them in a directory
   where you have sufficient quota, e.g., ``$VSC_DATA``.

Remote builds have several advantages:

- you only need a web browser to create them, so this approach is
  platform-independent,
- they can easily be shared with others.

However, local builds still offer more flexibility, especially when
some interactive setup is required.


.. _apptainer_definition_files:

Apptainer definition files
==========================

Below is an example of a Apptainer definition file::

   Bootstrap: docker
   From: ubuntu:xenial
   
   %post
       apt-get update
       apt-get install -y grace
               
   %runscript
       /usr/bin/grace

The resulting image will be based on the Ubuntu Xenial Xerus distribution
(16.04).  Once it is bootstrapped, the command in the ``%post`` section of
the definition file will be executed.  For this example, the Grace plotting
package will be installed.

.. note::

   This example is intended to illustrate that very old software that
   is no longer maintained can successfully be run on modern infrastructure.
   It is by no means intended to encourage you to start using Grace.

Apptainer definition files are very flexible. For more details,
we refer you to the documentation on `Apptainer Definition Files`_.

An important advantage of definition files is that they can easily
be shared, and improve reproducibility.


Conda environment in a definition file
--------------------------------------
:ref:`Conda environments<conda for Python>`
are a convenient solution when it comes to handling own Python-dependent
software installations. Having a containerized conda environment is often
useful for groups when working collectively on a common project.
One way to have a conda environment in a container is to create it from
an existing environment YAML file. If we have a conda environment exported
in a YAML format file called, e.g., ``user_conda_environment.yml``, then
from that file one can recreate the same environment in a Apptainer definition file::

   Bootstrap: docker
   From: continuumio/miniconda3

   %files
       user_conda_environment.yml

   %post
       /opt/conda/bin/conda env create -n user_conda_environment -f user_conda_environment.yml

   %runscript
       . /opt/conda/etc/profile.d/conda.sh
       conda activate user_conda_environment
       exec "$@"

The ``exec "$@"`` line will accept the user's input command, e.g.,
``python --version`` or ``R --version``, when running the container.

.. note::

   Creating a container with a conda environment in it can ask for a lot of memory.
   Therefore, that procedure might be best done on a compute node and not
   on the cluster login nodes.

