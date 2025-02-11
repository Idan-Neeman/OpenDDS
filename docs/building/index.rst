#######################
Building and Installing
#######################

.. toctree::
  :hidden:

  dependencies
  cmake
  android
  ios

*******************
Supported Platforms
*******************

We have built OpenDDS on number of different platforms and compilers.
See :ghfile:`README.md#supported-platforms` for a complete description of supported platforms.
See :ref:`cross-compiling` for how to cross compile for other platforms.

************************
Configuring and Building
************************

.. ifconfig:: is_release

  If not already done, download the source from :ghrelease:`GitHub`.

Use the :ghfile:`configure` script to prepare to build OpenDDS.
This script requires :ref:`deps-perl`.

.. tab:: Linux, macOS, BSDs, etc.

  To start the script change to the root of the OpenDDS source directory and run:

  .. code-block:: bash

    ./configure

.. tab:: Windows

  `Strawberry Perl <https://strawberryperl.com>`__ is recommended for Windows.

  To start the script open a `Visual Studio Developer Command Prompt <https://learn.microsoft.com/en-us/visualstudio/ide/reference/command-prompt-powershell>`__ that has C++ tools available, then change to the root of the OpenDDS source directory and run:

  .. code-block:: batch

    configure

Optionally add ``--help`` to the command line to see the advanced options available for this script.
The configure script will download :ref:`ACE/TAO <deps-ace-tao>` and configure it for your platform.
To use an existing ACE/TAO installation, either set the :envvar:`ACE_ROOT` and :envvar:`TAO_ROOT` environment variables or pass the ``--ace`` and ``--tao`` (if TAO is not at ``$ACE_ROOT/TAO``) options to configure.

.. seealso:: :doc:`dependencies` for a full list of dependencies including ones that can be configured with the configure script.

If configure runs successfully it will end with a message about the next steps for compiling OpenDDS.

.. tab:: Linux, macOS, BSDs, etc.

  OpenDDS on these platforms must be built using GNU Make:

  .. code-block::

    make -j 4

  OpenDDS supports parallel builds to speed up the build when using Make.
  Above 4 is used as an example for the max number of parallel jobs.
  If unsure what this number should be, use the number of CPU cores on the machine.

  The configure script creates an environment setup file called ``setenv.sh`` that sets all the environment variables the build and test steps rely on.
  The main makefile sets these itself, so ``setenv.sh`` is not needed when running ``make`` from the top level.
  It needs to be sourced before building other projects and running tests:

  .. code-block:: shell

    source setenv.sh

.. tab:: Windows

  The configure script will say how to open the solution file for OpenDDS in Visual Studio using ``devenv``.

  It can also be built directly from the command prompt by using MSBuild.
  For example, if the configure script was ran without any arguments, to do a Debug x64 build:

  .. code-block:: batch

    msbuild -p:Configuration=Debug,Platform=x64 -m DDS_TAOv2.sln

  .. seealso:: `Microsoft MSBuild Documentation <https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild>`__

  The configure script creates an environment setup file called ``setenv.cmd`` that sets all the environment variables the build and test steps rely on.
  For the command prompt that ran the configure script, these variables were set automatically.
  If running in another command prompt, the variables need to be set again before building other projects and running tests:

  .. code-block::

    setenv

Java
====

If you're building OpenDDS for use by :ref:`Java applications <java>`, please see the file :ghfile:`java/INSTALL` instead of this one.

.. _building-sec:

Security
========

..
    Sect<14.1>

:ref:`sec` is disabled by default, and must be enabled by passing ``--security`` to the configure script.
It requires :ref:`deps-xerces` and :ref:`deps-openssl`.

.. tab:: Linux, macOS, BSDs, etc.

  .. tab:: Installed from a Package Manager

    Most package managers should have Xerces and OpenSSL packages that can be installed if not already:

    - Debian/Ubuntu-based: ``sudo apt install libxerces-c-dev libssl-dev``
    - Redhat/Fedora-based: ``sudo yum install xerces-c-devel openssl-devel``
    - Homebrew ``brew install xerces-c openssl@3``

  .. tab:: Built from Source

    Download source and build according to instructions.

  If the libraries didn't get installed to ``/usr``, then the installation prefixes will have to be passed using ``--xerces`` and ``--openssl``.

.. tab:: Windows

  .. tab:: Installed from a Package Manager

    **Using Microsoft vcpkg**

    Microsoft vcpkg is a "C++ Library Manager for Windows, Linux, and macOS" which helps developers build/install dependencies.
    Although it is cross-platform, this guide only discusses vcpkg on Windows.

    As of this writing, vcpkg is only supported on Visual Studio 2015 Update 3 and later versions; if using an earlier version of Visual Studio, skip down to the manual setup instructions later in this section.

    * If OpenDDS tests will be built, install CMake or put the one that comes with Visual Studio on the ``PATH`` (see ``Common7\IDE\CommonExtensions\Microsoft\CMake``).

    * If you need to obtain and install vcpkg, navigate to `https://github.com/Microsoft/vcpkg <https://github.com/Microsoft/vcpkg>`__ and follow the instructions to obtain vcpkg by cloning the repository and bootstrapping it.

    * Fetch and build the dependencies; by default, vcpkg targets x86 so be sure to specify the x64 target if required by specifying it when invoking vcpkg install, as shown here:

      .. code-block:: batch

          vcpkg install openssl:x64-windows xerces-c:x64-windows

    * Configure OpenDDS by passing the ``--openssl`` and ``--xerces3`` options.
      As a convenience, it can be helpful to set an environment variable to store the path since it is the same location for both dependencies.

      .. code-block:: batch

          set VCPKG_INSTALL=c:\path\to\vcpkg\installed\x64-windows
          configure --security --openssl=%VCPKG_INSTALL% --xerces3=%VCPKG_INSTALL%

    * Compile with ``msbuild``:

      .. code-block:: batch

          msbuild /m DDS_TAOv2_all.sln

      Or by launching Visual Studio from this command prompt so it inherits the correct environment variables and building from there:

      .. code-block:: batch

          devenv DDS_TAOv2_all.sln

  .. tab:: Built from Source

    .. note::

       For all of the build steps listed here, check that each package targets the same architecture (either 32-bit or 64-bit) by compiling all dependencies within the same type of Developer Command Prompt.

    **Compiling OpenSSL**

    Official OpenSSL instructions can be found `here <https://wiki.openssl.org/index.php/Compilation_and_Installation#Windows>`__.

    #. Install :ref:`deps-perl` and add it to the ``PATH`` environment variable.

    #. Install Netwide Assembler (NASM).
       Click through the latest stable release and there is a win32 and win64 directory containing executable installers.
       The installer does not update the Path environment variable, so a manual entry (``%LOCALAPPDATA%\bin\NASM``) is necessary.

    #. Download the required version of OpenSSL by cloning the repository.

    #. Open a Developer Command Prompt (32-bit or 64-bit depending on the desired target architecture) and change into the freshly cloned openssl directory.

    #. Run the configure script and specify a required architecture (``perl Configure VC-WIN32`` or ``perl Configure VC-WIN64A``).

    #. Run ``nmake``

    #. Run ``nmake install``

    .. note::

       If the default OpenSSL location is desired, which will be searched by OpenDDS, open the "Developer Command Prompt" as an administrator before running the install.
       It will write to ``C:\Program Files`` or ``C:\Program Files (x86)`` depending on the architecture.

    **Compiling Xerces-C++ 3**

    Official Xerces instructions can be found `here <https://xerces.apache.org/xerces-c/build-3.html>`__.

    #. Download/extract the Xerces source files.

    #. Create a cmake build directory and change into it (from within the Xerces source tree).

       .. code-block:: batch

           mkdir build
           cd build

    #. Run cmake with the appropriate generator.
       In this case Visual Studio 2017 with 64-bit is being used so:

       .. code-block:: batch

           cmake -G "Visual Studio 15 2017 Win64" ..

    #. Run cmake again with the build switch and install target (this should be done in an administrator command-prompt to install in the default location as mentioned above).

       .. code-block:: batch

           cmake --build . --target install

    **Configuring and Building OpenDDS**:

    #. Change into the OpenDDS root folder and run configure with security enabled.

       * If the default location was used for OpenSSL and Xerces, configure should automatically find the dependencies:

         .. code-block:: batch

             configure --security

    #. If a different location was used (assuming environment variables ``NEW_SSL_ROOT`` and ``NEW_XERCES_ROOT`` point to their respective library directories):

       .. code-block:: batch

           configure --security --openssl=%NEW_SSL_ROOT% \
             --xerces3=%NEW_XERCES_ROOT%

    #. Compile with msbuild (or by opening the solution file in Visual Studio and building from there).

       .. code-block:: batch

           msbuild /m DDS_TAOv2_all.sln

.. _cross-compiling:

Cross Compiling
===============

Use the configure script, and set the target platform to one different than the host.
For example:

.. code-block:: shell

  ./configure --target=lynxos-178

Run configure with ``--target-help`` for details on the supported targets.
In this setup, configure will clone the OpenDDS and ACE/TAO source trees for host and target builds.
It will do a static build of the host tools (such as ``opendds_idl`` and ``tao_idl``) in the host environment, and a full build in the target environment.
Most parameters to configure are then assumed to be target parameters.

Any testing has to be done manually.

Raspberry Pi
------------

The instructions for building for the Raspberry Pi are on `opendds.org <https://opendds.org/quickstart/GettingStartedPi.html>`__.

Android
-------

Android support is documented in :doc:`android`.

Apple iOS
---------

Apple iOS support is documented in :doc:`ios`.

.. _install:

************
Installation
************

When OpenDDS is built using ``make``, if the configure script was run with an argument of ``--prefix=<prefix>`` the ``make install`` target is available.

After running ``make`` (and before ``make install``) you have one completely ready and usable OpenDDS.
Its ``DDS_ROOT`` is the top of the source tree -- the same directory from which you ran configure and make.
That ``DDS_ROOT`` should work for building application code, and some users may prefer using it this way.

After ``make install`` there is a second completely ready and usable OpenDDS that's under the installation prefix directory.
It contains the required libraries, code generators, header files, IDL files, and associated scripts and documentation.

.. note:: If configured with RapidJSON, OpenDDS will install the headers for RapidJSON, which might conflict with an existing installation.

Using an Installed OpenDDS
==========================

After ``make install`` completes, the shell script in ``<prefix>/share/dds/dds-devel.sh`` is used to set the ``DDS_ROOT`` environment variable.
The analogous files for ACE and TAO are ``<prefix>/share/ace/ace-devel.sh`` and ``<prefix>/share/tao/tao-devel.sh``.

The ``<prefix>`` tree does not contain a tool for makefile generation.
To use MPC to generate application makefiles, the ``MPC_ROOT`` subdirectory from the OpenDDS source tree can be used either in-place or copied elsewhere.
To use CMake to generate application makefiles, see :doc:`cmake`.

*****
Tests
*****

Tests are not built by default, ``--tests`` must be passed to the configure script.
All tests can be run using :ghfile:`tests/auto_run_tests.pl`.
See :doc:`/internal/running_tests` for running all tests or individual tests.
