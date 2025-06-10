.. _use_containers:

#############################
Containers on the HPC systems
#############################

The best-known container implementation is undoubtedly `Docker`_.  However,
Docker needs to run as the *root* superuser of the system which has several
security implications. Hence, HPC sites do not typically allow users to run
Docker containers.

Fortunately, `Apptainer`_ provides an alternative and safer approach for
containers that can be used by any regular user without *root* permissions.
Since Apptainer also provides the options to build images from Docker container
files, it is a suitable replacement for Docker itself. Therefore, Apptainer is
fully supported on all VSC clusters.


When should I use containers?
=============================

If the software you intend to use is available on the VSC infrastructure,
don't use containers.  This software has been built to use specific
hardware optimizations, while software deployed via containers is
typically built for the common denominator.

Good use cases include:

* Containers can be useful to run software that is hard to install
  on HPC systems, e.g., GUI applications, legacy software, and so on.

* Containers can be useful to deal with compatibility issues between
  Linux flavors.

* You want to create a workflow that can run on VSC infrastructure,
  but can also be burst to a third-party compute cloud (e.g., AWS
  or Microsoft Azure) when required.

* You want to maximize the period your software can be run in a
  reproducible way.

How can I run a Apptainer image?
================================

Once you have an image, there are several options to run the container.

#. You can invoke any application that is in the ``$PATH`` of the
   container, e.g., for the image containing Grace::

   $ apptainer  exec  grace.sif  xmgrace

#. In case the definition file specified a ``%runscript`` directive,
   this can be executed using::

   $ apptainer  run  grace.sif

#. The container can be run as a shell::

   $ apptainer  shell  grace.sif

By default, your home directory in the container will be mounted
with the same path as it has on the host.  The current working
directory in the container is that on the host in which you
invoked ``apptainer``.

.. note::

   Although you can move to a parent directory of the current working
   directory in the container, you will not see its contents on the host.
   Only the current working directory and its sub-directories on the host
   are mounted.

Additional host directories can be mounted in the container as well by
using the ``-B`` option.  Mount points are created dynamically (using
overlays), so they do not have to exist in the image.  For example,
to mount the ``$VSC_SCRATCH`` directory, you would use::

   $ apptainer  exec  -B $VSC_SCRATCH:/scratch  grace.sif  xmgrace

Your ``$VSC_SCRATCH`` directory is now accessible from within the
image in the directory ``/scratch``.

.. note::

   If you want existing scripts to work from within the image without
   having to change paths, it may be convenient to use identical
   mount points in the image and on the host, e.g., for the
   ``$VSC_DATA`` directory::

      $ apptainer  exec  -B $VSC_DATA:$VSC_DATA  grace.sif  xmgrace

   Or, more concisely::

      $ apptainer  exec  -B $VSC_DATA  grace.sif  xmgrace

   The host environment variables are defined in the image, hence
   scripts that use those will work.


Can I use apptainer images in a job?
------------------------------------

Yes, you can.  Apptainer images can be part of any workflow, e.g.,
the following script would create a plot in the Grace container::

   #!/bin/bash -l
   #PBS -l nodes=1:ppn=1
   #PBS -l walltime=00:30:00
   
   cd $PBS_O_WORKDIR
   apptainer exec grace.sif gracebat -data data.dat \
                                       -batch plot.bat
   
Ensure that the container has access to all the required directories
by providing additional bindings if necessary.


Can I run parallel applications using a Apptainer image?
----------------------------------------------------------

For shared memory applications there is absolutely no problem.

For distributed applications it is highly recommended to use
the same implementation and version of the MPI libraries on
the host and in the image.  You also want to install the
appropriate drivers for the interconnect, as well as the
low-level communication libraries, e.g., ibverbs.

For this type of scenario, it is probably best to contact :ref:`user
support <user support VSC>`.

.. note::

   For distributed applications you may expect some mild performance
   degradation.


Can I run a service from a Apptainer image?
---------------------------------------------

Yes, it is possible to run services such as databases or web
applications that are installed in Apptainer images.

For this type of scenario, it is probably best to contact :ref:`user
support <user support VSC>`.


