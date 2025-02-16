.. _conan_tools_gnu_autotools_toolchain:

AutotoolsToolchain
==================

.. warning::

    These tools are **experimental** and subject to breaking changes.


The ``AutotoolsToolchain`` is the toolchain generator for Autotools. It will generate shell scripts containing
environment variable definitions that the autotools build system can understand.

The ``AutotoolsToolchain`` generator can be used by name in conanfiles:

.. code-block:: python
    :caption: conanfile.py

    class Pkg(ConanFile):
        generators = "AutotoolsToolchain"

.. code-block:: text
    :caption: conanfile.txt

    [generators]
    AutotoolsToolchain

And it can also be fully instantiated in the conanfile ``generate()`` method:

.. code:: python

    from conans import ConanFile
    from conan.tools.gnu import AutotoolsToolchain

    class App(ConanFile):
        settings = "os", "arch", "compiler", "build_type"

        def generate(self):
            tc = AutotoolsToolchain(self)
            tc.generate()

The ``AutotoolsToolchain`` will generate after a ``conan install`` command the *conanautotoolstoolchain.sh* or *conanautotoolstoolchain.bat* files:

.. code-block:: bash

    $ conan install conanfile.py # default is Release
    $ source conanautotoolstoolchain.sh
    # or in Windows
    $ conanautotoolstoolchain.bat

This generator will define aggregated variables ``CPPFLAGS``, ``LDFLAGS``, ``CXXFLAGS``, ``CFLAGS`` that
accumulate all dependencies information, including transitive dependencies, with flags like ``-stdlib=libstdc++``, ``-std=gnu14``, architecture flags, etc.

This generator will also generate a file called ``conanbuild.json`` containing two keys:

- **configure_args**: Arguments to call the ``configure`` script.
- **make_args**: Arguments to call the ``make`` script.

The :ref:`Autotools build helper<conan_tools_gnu_build_helper>` will use that ``conanbuild.json`` file to seamlessly call
the configure and make script using these precalculated arguments.

Attributes
++++++++++

You can change some attributes before calling the ``generate()`` method if you want to change some of the precalculated
values:

.. code:: python

    from conans import ConanFile
    from conan.tools.gnu import AutotoolsToolchain

    class App(ConanFile):
        settings = "os", "arch", "compiler", "build_type"

        def generate(self):
            tc = AutotoolsToolchain(self)
            tc.configure_args.append("--my_argument")
            tc.generate()


* **configure_args** (Defaulted to ``[]``): Additional arguments to be passed to the configure script.
* **make_args** (Defaulted to ``[]``): Additional arguments to be passed to he make script.
* **defines** (Defaulted to ``[]``): Additional defines.
* **cxxflags** (Defaulted to ``[]``): Additional cxxflags.
* **cflags** (Defaulted to ``[]``): Additional cflags.
* **ldflags** (Defaulted to ``[]``): Additional ldflags.
* **default_configure_install_args** (Defaulted to ``False``): If True it will pass automatically the following flags to the configure script:

   * ``--prefix``: With the self.package_folder value.
   * ``--bindir=${prefix}/bin``
   * ``--sbindir=${prefix}/bin``
   * ``--libdir=${prefix}/lib``
   * ``--includedir=${prefix}/include``
   * ``--oldincludedir=${prefix}/includev``
   * ``--datarootdir=${prefix}/share``

* **ndebug**: "NDEBUG" if the ``settings.build_type`` != `Debug`.
* **gcc_cxx11_abi**: "_GLIBCXX_USE_CXX11_ABI" if ``gcc/libstdc++``.
* **libcxx**: Flag calculated from ``settings.compiler.libcxx``.
* **fpic**: True/False from ``options.fpic`` if defined.
* **cppstd**: Flag from ``settings.compiler.cppstd``
* **arch_flag**: Flag from ``settings.arch``
* **build_type_flags**: Flags from ``settings.build_type``
* **apple_arch_flag**: Only when cross-building with Apple systems. Flags from ``settings.arch``.
* **apple_isysroot_flag**: Only when cross-building with Apple systems. Path to the root sdk.
