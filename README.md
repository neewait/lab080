## Лабораторная работа 8

## Ход выполнения

Клонировал репозиторий от ЛР (лабораторной работы)  №6

```console
$ git clone https://github.com/3Artem99/lab06 lab08
```

```console
Клонирование в «lab08»…
remote: Enumerating objects: 162, done.
remote: Counting objects: 100% (162/162), done.
remote: Compressing objects: 100% (94/94), done.
remote: Total 162 (delta 75), reused 128 (delta 58), pack-reused 0
Получение объектов: 100% (162/162), 1.04 МиБ | 3.86 МиБ/с, готово.
Определение изменений: 100% (75/75), готово.
```

Переключился на репозиторий для ЛР №8
```console
$ git remote remove origin
$ git remote add origin https://github.com/3Artem99/lab08
```

Из туториала для [ЛР №7](https://github.com/tp-labs/lab07) создал папку демо и в ней создал файл "main.cpp"

```console
$ mkdir demo
$ cat >demo/main.cpp<<EOF
>EOF
```

main.cpp:

```console
#include <print.hpp>

#include <cstdlib>

int main(int argc, char* argv[])
{
  const char* log_path = std::getenv("LOG_PATH");
  if (log_path == nullptr)
  {
    std::cerr << "undefined environment variable: LOG_PATH" << std::endl;
    return 1;
  }
  std::string text;
  while (std::cin >> text)
  {
    std::ofstream out{log_path, std::ios_base::app};
    print(text, out);
    out << std::endl;
  }
}
```

Изменил файл "CMakeLists.txt":

```console
cmake_minimum_required(VERSION 3.4)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_EXAMPLES "Build examples" OFF)
option(BUILD_TESTS "Build tests" OFF)

project(print)
set(PRINT_VERSION_MAJOR 0)
set(PRINT_VERSION_MINOR 1)
set(PRINT_VERSION_PATCH 0)
set(PRINT_VERSION_TWEAK 0)
set(PRINT_VERSION
  ${PRINT_VERSION_MAJOR}.${PRINT_VERSION_MINOR}.${PRINT_VERSION_PATCH}.${PRINT_VERSION_TWEAK})
set(PRINT_VERSION_STRING "v${PRINT_VERSION}")

add_library(print STATIC ${CMAKE_CURRENT_SOURCE_DIR}/sources/print.cpp)

target_include_directories(print PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

if(BUILD_EXAMPLES)
  file(GLOB EXAMPLE_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/examples/*.cpp")
  foreach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
    get_filename_component(EXAMPLE_NAME ${EXAMPLE_SOURCE} NAME_WE)
    add_executable(${EXAMPLE_NAME} ${EXAMPLE_SOURCE})
    target_link_libraries(${EXAMPLE_NAME} print)
    install(TARGETS ${EXAMPLE_NAME}
      RUNTIME DESTINATION bin
    )
  endforeach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
endif()

install(TARGETS print
    EXPORT print-config
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include)
install(EXPORT print-config DESTINATION cmake)

if(BUILD_TESTS)
    enable_testing()
    file(GLOB ${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
    add_executable(check ${${PROJECT_NAME}_TEST_SOURCES})
    target_link_libraries(check ${PROJECT_NAME} GTest::main)
    add_test(NAME check COMMAND check)
endif()

add_executable(demo demo/main.cpp)
target_link_libraries(demo print)
install(TARGETS demo RUNTIME DESTINATION bin)

include(CPackConfig.cmake)
```

Собираю проект:

```console
$ cmake -B build
-- The C compiler identification is GNU 11.3.0
-- The CXX compiler identification is GNU 11.3.0
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
-- Configuring done
-- Generating done
-- Build files have been written to: /home/artem99/3Artem99/workspace/projects/lab08/build
```

```console
$ cmake --build build
[ 25%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[ 50%] Linking CXX static library libprint.a
[ 50%] Built target print
[ 75%] Building CXX object CMakeFiles/demo.dir/demo/main.cpp.o
[100%] Linking CXX executable demo
[100%] Built target demo
```

Создаю файл "Dockerfile":

```console
$ cat >Dockerfile<<EOF
> EOF
```

Файл "Dockerfile":

```console
FROM ubuntu:18.04

RUN apt update
RUN apt install -yy gcc g++ cmake

COPY . print/
WORKDIR print

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build
RUN cmake --build _build --target install

ENV LOG_PATH /home/logs/log.txt

VOLUME /home/logs

#WORKDIR _install/bin
WORKDIR /print/_install/bin

ENTRYPOINT ./demo
```
___

```console
$ docker build -t logger .
```
<details><summary>ResultOfCommand</summary>
  <p>
  
```sh
Sending build context to Docker daemon  2.605MB
Step 1/12 : FROM ubuntu:18.04
 ---> f9a80a55f492
Step 2/12 : RUN apt update
 ---> Using cache
 ---> f4d04fe395d1
Step 3/12 : RUN apt install -yy gcc g++ cmake
 ---> Using cache
 ---> d0fcbdcfc622
Step 4/12 : COPY . print/
 ---> 431b88854b07
Step 5/12 : WORKDIR print
 ---> Running in 27f89a66f3b9
Removing intermediate container 27f89a66f3b9
 ---> 564a43cf1dcb
Step 6/12 : RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
 ---> Running in 52cb463a0403
-- The C compiler identification is GNU 7.5.0
-- The CXX compiler identification is GNU 7.5.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /print/_build
Removing intermediate container 52cb463a0403
 ---> 13e3126d4203
Step 7/12 : RUN cmake --build _build
 ---> Running in b61c05a5d86a
Scanning dependencies of target print
[ 25%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[ 50%] Linking CXX static library libprint.a
[ 50%] Built target print
Scanning dependencies of target demo
[ 75%] Building CXX object CMakeFiles/demo.dir/demo/main.cpp.o
[100%] Linking CXX executable demo
[100%] Built target demo
Removing intermediate container b61c05a5d86a
 ---> 9f16d2a5305e
Step 8/12 : RUN cmake --build _build --target install
 ---> Running in 64d6372eecb4
[ 50%] Built target print
[100%] Built target demo
Install the project...
-- Install configuration: "Release"
-- Installing: /print/_install/lib/libprint.a
-- Installing: /print/_install/include
-- Installing: /print/_install/include/print.hpp
-- Installing: /print/_install/cmake/print-config.cmake
-- Installing: /print/_install/cmake/print-config-release.cmake
-- Installing: /print/_install/bin/demo
Removing intermediate container 64d6372eecb4
 ---> cde9361e9388
Step 9/12 : ENV LOG_PATH /home/logs/log.txt
 ---> Running in 6244631450a5
Removing intermediate container 6244631450a5
 ---> 2603343849a5
Step 10/12 : VOLUME /home/logs
 ---> Running in 2c213dd9c268
Removing intermediate container 2c213dd9c268
 ---> fff920dea68a
Step 11/12 : WORKDIR /print/_install/bin
 ---> Running in 63e8085d389d
Removing intermediate container 63e8085d389d
 ---> 087e75cbb8d5
Step 12/12 : ENTRYPOINT ./demo
 ---> Running in 1959df2a4d06
Removing intermediate container 1959df2a4d06
 ---> 048d5e329e63
Successfully built 048d5e329e63
Successfully tagged logger:latest
```

</p>
</details>
  
```console
$ sudo docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
logger       latest    048d5e329e63   40 seconds ago   336MB
<none>       <none>    f7cd3e658164   8 minutes ago    336MB
<none>       <none>    db0cd6a126cc   10 minutes ago   336MB
<none>       <none>    9d5ad30f6ad5   12 minutes ago   336MB
ubuntu       18.04     f9a80a55f492   5 days ago       63.2MB
```

Создал папку "logs":

```console
$ mkdir logs
```

```console
$ sudo docker run -it -v "$(pwd)/logs/:/home/logs/" logger
Hello its first text      
Its second text
third
party
```

```console
$ sudo docker inspect logger
```

<details><summary>Output</summary>
<p>
  
```sh
[
    {
        "Id": "sha256:048d5e329e63d5563b768f8981797cac6c05956981ea58dea36c90a5989c8ad2",
        "RepoTags": [
            "logger:latest"
        ],
        "RepoDigests": [],
        "Parent": "sha256:087e75cbb8d5eb5ca1c2c4052cf31c2ba54dd6734d4cdcddd88f2ba81ee89f42",
        "Comment": "",
        "Created": "2023-06-04T11:57:49.036038713Z",
        "Container": "1959df2a4d0659771e95ffcf0e3af9ae240b2793a12f15a3dcb15e8aa56cda59",
        "ContainerConfig": {
            "Hostname": "1959df2a4d06",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LOG_PATH=/home/logs/log.txt"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "ENTRYPOINT [\"/bin/sh\" \"-c\" \"./demo\"]"
            ],
            "Image": "sha256:087e75cbb8d5eb5ca1c2c4052cf31c2ba54dd6734d4cdcddd88f2ba81ee89f42",
            "Volumes": {
                "/home/logs": {}
            },
            "WorkingDir": "/print/_install/bin",
            "Entrypoint": [
                "/bin/sh",
                "-c",
                "./demo"
            ],
            "OnBuild": null,
            "Labels": {
                "org.opencontainers.image.ref.name": "ubuntu",
                "org.opencontainers.image.version": "18.04"
            }
        },
        "DockerVersion": "20.10.24",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LOG_PATH=/home/logs/log.txt"
            ],
            "Cmd": null,
            "Image": "sha256:087e75cbb8d5eb5ca1c2c4052cf31c2ba54dd6734d4cdcddd88f2ba81ee89f42",
            "Volumes": {
                "/home/logs": {}
            },
            "WorkingDir": "/print/_install/bin",
            "Entrypoint": [
                "/bin/sh",
                "-c",
                "./demo"
            ],
            "OnBuild": null,
            "Labels": {
                "org.opencontainers.image.ref.name": "ubuntu",
                "org.opencontainers.image.version": "18.04"
            }
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 336187061,
        "VirtualSize": 336187061,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/snap/docker/common/var-lib-docker/overlay2/b8b04f65ebf3e5799d1534a1007e9058432929581dc766f58f80b097bdd71f59/diff:/var/snap/docker/common/var-lib-docker/overlay2/3c1228ce2745514a3b4401937f2d40da9557eb66b97776c5bfcbfa53bbcab1ec/diff:/var/snap/docker/common/var-lib-docker/overlay2/f8fb92353439481abef1ccce51c7d96be0b24f34e269964248e213695523ac18/diff:/var/snap/docker/common/var-lib-docker/overlay2/e2b7b1b24906de63d458297a1b58da38ebf6589c525894f1d2d3d6889a29aecd/diff:/var/snap/docker/common/var-lib-docker/overlay2/b6280d885ed5f71808026b434eb64162344d8850ce422c2355f8907ec650e3dc/diff:/var/snap/docker/common/var-lib-docker/overlay2/a42cb2d1f9acb2ccf36a9cd6742e7ffce89c3428dc582dffd76509bed3711634/diff",
                "MergedDir": "/var/snap/docker/common/var-lib-docker/overlay2/6f8e7b757e95b10bdd48b88f18adc0bb11fb4ed5b258d0f6bec7c2acfb6e294d/merged",
                "UpperDir": "/var/snap/docker/common/var-lib-docker/overlay2/6f8e7b757e95b10bdd48b88f18adc0bb11fb4ed5b258d0f6bec7c2acfb6e294d/diff",
                "WorkDir": "/var/snap/docker/common/var-lib-docker/overlay2/6f8e7b757e95b10bdd48b88f18adc0bb11fb4ed5b258d0f6bec7c2acfb6e294d/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:548a79621a426b4eb077c926eabac5a8620c454fb230640253e1b44dc7dd7562",
                "sha256:eba373e842b1502ca369a642e03e93b1b8c700fddd64f794cf02d97f802774fc",
                "sha256:d72e6008887c516fd472381f450c0bc86bf27fbf2dbb92c74d8694d5fbf8ddf7",
                "sha256:a14a3db0de2d403121a2c93c31f6cb46fb4d1a61f9b9f623da91c9d984aeef83",
                "sha256:3dc1c9b6c969f7cee01f9e524c6b9d384f41785d7adeb37b95c063e5003a8f81",
                "sha256:c2252ddcdd803816adc8d1fa06cd7f7a156ba6149c5f2876f5586524b49681d1",
                "sha256:b84a988746def71201e3197494aa167d2b3b5325249c658dd61ba6b38885b774"
            ]
        },
        "Metadata": {
            "LastTagTime": "2023-06-04T14:57:49.169855054+03:00"
        }
    }
]

```
</p>
</details>

```console
$ cat logs/log.txt
Hello
its
first
text
Its
second
text
third
party
```

Пушу всё в репозиторий lab08
```console
$ git add .
$ git commit -m "Lab08"
<details>

<summary>Detail</summary>
[main d53c5a6] Lab08
 56 files changed, 4749 insertions(+), 173 deletions(-)
 rewrite CMakeLists.txt (77%)
 create mode 100644 Dockerfile
 rewrite README.md (88%)
 create mode 100644 build/CMakeCache.txt
 create mode 100644 build/CMakeFiles/3.22.1/CMakeCCompiler.cmake
 create mode 100644 build/CMakeFiles/3.22.1/CMakeCXXCompiler.cmake
 create mode 100755 build/CMakeFiles/3.22.1/CMakeDetermineCompilerABI_C.bin
 create mode 100755 build/CMakeFiles/3.22.1/CMakeDetermineCompilerABI_CXX.bin
 create mode 100644 build/CMakeFiles/3.22.1/CMakeSystem.cmake
 create mode 100644 build/CMakeFiles/3.22.1/CompilerIdC/CMakeCCompilerId.c
 create mode 100755 build/CMakeFiles/3.22.1/CompilerIdC/a.out
 create mode 100644 build/CMakeFiles/3.22.1/CompilerIdCXX/CMakeCXXCompilerId.cpp
 create mode 100755 build/CMakeFiles/3.22.1/CompilerIdCXX/a.out
 create mode 100644 build/CMakeFiles/CMakeDirectoryInformation.cmake
 create mode 100644 build/CMakeFiles/CMakeOutput.log
 create mode 100644 build/CMakeFiles/Export/cmake/print-config-noconfig.cmake
 create mode 100644 build/CMakeFiles/Export/cmake/print-config.cmake
 create mode 100644 build/CMakeFiles/Makefile.cmake
 create mode 100644 build/CMakeFiles/Makefile2
 create mode 100644 build/CMakeFiles/TargetDirectories.txt
 create mode 100644 build/CMakeFiles/cmake.check_cache
 create mode 100644 build/CMakeFiles/demo.dir/DependInfo.cmake
 create mode 100644 build/CMakeFiles/demo.dir/build.make
 create mode 100644 build/CMakeFiles/demo.dir/cmake_clean.cmake
 create mode 100644 build/CMakeFiles/demo.dir/compiler_depend.make
 create mode 100644 build/CMakeFiles/demo.dir/compiler_depend.ts
 create mode 100644 build/CMakeFiles/demo.dir/demo/main.cpp.o
 create mode 100644 build/CMakeFiles/demo.dir/demo/main.cpp.o.d
 create mode 100644 build/CMakeFiles/demo.dir/depend.make
 create mode 100644 build/CMakeFiles/demo.dir/flags.make
 create mode 100644 build/CMakeFiles/demo.dir/link.txt
 create mode 100644 build/CMakeFiles/demo.dir/progress.make
 create mode 100644 build/CMakeFiles/print.dir/DependInfo.cmake
 create mode 100644 build/CMakeFiles/print.dir/build.make
 create mode 100644 build/CMakeFiles/print.dir/cmake_clean.cmake
 create mode 100644 build/CMakeFiles/print.dir/cmake_clean_target.cmake
 create mode 100644 build/CMakeFiles/print.dir/compiler_depend.make
 create mode 100644 build/CMakeFiles/print.dir/compiler_depend.ts
 create mode 100644 build/CMakeFiles/print.dir/depend.make
 create mode 100644 build/CMakeFiles/print.dir/flags.make
 create mode 100644 build/CMakeFiles/print.dir/link.txt
 create mode 100644 build/CMakeFiles/print.dir/progress.make
 create mode 100644 build/CMakeFiles/print.dir/sources/print.cpp.o
 create mode 100644 build/CMakeFiles/print.dir/sources/print.cpp.o.d
 create mode 100644 build/CMakeFiles/progress.marks
 create mode 100644 build/CPackConfig.cmake
 create mode 100644 build/CPackSourceConfig.cmake
 create mode 100644 build/Makefile
 create mode 100644 build/cmake_install.cmake
 create mode 100755 build/demo
 create mode 100644 build/libprint.a
 create mode 100644 demo/main.cpp
 create mode 100644 include/print.hpp
 create mode 100644 logs/log.txt
 create mode 100644 sources/activate
 create mode 100644 sources/print.cpp
 </details>
```
```console
$ git push origin main
Username for 'https://github.com': 3Artem99
Password for 'https://3Artem99@github.com': 
Перечисление объектов: 75, готово.
Подсчет объектов: 100% (75/75), готово.
При сжатии изменений используется до 4 потоков
Сжатие объектов: 100% (63/63), готово.
Запись объектов: 100% (72/72), 50.15 КиБ | 3.58 МиБ/с, готово.
Всего 72 (изменений 11), повторно использовано 0 (изменений 0), повторно использовано пакетов 0
remote: Resolving deltas: 100% (11/11), completed with 1 local object.
To https://github.com/3Artem99/lab06
   08db504..d53c5a6  main -> main
```
