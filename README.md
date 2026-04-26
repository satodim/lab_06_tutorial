# lab_05
##Tutorial
### 1. Подготовка окружения и переход в рабочие папки
```sh
export GITHUB_USERNAME=<имя_пользователя>
alias gsed=sed
cd ${GITHUB_USERNAME}/workspace
pushd .
source scripts/activate
```
### 2. Клонирование предыдущей работы
```sh
git clone https://github.com/${GITHUB_USERNAME}/lab04 projects/lab05
cd projects/lab05
git remote remove origin
git remote add origin https://github.com/${GITHUB_USERNAME}/lab05
```
### 3. Добавление Google Test   
```sh
mkdir third-party
git submodule add https://github.com/google/googletest third-party/gtest
cd third-party/gtest && git checkout release-1.8.1 && cd ../..
```
*Вывод:*
```sh
Cloning into '/home/vboxuser/satodim/workspace/workspace/projects/lab_05_tutorial/third-party/gtest'...
remote: Enumerating objects: 28627, done.
remote: Counting objects: 100% (61/61), done.
remote: Compressing objects: 100% (46/46), done.
remote: Total 28627 (delta 32), reused 15 (delta 15), pack-reused 28566 (from 2)
Receiving objects: 100% (28627/28627), 13.74 MiB | 1.92 MiB/s, done.
Resolving deltas: 100% (21273/21273), done.
```
*Вывод2:*
```sh
Note: switching to 'release-1.8.1'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 2fe3bd99 Merge pull request #1433 from dsacre/fix-clang-warnings
```
### 4.Настройка Cmake для тестов
```sh
sed -i '/option(BUILD_EXAMPLES...)/a\option(BUILD_TESTS "Build tests" OFF)' CMakeLists.txt
cat >> CMakeLists.txt <<EOF
if(BUILD_TESTS)
  enable_testing()
  add_subdirectory(third-party/gtest)
  file(GLOB \${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
  add_executable(check \${\${PROJECT_NAME}_TEST_SOURCES})
  target_link_libraries(check \${PROJECT_NAME} gtest_main)
  add_test(NAME check COMMAND check)
endif()
EOF
```
### 5. Создание теста
```sh
tests/test1.cpp <<EOF
#include <print.hpp>

#include <gtest/gtest.h>

TEST(Print, InFileStream)
{
  std::string filepath = "file.txt";
  std::string text = "hello";
  std::ofstream out{filepath};

  print(text, out);
  out.close();

  std::string result;
  std::ifstream in{filepath};
  in >> result;

  EXPECT_EQ(result, text);
}
EOF
```
### 6.  Сборка проекта с тестами
```sh
cmake -H. -B_build -DBUILD_TESTS=ON
cmake --build _build
```
Получил ошибку из-за старой версии
*Фикс:*
```sh
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_05_tutorial$ rm -rf _build
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_05_tutorial$ cd third-party/gtest
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_05_tutorial/third-party/gtest$ git checkout release-1.8.1
HEAD is now at 2fe3bd99 Merge pull request #1433 from dsacre/fix-clang-warnings
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_05_tutorial/third-party/gtest$ cd ../..
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_05_tutorial$ cmake -H. -B_build \
  -DBUILD_TESTS=ON \
  -DCMAKE_CXX_FLAGS="-Wno-error -Wno-maybe-uninitialized"
```
*Итоговы вывод:*
```sh
CMake Deprecation Warning at CMakeLists.txt:1 (cmake_minimum_required):
  Compatibility with CMake < 3.5 will be removed from a future version of
  CMake.

  Update the VERSION argument <min> value or use a ...<max> suffix to tell
  CMake that the project does not need compatibility with older versions.


-- The C compiler identification is GNU 13.3.0
-- The CXX compiler identification is GNU 13.3.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
CMake Deprecation Warning at third-party/gtest/CMakeLists.txt:1 (cmake_minimum_required):
  Compatibility with CMake < 3.5 will be removed from a future version of
  CMake.

  Update the VERSION argument <min> value or use a ...<max> suffix to tell
  CMake that the project does not need compatibility with older versions.


CMake Deprecation Warning at third-party/gtest/googlemock/CMakeLists.txt:42 (cmake_minimum_required):
  Compatibility with CMake < 3.5 will be removed from a future version of
  CMake.

  Update the VERSION argument <min> value or use a ...<max> suffix to tell
  CMake that the project does not need compatibility with older versions.


CMake Deprecation Warning at third-party/gtest/googletest/CMakeLists.txt:49 (cmake_minimum_required):
  Compatibility with CMake < 3.5 will be removed from a future version of
  CMake.

  Update the VERSION argument <min> value or use a ...<max> suffix to tell
  CMake that the project does not need compatibility with older versions.


CMake Warning (dev) at third-party/gtest/googletest/cmake/internal_utils.cmake:239 (find_package):
  Policy CMP0148 is not set: The FindPythonInterp and FindPythonLibs modules
  are removed.  Run "cmake --help-policy CMP0148" for policy details.  Use
  the cmake_policy command to set the policy and suppress this warning.

Call Stack (most recent call first):
  third-party/gtest/googletest/CMakeLists.txt:84 (include)
This warning is for project developers.  Use -Wno-dev to suppress it.

-- Found PythonInterp: /usr/bin/python3 (found version "3.12.3") 
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD - Success
-- Found Threads: TRUE  
-- Configuring done (0.5s)
-- Generating done (0.0s)
-- Build files have been written to: /home/vboxuser/satodim/workspace/workspace/projects/lab_05_tutorial/_build
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_05_tutorial$ cmake --build _build
[  8%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[ 16%] Linking CXX static library libprint.a
[ 16%] Built target print
[ 25%] Building CXX object third-party/gtest/googlemock/gtest/CMakeFiles/gtest.dir/src/gtest-all.cc.o
[ 33%] Linking CXX static library libgtest.a
[ 33%] Built target gtest
[ 41%] Building CXX object third-party/gtest/googlemock/gtest/CMakeFiles/gtest_main.dir/src/gtest_main.cc.o
[ 50%] Linking CXX static library libgtest_main.a
[ 50%] Built target gtest_main
[ 58%] Building CXX object CMakeFiles/check.dir/tests/test1.cpp.o
[ 66%] Linking CXX executable check
[ 66%] Built target check
[ 75%] Building CXX object third-party/gtest/googlemock/CMakeFiles/gmock.dir/src/gmock-all.cc.o
[ 83%] Linking CXX static library libgmock.a
[ 83%] Built target gmock
[ 91%] Building CXX object third-party/gtest/googlemock/CMakeFiles/gmock_main.dir/src/gmock_main.cc.o
[100%] Linking CXX static library libgmock_main.a
[100%] Built target gmock_main
```
### 7. Запуск тестов
```sh
cmake --build _build --target test
_build/check
cmake --build _build --target test -- ARGS=--verbose
```
*ВЫводы:*
```sh
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_05_tutorial$ cmake --build _build --target test
Running tests...
Test project /home/vboxuser/satodim/workspace/workspace/projects/lab_05_tutorial/_build
    Start 1: check
1/1 Test #1: check ............................   Passed    0.00 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.01 sec
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_05_tutorial$ _build/check
Running main() from /home/vboxuser/satodim/workspace/workspace/projects/lab_05_tutorial/third-party/gtest/googletest/src/gtest_main.cc
[==========] Running 1 test from 1 test case.
[----------] Global test environment set-up.
[----------] 1 test from Print
[ RUN      ] Print.InFileStream
[       OK ] Print.InFileStream (0 ms)
[----------] 1 test from Print (1 ms total)

[----------] Global test environment tear-down
[==========] 1 test from 1 test case ran. (1 ms total)
[  PASSED  ] 1 test.
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_05_tutorial$ cmake --build _build --target test -- ARGS=--verbose
Running tests...
UpdateCTestConfiguration  from :/home/vboxuser/satodim/workspace/workspace/projects/lab_05_tutorial/_build/DartConfiguration.tcl
UpdateCTestConfiguration  from :/home/vboxuser/satodim/workspace/workspace/projects/lab_05_tutorial/_build/DartConfiguration.tcl
Test project /home/vboxuser/satodim/workspace/workspace/projects/lab_05_tutorial/_build
Constructing a list of tests
Done constructing a list of tests
Updating test list for fixtures
Added 0 tests to meet fixture requirements
Checking test dependency graph...
Checking test dependency graph end
test 1
    Start 1: check

1: Test command: /home/vboxuser/satodim/workspace/workspace/projects/lab_05_tutorial/_build/check
1: Working Directory: /home/vboxuser/satodim/workspace/workspace/projects/lab_05_tutorial/_build
1: Test timeout computed to be: 10000000
1: Running main() from /home/vboxuser/satodim/workspace/workspace/projects/lab_05_tutorial/third-party/gtest/googletest/src/gtest_main.cc
1: [==========] Running 1 test from 1 test case.
1: [----------] Global test environment set-up.
1: [----------] 1 test from Print
1: [ RUN      ] Print.InFileStream
1: [       OK ] Print.InFileStream (0 ms)
1: [----------] 1 test from Print (0 ms total)
1: 
1: [----------] Global test environment tear-down
1: [==========] 1 test from 1 test case ran. (1 ms total)
1: [  PASSED  ] 1 test.
1/1 Test #1: check ............................   Passed    0.01 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.01 sec
```
### 8. Работа с Github Actions, потому что мы не работаем с Travis
```sh
mkdir -p .github/workflows
nano .github/workflows/ci.yml
```
ci.yml
```sh
name: CMake Build and Test

on:
  push:
    branches: [ "master", "main" ]
  pull_request:
    branches: [ "master", "main" ]

env:
  BUILD_TYPE: Release

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4
      with:
        submodules: 'recursive'

    - name: Configure CMake
      run: cmake -B ${{github.workspace}}/_build -DCMAKE_BUILD_TYPE=${{env.BUILD_TYPE}} -DBUILD_TESTS=ON

    - name: Build
      run: cmake --build ${{github.workspace}}/_build --config ${{env.BUILD_TYPE}}

    - name: Test
      working-directory: ${{github.workspace}}/_build
      run: ctest -C ${{env.BUILD_TYPE}} --output-on-failure
```
### 9. Осталось только  закоммитить изменения и  запушить все на удаленный репозиторий
```sh
git add .github CMakeLists.txt file.txt tests/
git commit -m "added tests and workflows"
```
*Вывод:*
```sh
mmit -m "added tests and workflows"
[main 1a12423] added tests and workflows
 4 files changed, 95 insertions(+)
 create mode 100644 .github/workflows/ci.yml
 create mode 100644 CMakeLists.txt
 create mode 100644 file.txt
 create mode 100644 tests/test1.cpp
```
```sh
git push origin main
```
*Вывод:*
```sh
Enumerating objects: 50, done.
Counting objects: 100% (50/50), done.
Delta compression using up to 2 threads
Compressing objects: 100% (30/30), done.
Writing objects: 100% (50/50), 13.65 KiB | 6.83 MiB/s, done.
Total 50 (delta 12), reused 36 (delta 10), pack-reused 0
remote: Resolving deltas: 100% (12/12), done.
To https://github.com/satodim/lab_05_tutorial
 * [new branch]      main -> main
```
