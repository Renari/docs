CMakeDeps
---------

.. warning::

    These tools are **experimental** and subject to breaking changes.


Available since: `1.33.0 <https://github.com/conan-io/conan/releases/tag/1.33.0>`_

The ``CMakeDeps`` helper will generate one **xxxx-config.cmake** file per dependency, together with other necessary *.cmake* files
like version, flags and directory data or configuration. It can be used like:


.. code-block:: python

    from conans import ConanFile

    class App(ConanFile):
        settings = "os", "arch", "compiler", "build_type"
        requires = "hello/0.1"
        generators = "CMakeDeps"


The full instantiation, that allows custom configuration can be done in the ``generate()`` method:


.. code-block:: python

    from conans import ConanFile
    from conan.tools.cmake import CMakeDeps

    class App(ConanFile):
        settings = "os", "arch", "compiler", "build_type"
        requires = "hello/0.1"

        def generate(self):
            cmake = CMakeDeps(self)
            cmake.generate()

There are some attributes you can adjust in the created ``CMakeDeps`` object to change the default behavior:

configurations
++++++++++++++

Allows to define custom user CMake configurations besides the standard
Release, Debug, etc ones. If the **settings.yml** file is customized to add new configurations to the
``settings.build_type``, then, adding it explicitly to ``.configurations`` is not necessary.

.. code-block:: python

    def generate(self):
        cmake = CMakeDeps(self)
        cmake.configurations.append("ReleaseShared")
        if self.options["hello"].shared:
            cmake.configuration = "ReleaseShared"
        cmake.generate()


build_context_activated
+++++++++++++++++++++++

When you have a **build-require**, by default, the config files (`xxx-config.cmake`) files are not generated.
But you can activate it using the **build_context_activated** attribute:

.. code-block:: python

    build_requires = ["my_tool/0.0.1"]

    def generate(self):
        cmake = CMakeDeps(self)
        # generate the config files for the build require
        cmake.build_context_activated = ["my_tool"]
        cmake.generate()


build_context_suffix
++++++++++++++++++++

When you have the same package as a **build-require** and as a **regular require** it will cause a conflict in the generator
because the file names of the config files will collide as well as the targets names, variables names etc.

For example, this is a typical situation with some requirements (capnproto, protobuf...) that contain
a tool used to generate source code at build time (so it is a **build_require**),
but also providing a library to link to the final application, so you also have a **regular require**.
Solving this conflict is specially important when we are cross-building because the tool
(that will run in the building machine) belongs to a different binary package than the library, that will "run" in the
host machine.

You can use the **build_context_suffix** attribute to specify a suffix for a requirement,
so the files/targets/variables of the requirement in the build context (build require) will be renamed:

.. code-block:: python

    build_requires = ["my_tool/0.0.1"]
    requires = ["my_tool/0.0.1"]

    def generate(self):
        cmake = CMakeDeps(self)
        # generate the config files for the build require
        cmake.build_context_activated = ["my_tool"]
        # disambiguate the files, targets, etc
        cmake.build_context_suffix = {"my_tool": "_BUILD"}
        cmake.generate()



build_context_build_modules
+++++++++++++++++++++++++++

Also there is another issue with the **build_modules**. As you may know, the recipes of the requirements can declare a
`cppinfo.build_modules` entry containing one or more **.cmake** files.
When the requirement is found by the cmake ``find_package()``
function, Conan will include automatically these files.

By default, Conan will include only the build modules from the
``host`` context (regular requires) to avoid the collision, but you can change the default behavior.

Use the **build_context_build_modules** attribute to specify require names to include the **build_modules** from
**build_requires**:

.. code-block:: python

    build_requires = ["my_tool/0.0.1"]

    def generate(self):
        cmake = CMakeDeps(self)
        # generate the config files for the build require
        cmake.build_context_activated = ["my_tool"]
        # Choose the build modules from "build" context
        cmake.build_context_build_modules = ["my_tool"]
        cmake.generate()

Properties
++++++++++

The following properties affect the CMakeDeps generator:

- **cmake_file_name**: The config file generated for the current package will follow the ``<VALUE>-config.cmake`` pattern,
  so to find the package you write ``find_package(<VALUE>)``.
- **cmake_target_name**: Name of the target to be consumed. When set on the root ``cpp_info``,
  it changes the namespace of the target. When set to a component, it changes the name of the target
  (see the example below).
- **cmake_build_modules**: List of ``.cmake`` files (route relative to root package folder) that are automatically
  included when the consumer run the ``find_package()``.
- **skip_deps_file**: It tells the ``CMakeDeps`` generator to skip the creation of files for the package declaring
  this property. It can be used, for instance, to create a system wrapper package so the consumers find
  the config files in the CMake installation config path and not in the generated by Conan (because it has been skipped).

Example:

.. code-block:: python

    def package_info(self):
        ...
        # MyFileName-config.cmake
        self.cpp_info.set_property("cmake_file_name", "MyFileName")
        # Foo:: namespace for the targets (Foo::Foo if no components)
        self.cpp_info.set_property("cmake_target_name", "Foo")
        # Foo::Var target name for the component "mycomponent"
        self.cpp_info.components["mycomponent"].set_property("cmake_target_name", "Var")
        # Automatically include the lib/mypkg.cmake file when calling find_package()
        self.cpp_info.components["mycomponent"].set_property("cmake_build_modules", [os.path.join("lib", "mypkg.cmake")])

        # Skip this package when generating the files for the whole dependency tree in the consumer
        # note: it will make useless the previous adjustements.
        self.cpp_info.set_property("skip_deps_file", True)
