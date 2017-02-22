# CMake Tutorial

## 1. A Basic Starting Point
The most basic project is an executable built from source files. Here is our `CMakeLists.txt` starting point:

```CMake
cmake_minimum_required (VERSION 2.6)
project (Tutorial)
add_executable(Tutorial tutorial.cpp)
```
    
* `cmake_minimum_required (VERSION X.Y)` - minimum ver of CMake req
* `project (ProjectName)` - set CMake project name, build dir (`PROJECT_BINARY_DIR`), and source dir (`PROJECT_SOURCE_DIR`)
* `add_executable (ExecutableName source_name.cpp ...)` - build executable `ExecutableName` from sources `source_name.cpp ...`

---

Suppose we have the following file, `tutorial.cpp`:

```C++	
#include <iostream>
#include <cmath>

using namespace std;

int main (int argc, char *argv[]) {
    if (argc < 2) {
	    cout << "Usage: " << argv[0] << " number\n";
	    return 1;
	}
	
	double inputValue = atof(argv[1]);
	double outputValue = sqrt(inputValue);
	cout << "The square root of " << inputValue << " is " << outputValue << endl;
	
	return 0;
}
```

Our CMake project `Tutorial`'s `CMakeLists.txt` file will look like this:

```CMake
cmake_minimum_required (VERSION 2.6)

project (Tutorial)

# The version number
set (Tutorial_VERSION_MAJOR 1)
set (Tutorial_VERSION_MINOR 0)

# Configure a header file to pass some of CMake's settings
# to the source code
configure_file (
	"${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
    "${PROJECT_BINARY_DIR}/TutorialConfig.h"
)

# Add the binary tree to the search path for include files
# so that we'll find TutorialConfig.h
include_directories("${PROJECT_BINARY_DIR}"

add_executable(Tutorial tutorial.cpp)
```

* `set (VARIABLE_NAME value)` - set `VARIABLE_NAME` in `CMakeLists.txt` to `value`
* `configure_file ( TEMPLATE_PATH OUTPUT_PATH )` - render the template in `TEMPLATE_PATH` with values `set` in `CMakeLists.txt` and output into `OUTPUT_PATH`
* `include_directories( INCLUDE_DIR ... )` - search `INCLUDE_DIR ...` for include files

The template file for our project is called `TutorialConfig.h.in`, and it looks like this:
```CMake
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```
Just to make sure you understand, `@Tutorial_VERSION_MAJOR@` and `@Tutorial_VERSION_MINOR@` are values `set` in `CMakeLists.txt`. To make sure the source uses the header, just add this to the source `tutorial.cpp`:

	#include "TutorialConfig.h"
    
`${PROJECT_SOURCE_DIR}/TutorialConfig.h.in` will be rendered into `T${PROJECT_BINARY_DIR}/utorialConfig.h` when the project is built/made.


## 2. Adding a Library
Now we need to add a library to our project. This library will contain our own implementation for computing the square root of a number. The executable can use this library instead of defining it itself. Suppose we name the library into a subdirectory `MathFunctions`:

	add_library (MathFunctions mysqrt.cpp)

* `add_library (LibraryName source.cpp ...)` - create a library named `LibraryName` from sources `source.cpp ...`

The corresponding header(s) must be found by CMake. We also need to "add" the path of the library source files into the CMake search path so CMake can find them. Finally, we need to make sure the linker links to this library. The following code does all three steps:
```CMake
include_directories ("${PROJECT_SOURCE_DIR}/MathFunctions"
add_subdirectory (MathFunctions)

add_executable (Tutorial tutorial.cpp)
target_link_libraries (Tutorial MathFunctions)
```
* `add_subdirectory (DirectoryName)` - add source files in `DirectoryName` to search path (use this to add a search path for a library's sources)
* `target_link_libraries (ExecutableName LibraryName)` - link executable `ExecutableName` to library `LibraryName`

---

What if we want to make the `MathFunctions` library optional? The first step to do so is to add an option into `CMakeLists.txt`:

	option (USE_MYMATH "Use tutorial provided math implementation" ON)

* `option(OPTION_NAME "Description" DEFAULT_SWITCH)` - define an option `OPTION_NAME` described by `"Description"` that defaults to `DEFAULT_SWITCH`

Next, we modify our previous `CMakeLists.txt` file:
```CMake
if (USE_MYMATH)
  include_directories("${PROJECT_SOURCE_DIR}/MathFunctions")
  add_subdirectory(MathFunctions)
  set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif (USE_MYMATH)

add_executable (Tutorial tutorial.cpp)
target_link_libraries (Tutorial ${EXTRA_LIBS})
```
Notice above that we can use if expressions with options. Also notice a trick to append `MathFunctions` to our variable `EXTRA_LIBS` to collect up any "optional" libraries that are to be linked in the executable. This trick is used to keep larger projects with many optional components clean.

Just as we can parameterize our `CMakeLists.txt` with the option `USE_MYMATH`, so can we do so with our source `tutorial.cpp`:
```C++
#include <iostream>
#include <cmath>
#include "TutorialConfig.h"

#ifdef USE_MYMATH
#include "MathFunctions.h"
#endif

int main (int argc, char *argv[]) {
	if (argc < 2) {
    	cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << '.'
        	<< Tutorial_VERSION_MINOR << endl;
        cout << "Usage: " << argv[0] << " number\n";
        return 1;
    }
    
    double inputValue = atof(argv[1]);
    
    #ifdef USE_MYMATH
    double outputValue = mysqrt(inputValue);
    #else
    double outputValue = sqrt(inputValue);
    #endif
    
    cout << "The square root of " << inputValue << " is " << outputValue << endl;
	
	return 0;
}
```

To enable this option paramterization for source files, just make sure the following is defined in `TutorialConfig.h.in`.

	#cmakedefine USE_MYMATH


## 3. Installing and Testing
For this next step we will add install rules and testing support to our project. To "install" is to place an executable, library, or some other file into a specified directory from which the installed item is to be used by the end-user. For the `MathFunctions` library, we setup the library and the header file to be installed by adding the following two lines to `MathFunctions`' `CMakeLists.txt` file:

	install (TARGETS MathFunctions DESTINATION bin)
    install (FILES MathFunctions.h DESTINATION include)

* `install (TARGETNAME/FILES PathName1 DESTINATION PathName2)` - setup an install rule that installs `PathName1` to the DESTINATION `PathName2`
  * `TARGETS` specifies an executable/library for `PathName1`
  * `FILES` specifies file paths for `PathName1`

Make sure to add the following lines to the top-level `CMakeLists.txt` file:

	# add the install targets
    install (TARGETS Tutorial DESTINATION bin)
    install (FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h" DESTINATION include)

That is all there is to it. At this point you should be able to build the tutorial, then type `make install` (or build the INSTALL target from an IDE) and it will install the appropriate header files, libraries, and executables. The CMake variable `CMAKE_INSTALL_PREFIX` is used to determine the root of where the files will be installed:

	cmake -DCMAKE_INSTALL_PREFIX=yourpath
    
    # also works
    export DESTDIR=yourpath
    make install

---

Adding tests is also a fairly straight forward process. At the end of the top level `CMakeLists.txt` file we can add a number of basic tests to verify that the application is working correctly.
```CMake
# does the application run?
add_test (TutorialRuns Tutorial 25)

# does it sqrt 25?
add_test (TutorialComp25 Tutorial 25)

set_tests_properties (TutorialComp25
  PROPERTIES PASS_REGULAR_EXPRESSION "25 is 5")

# does it handle negative numbers?
add_test (TutorialNegative Tutorial -25)
set_tests_properties (TutorialNegative
  PROPERTIES PASS_REGULAR_EXPRESSION "-25 is 0")

# does it handle small numbers?
add_test (TutorialSmall Tutorial 0.0001)
set_tests_properties (TutorialSmall
  PROPERTIES PASS_REGULAR_EXPRESSION "0.0001 is 0.01")

# does the usage message work?
add_test (TutorialUsage Tutorial)
set_tests_properties (TutorialUsage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number")
```

* `add_test (TestName app-execution-line)` - add a test that just executes the app
* `set_tests_properties (TestName PROPERTIES PASS_REGULAR_EXPRESSION "regex-expr")` - check if `stdout` output matches the provided regex expression

Adding tests like this is a bit cumbersome. Let's define a macro to make testing more convenient:
```CMake
# define a macro to simplify adding tests
macro (do_test arg result)
  add_test (TutorialComp${arg} Tutorial ${arg})
  set_tests_properties (TutorialComp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result})
endmacro (do_test)

# do a bunch of result based tests
do_test (25 "25 is 5")
do_test (-25 "-25 is 0")
```


## 4. Adding System Introspection
Let's consider adding code whose features depend on the target platform. For example, if the platform has `log`, then we'll use that to compute the square root within the `mysqrt` function. We first test for the availability of these functions using `CheckFunctionExists.cmake` macro in the top level `CMakeLists.txt` file as follows:
```CMake
# does this system provide the log and exp functions?
include (CheckFunctionExists.cmake)
check_function_exists (log HAVE_LOG)
check_function_exists (exp HAVE_EXP)

* `include (CMakeFile)` - inline include another CMake file
* `check_function_exists (FuncName Answer)` - check if `FuncName` is provided by libraries on the system and store the result to `Answer`
```

Next, we modify `TutorialConfig.h.in` to define those values if CMake found them on the platform as follows:

	// does the platform provide exp and log functions?
    #cmakedefine HAVE_LOG
    #cmakedefine HAVE_EXP

`check_function_exists` will "turn on" `HAVE_LOG` or `HAVE_EXP` when they in fact exist. Now modify our library source:

	// if we have both log and exp then use them
    #if defined (HAVE_LOG) && defined (HAVE_EXP)
    result = exp(log(x)*0.5);
    #else  // otherwise use an iterative approach
    ...
    #endif
    

## 5. Adding a Generated File and Generator
We might want to create a table of precomputed square roots as part of the build process, and then compile that table into our application. To accomplish this, we first need a program that will generate the table. In the `MathFunctions` subdirectory, a new source file named `MakeTable.cpp` will do just that:
```C++
// A simple program that builds a sqrt table
#include <iostream>
#include <fstream>
#include <cmath>

using namespace std;

int main (int argc, char *argv[]) {
    // make sure we have enough arguments
    if (argc < 2)
    	return 1;
    
    // open the output file
    ofstream of(argv[1]);
    if (! of.is_open())
    	return 1;
    
    // create a source file with a table of square roots
    of << "double sqrtTable[] = {\n";
    for (int i=0; i<10; ++i) {
    	double result  = sqrt(static_cast<double>(i));
        of << result << ",\n";
    }
	
    of << "0};\n";
    
	return 0;
}
```

Let's add the appropriate commands to `MathFunctions`' `CMakeLists.txt` file to build the MakeTable executable, and then run it as part of the build process.
```CMake
# add the executable that generates the table
add_executable (MakeTable MakeTable.cpp)

# add the command to generate the source code
add_custom_command (
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  DEPENDS MakeTable
)

# add the binary tree directory to the search path for
# include files
include_directories (${CMAKE_CURRENT_BINARY_DIR})

# add the main library
add_library (MathFunctions mysqrt.cpp ${CMAKE_CURRENT_BINARY_DIR}/Table.h)
```

* `add_custom_command (OUTPUT file COMMAND cmd DEPENDS dependency)` - execute `cmd` iff `dependency` is fulfilled; this command is expected to produce `file` as output

---

At this point, our top-level `CMakeLists.txt` file looks like this:
```CMake
cmake_minimum_required (VERSION 2.6)
project (Tutorial)

# The version number
set (Tutorial_VERSION_MAJOR 1)
set (Tutorial_VERSION_MINOR 0)

# Does this system provide log and exp functions?
include (${CMAKE_ROOT}/Modules/CheckFunctionExists.cmake)

check_function_exists (log HAVE_LOG)
check_function_exists (exp HAVE_EXP)

# Should we use our own math functions?
option (USE_MYMATH
  "Use tutorial provided math implementation" ON)

# Configure header file to pass some of CMake settings
# to the source code
configure_file (
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
)

# Add the binary tree to the search path for include files
# so that we'll find TutorialConfig.h
include_directories ("${PROJECT_BINARY_DIR}")

# Add the MathFunctions library?
if (USE_MYMATH)
  include_directories ("${PROJECT_SOURCE_DIR}/MathFunctions")
  add_subdirectory (MathFunctions)
  set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif (USE_MYMATH)

# Add the executable
add_executable (Tutorial tutorial.cpp)
target_link_libraries (Tutorial ${EXTRA_LIBS})

# Add the install targets
install (TARGETS Tutorial DESTINATION bin)
install (FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
		 DESTINATION include)

# Test if the application runs
add_test (TutorialRuns Tutorial 25)

# does the usage message work?
add_test (TutorialUsage Tutorial)
set_tests_properties (TutorialUsage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number")

# define a macro to simplify adding tests
macro (do_test arg result)
  add_test (TutorialComp${arg} Tutorial ${arg})
  set_tests_properties (TutorialComp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result})
endmacro (do_test)

# do a bunch of result based tests
do_test (4 "4 is 2")
do_test (9 "9 is 3")
do_test (5 "5 is 2.236")
do_test (7 "7 is 2.645")
do_test (25 "25 is 5")
do_test (-25 "-25 is 0")
do_test (0.0001 "0.0001 is 0.01")
```
`TutorialConfig.h.in` looks like:
```CMake
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
#cmakedefine USE_MYMATH
#cmakedefine HAVE_LOG
#cmakedefine HAVE_EXP
``` 
`MathFunctions`' `CMakeLists.txt` file looks like:
```CMake
# First we add the executable that generates the table
add_executable(MakeTable MakeTable.cpp)

# Add the command to generate the source code
add_custom_command (
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  DEPENDS MakeTable
  COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
)

# Add the binary tree directory to search path
# for include files
include_directories (${CMAKE_CURRENT_BINARY_DIR})

# add the main library
add_library(MathFunctions mysqrt.cpp ${CMAKE_CURRENT_BINARY_DIR}/Table.h)

install (TARGETS MathFunctions DESTINATION bin)
install (FILES MathFunctions.h DESTINATION include)
```


## 6. Building an Installer
Nowe we want to distribute our project to other people. We want to provide both binary and source distributions on a variety of platforms. This is a little different from the install we did previously in step 3, where we were installing the binaries that we had built from the source code. In this example, we will be building installation packages that support binary installations and package management features as found in cygwin, debian, RPMs, etc. To accomplish this we will use CPack to create platform specific installers. Specifically we'll need to add a few lines to the bottom of our top-level `CMakeLists.txt` file:
```CMake
# build a CPack driven installer package
include (InstallRequiredSystemLibraries)
set (CPACK_RESOURCE_FILE_LICENSE
  "${CMAKE_CURRENT_SOURCE_DIR}/License.txt"
set (CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set (CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
include (CPACK)
```
The next step is to build the project in the usual manner and then run CPack on it.  
To build a binary distribution you would run:

	cpack -C CPackConfig.cmake

To create a source distribution you would type

	cpack -C CPackSourceConfig.cmake

