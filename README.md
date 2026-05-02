# lab_06
##Tutorial
### 1. Подготовка окружения и переход в рабочие папки
```sh
$ export GITHUB_USERNAME=<имя_пользователя>
$ export GITHUB_EMAIL=<адрес_почтового_ящика>
$ alias edit=<nano|vi|vim|subl>
$ alias gsed=sed # for *-nix system
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
```
### 2. Клонирование предыдущей работы
```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab05 projects/lab06
$ cd projects/lab06
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab06
```
### 3. Изменяем CMakeLists.txt, добавляя переменные, задающие версию проекта
```sh
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_STRING "v\${PRINT_VERSION}")
' CMakeLists.txt
$ gsed -i '/project(print)/a\
set(PRINT_VERSION\
  \${PRINT_VERSION_MAJOR}.\${PRINT_VERSION_MINOR}.\${PRINT_VERSION_PATCH}.\${PRINT_VERSION_TWEAK})
' CMakeLists.txt
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_TWEAK 0)
' CMakeLists.txt
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_PATCH 0)
' CMakeLists.txt
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_MINOR 1)
' CMakeLists.txt
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_MAJOR 0)
' CMakeLists.txt
$ git diff
```
*Вывод:*
```sh
diff --git a/CMakeLists.txt b/CMakeLists.txt
index aa7a323..71b64e3 100644
--- a/CMakeLists.txt
+++ b/CMakeLists.txt
@@ -7,6 +7,13 @@ option(BUILD_EXAMPLES "Build examples" OFF)
 option(BUILD_TESTS "Build tests" OFF)
 
 project(print)
+set(PRINT_VERSION_MAJOR 0)
+set(PRINT_VERSION_MINOR 1)
+set(PRINT_VERSION_PATCH 0)
+set(PRINT_VERSION_TWEAK 0)
+set(PRINT_VERSION
+  ${PRINT_VERSION_MAJOR}.${PRINT_VERSION_MINOR}.${PRINT_VERSION_PATCH}.${PRINT_VERSION_TWEAK})
+set(PRINT_VERSION_STRING "v${PRINT_VERSION}")
 
 add_library(print STATIC ${CMAKE_CURRENT_SOURCE_DIR}/sources/print.cpp)
```
### 4.Создаем файлы описания пакетов DESCRIPTION  и ChangeLog.md
```sh
$ touch DESCRIPTION && edit DESCRIPTION
$ touch ChangeLog.md
$ export DATE="`LANG=en_US date +'%a %b %d %Y'`"
$ cat > ChangeLog.md <<EOF
* ${DATE} ${GITHUB_USERNAME} <${GITHUB_EMAIL}> 0.1.0.0
- Initial RPM release
EOF
```
### 5. Создаем и заполняем CPackConfig.cmake (настраиваем CPack) и подключаем его в CMakeLists.txt
*CangeLog.md*
```sh
$ cat > CPackConfig.cmake <<EOF
include(InstallRequiredSystemLibraries)
EOF
$ cat >> CPackConfig.cmake <<EOF
set(CPACK_PACKAGE_CONTACT ${GITHUB_EMAIL})
set(CPACK_PACKAGE_VERSION_MAJOR \${PRINT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR \${PRINT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH \${PRINT_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION_TWEAK \${PRINT_VERSION_TWEAK})
set(CPACK_PACKAGE_VERSION \${PRINT_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_FILE \${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "static C++ library for printing")
EOF
$ cat >> CPackConfig.cmake <<EOF

set(CPACK_RESOURCE_FILE_LICENSE \${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
set(CPACK_RESOURCE_FILE_README \${CMAKE_CURRENT_SOURCE_DIR}/README.md)
EOF
$ cat >> CPackConfig.cmake <<EOF

set(CPACK_RPM_PACKAGE_NAME "print-devel")
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "print")
set(CPACK_RPM_CHANGELOG_FILE \${CMAKE_CURRENT_SOURCE_DIR}/ChangeLog.md)
set(CPACK_RPM_PACKAGE_RELEASE 1)
EOF
$ cat >> CPackConfig.cmake <<EOF

set(CPACK_DEBIAN_PACKAGE_NAME "libprint-dev")
set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)
EOF
$ cat >> CPackConfig.cmake <<EOF

include(CPack)
EOF
$ cat >> CMakeLists.txt <<EOF

include(CPackConfig.cmake)
EOF
```
### 6.  Редактируем названия лабы в REALME.md
```sh
$ gsed -i 's/lab05/lab06/g' README.md
```
### 7. По классике коммитим и пушим вся на репозиторий 
```sh
$ git add .
$ git commit -m"added cpack config"
$ git tag v0.1.0.0
$ git push origin master --tags
```
### 8. Билдим проект
```sh
$ cmake -H. -B_build
$ cmake --build _build
$ cd _build
$ cpack -G "TGZ"
$ cd ..
$ cmake -H. -B_build -DCPACK_GENERATOR="TGZ"
$ cmake --build _build --target package
```
*Вывод:*
```sh
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_06_tutorial$ cmake -H. -B_build
CMake Deprecation Warning at CMakeLists.txt:1 (cmake_minimum_required):
  Compatibility with CMake < 3.5 will be removed from a future version of
  CMake.

  Update the VERSION argument <min> value or use a ...<max> suffix to tell
  CMake that the project does not need compatibility with older versions.


-- Configuring done (0.0s)
-- Generating done (0.0s)
-- Build files have been written to: /home/vboxuser/satodim/workspace/workspace/projects/lab_06_tutorial/_build
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_06_tutorial$ cmake --build _build
[ 50%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[100%] Linking CXX static library libprint.a
[100%] Built target print
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_06_tutorial$ cd _build
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_06_tutorial/_build$ cpack -G "TGZ"
CPack: Create package using TGZ
CPack: Install projects
CPack: - Run preinstall target for: print
CPack: - Install project: print []
CPack: Create package
CPack: - package: /home/vboxuser/satodim/workspace/workspace/projects/lab_06_tutorial/_build/print-0.1.0.0-Linux.tar.gz generated.
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_06_tutorial/_build$ cd ..
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_06_tutorial$ cmake -H. -B_build -DCPACK_GENERATOR="TGZ"
CMake Deprecation Warning at CMakeLists.txt:1 (cmake_minimum_required):
  Compatibility with CMake < 3.5 will be removed from a future version of
  CMake.

  Update the VERSION argument <min> value or use a ...<max> suffix to tell
  CMake that the project does not need compatibility with older versions.


-- Configuring done (0.0s)
-- Generating done (0.0s)
-- Build files have been written to: /home/vboxuser/satodim/workspace/workspace/projects/lab_06_tutorial/_build
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_06_tutorial$ cmake --build _build --target package
[100%] Built target print
Run CPack packaging tool...
CPack: Create package using TGZ
CPack: Install projects
CPack: - Run preinstall target for: print
CPack: - Install project: print []
CPack: Create package
CPack: - package: /home/vboxuser/satodim/workspace/workspace/projects/lab_06_tutorial/_build/print-0.1.0.0-Linux.tar.gz generated.

```
### 8. Сохранение артефакта
```sh
$ mkdir artifacts
$ mv _build/*.tar.gz artifacts
$ tree artifacts
```
*Вывод:*
```sh
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_06_tutorial$ mkdir artifacts
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_06_tutorial$ mv _build/*.tar.gz artifacts
vboxuser@Linuxoid:~/satodim/workspace/workspace/projects/lab_06_tutorial$ tree artifacts
artifacts
└── print-0.1.0.0-Linux.tar.gz

1 directory, 1 file

```
