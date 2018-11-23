# Use The Tools Available

An automated framework for executing these tools should be established very early in the development process. It should not take more than 2-3 commands to checkout the source code, build, and execute the tests. Once the tests are done executing, you should have an almost complete picture of the state and quality of the code.

## Source Control

Source control is an absolute necessity for any software development project. If you are not using one yet, start using one.
We currently use:

 * [Git](https://git-scm.com/) - Our standard source control software. We have a gitolite server running, ask Paul or Tobias.
 
## Build Tool

Use an industry standard widely accepted build tool. This prevents you from reinventing the wheel whenever you discover / link to a new library / package your product / etc. 
As far as I know, we currently use no build tool and instead rely on handcrafted configure scripts and make files. When starting a new project, consider using a build tool
to make your life easier.

Examples include:

 * [CMake](http://www.cmake.org/)
   * Consider: https://github.com/sakra/cotire/ for build performance
   * Consider: https://github.com/toeb/cmakepp for enhanced usability
   * Utilize: https://cmake.org/cmake/help/v3.6/command/target_compile_features.html for C++ standard flags
   * Consider: https://github.com/cheshirekow/cmake_format for automatic formatting of your CMakeLists.txt
   * See the [Further Reading](10-Further_Reading.md) section for CMake specific best practices
 * [Waf](https://waf.io/)
 * [FASTBuild](http://www.fastbuild.org/)
 * [Ninja](https://ninja-build.org/) - can greatly improve the incremental build time of your larger projects. Can be used as a target for CMake.
 * [Bazel](http://bazel.io/) - Note: MacOS and Linux only.
 * [gyp](https://chromium.googlesource.com/external/gyp/) - Google's build tool for chromium.
 * [maiken](https://github.com/Dekken/maiken) - Crossplatform build tool with Maven-esque configuration style.
 * [Qt Build Suite](http://doc.qt.io/qbs/) - Crossplatform build tool From Qt.
 * [meson](http://mesonbuild.com/index.html) - Open source build system meant to be both extremely fast, and, even more importantly, as user friendly as possible.
 * [premake](https://premake.github.io/) 

## Continuous Integration

Once you have picked your build tool, set up a continuous integration environment.

Continuous Integration (CI) tools automatically build the source code as changes are pushed to the repository. We have a local Jenkins instance. Again, ask Paul or Tobias.

* [Jenkins CI](https://jenkins-ci.org/)
   * Java Application Server is required
   * supports Windows, OS X, and Linux
   * extendable with a lot of plugins

## Compilers

Use every available and reasonable set of warning options. Some warning options only work with optimizations enabled, or work better the higher the chosen level of optimization is, for example [`-Wnull-dereference`](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html#index-Wnull-dereference-367) with GCC.

You should use as many compilers as you can for your platform(s). Each compiler implements the standard slightly differently and supporting multiple will help ensure the most portable, most reliable code.

### GCC / Clang

`-Wall -Wextra -Wshadow -Wnon-virtual-dtor -pedantic`

 * `-Wall -Wextra` reasonable and standard
 * `-Wshadow` warn the user if a variable declaration shadows one from a parent context
 * `-Wnon-virtual-dtor` warn the user if a class with virtual functions has a non-virtual destructor. This helps catch hard to track down memory errors
 * `-Wold-style-cast` warn for c-style casts
 * `-Wcast-align` warn for potential performance problem casts
 * `-Wunused` warn on anything being unused
 * `-Woverloaded-virtual` warn if you overload (not override) a virtual function
 * `-Wpedantic` warn if non-standard C++ is used
 * `-Wconversion` warn on type conversions that may lose data
 * `-Wsign-conversion` warn on sign conversions
 * `-Wmisleading-indentation` warn if indentation implies blocks where blocks do not exist
 * `-Wduplicated-cond` warn if `if` / `else` chain has duplicated conditions
 * `-Wduplicated-branches` warn if `if` / `else` branches have duplicated code
 * `-Wlogical-op` warn about logical operations being used where bitwise were probably wanted
 * `-Wnull-dereference` warn if a null dereference is detected
 * `-Wuseless-cast` warn if you perform a cast to the same type
 * `-Wdouble-promotion` warn if `float` is implicit promoted to `double`
 * `-Wformat=2` warn on security issues around functions that format output (ie `printf`)
 * `-Wlifetime` (clang only currently) shows object lifetime issues
 
Consider using `-Weverything` and disabling the few warnings you need to on Clang


`-Weffc++` warning mode can be too noisy, but if it works for your project, use it also.

### General

Start with very strict warning settings from the beginning. Trying to raise the warning level after the project is underway can be painful.

Consider using the *treat warnings as errors* setting. `-Werror` with GCC / Clang

## Static Analyzers

The best bet is the static analyzer that you can run as part of your automated build system. Cppcheck and clang meet that requirement for free options.

### Cppcheck
[Cppcheck](http://cppcheck.sourceforge.net/) is free and open source. It strives for 0 false positives and does a good job at it. Therefore all warnings should be enabled: `--enable=all`

Notes:

 * For correct work it requires well formed path for headers, so before usage don't forget to pass: `--check-config`.
 * Finding unused headers does not work with `-j` more than 1. 
 * Remember to add `--force` for code with a lot number of `#ifdef` if you need check all of them.
 
### Clang's Static Analyzer

Clang's analyzer's default options are good for the respective platform. It can be used directly [from CMake](http://garykramlich.blogspot.com/2011/10/using-scan-build-from-clang-with-cmake.html). They can also be called via clang-check and clang-tidy from the [LLVM-based Tools](#llvm-based-tools).

Also, [CodeChecker](https://github.com/Ericsson/CodeChecker) is available as a front-end to clang's static analysis.

`clang-tidy` can be easily used with Visual Studio via the [Clang Power Tools](https://caphyon.github.io/clang-power-tools/) extension.

## Runtime Checkers

### Valgrind

[Valgrind](http://www.valgrind.org/) is a runtime code analyzer that can detect memory leaks, race conditions, and other associated problems. It is supported on various Unix platforms.

### GCC / Clang Sanitizers

These tools provide many of the same features as Valgrind, but built into the compiler. They are easy to use and provide a report of what went wrong.

 * AddressSanitizer
 * MemorySanitizer
 * ThreadSanitizer
 * UndefinedBehaviorSanitizer

Be aware of the sanitizer options available, including runtime options. https://kristerw.blogspot.com/2018/06/useful-gcc-address-sanitizer-checks-not.html

## Ignoring Warnings

If it is determined by team consensus that the compiler or analyzer is warning on something that is either incorrect or unavoidable, the team will disable the specific error to as localized part of the code as possible.

Be sure to reenable the warning after disabling it for a section of code. You do not want your disabled warnings to [leak into other code](http://www.forwardscattering.org/post/48).

## Testing

CMake, mentioned above, has a built in framework for executing tests. Make sure whatever build system you use has a way to execute tests built in.

To further aid in executing tests, consider a library such as [Google Test](https://github.com/google/googletest), [Catch](https://github.com/philsquared/Catch), [CppUTest](https://github.com/cpputest/cpputest) or [Boost.Test](http://www.boost.org/doc/libs/release/libs/test/) to help you organize the tests.

The Master Hearing Aid team currently uses the Google Test framework

### Unit Tests

Unit tests are for small chunks of code, individual functions which can be tested standalone.

### Integration Tests

There should be a test enabled for every feature or bug fix that is committed. See also [Code Coverage Analysis](#code-coverage-analysis). These are tests that are higher level than unit tests. They should still be limited in scope to individual features.

### Negative Testing

Don't forget to make sure that your error handling is being tested and works properly as well. This will become obvious if you aim for 100% code coverage.

## Debugging

### ClangFormat

[ClangFormat](http://clang.llvm.org/docs/ClangFormat.html) can check and correct code formatting to match organizational conventions automatically. [Multipart series](https://engineering.mongodb.com/post/succeeding-with-clangformat-part-1-pitfalls-and-planning/) on utilizing clang-format.
