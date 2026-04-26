# Лабораторная работа №5
Изучение фреймворков тестирования на примере **GTest**
## Tutorial
Пропустим всю уже привычную преамбулу и сразу приступим к командам, непосредственно относящимся к лабе.


Создаем папку для сторонних библиотек и клонируем туда GTest.
```sh
$ mkdir third-party
$ git submodule add https://github.com/google/googletest third-party/gtest
```

<details>
<summary> Вывод команды </summary>

```
Cloning into '/home/univer/Desktop/timp/Mimocake/workspace/projects/lab05/third-party/gtest'...
remote: Enumerating objects: 28627, done.
remote: Counting objects: 100% (61/61), done.
remote: Compressing objects: 100% (46/46), done.
remote: Total 28627 (delta 32), reused 15 (delta 15), pack-reused 28566 (from 2)
Receiving objects: 100% (28627/28627), 13.74 MiB | 983.00 KiB/s, done.
Resolving deltas: 100% (21273/21273), done.
```

</details>

Поскольку репо у GTest имеет огромное количество веток и релизов, выберем именно тот, который мы хотим и переключимся на него командой `checkout`.
```sh
$ cd third-party/gtest && git checkout release-1.8.1 && cd ../..
```

<details>
<summary> Вывод команды </summary>

```
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

</details>
P.s. На моем компьютере код этой версии выдавал ошибку и не компилировался, поэтому я переключился на версию 1.16.0. Также нужно поменять в настойках `CMakeLists.txt` версию языка на C++14.

И коммитим этот шаг
```sh
$ git add .
$ git commit -a -m "added gtest framework"
```

<details>

<summary> Вывод команды </summary>

```
 [main (root-commit) 4734df4] added gtest framework
 7 files changed, 72 insertions(+)
 create mode 100644 .gitmodules
 create mode 100644 lab05/CMakeLists.txt
 create mode 100644 lab05/examples/example1.cpp
 create mode 100644 lab05/examples/example2.cpp
 create mode 100644 lab05/include/print.hpp
 create mode 100644 lab05/sources/print.cpp
 create mode 160000 lab05/third-party/gtest
```

</details>

Теперь вот такой длинной командой модифицируем `CMakeLists.txt`, чтобы там содержалась настройка тестировки.
```sh
$ sed -i '/option(BUILD_EXAMPLES "Build examples" OFF)/a option(BUILD_TESTS "Build tests" OFF)' CMakeLists.txt
```

И еще раз дополним `CMakeLists.txt`, указывая, как собирать скрипты тестировки.
```
$ cat >> CMakeLists.txt <<EOF

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

Создадим папку `tests` и в ней файл `test1.cpp` следующего содержания:
```cpp
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
```
Это и есть наш тестирующий скрипт.

Теперь собираем проект
```sh
$ cmake -H. -B build -D BUILD_TESTS=ON
$ cmake --build build
```

<details>

<summary> Вывод команды </summary>

```
-- The C compiler identification is GNU 11.4.0
-- The CXX compiler identification is GNU 11.4.0
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
-- Looking for pthread.h
-- Looking for pthread.h - found
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD - Success
-- Found Threads: TRUE  
-- Configuring done
-- Generating done
-- Build files have been written to: /home/univer/Desktop/timp/Mimocake/workspace/projects/lab05/build

[  8%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[ 16%] Linking CXX static library libprint.a
[ 16%] Built target print
[ 25%] Building CXX object third-party/gtest/googletest/CMakeFiles/gtest.dir/src/gtest-all.cc.o
[ 33%] Linking CXX static library ../../../lib/libgtest.a
[ 33%] Built target gtest
[ 41%] Building CXX object third-party/gtest/googletest/CMakeFiles/gtest_main.dir/src/gtest_main.cc.o
[ 50%] Linking CXX static library ../../../lib/libgtest_main.a
[ 50%] Built target gtest_main
[ 58%] Building CXX object CMakeFiles/check.dir/tests/test1.cpp.o
[ 66%] Linking CXX executable check
[ 66%] Built target check
[ 75%] Building CXX object third-party/gtest/googlemock/CMakeFiles/gmock.dir/src/gmock-all.cc.o
[ 83%] Linking CXX static library ../../../lib/libgmock.a
[ 83%] Built target gmock
[ 91%] Building CXX object third-party/gtest/googlemock/CMakeFiles/gmock_main.dir/src/gmock_main.cc.o
[100%] Linking CXX static library ../../../lib/libgmock_main.a
[100%] Built target gmock_main

```

</details>


Теперь можно сбилдить конкретно цель `test`
```sh
$ cmake --build build --target test
```
И получим такой вывод:
```
Running tests...
Test project /home/univer/Desktop/timp/Mimocake/workspace/projects/lab05/build
    Start 1: check
1/1 Test #1: check ............................   Passed    0.01 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.01 sec
```

Теперь можно запустить программу:
```sh
$ build/tests
```

```
[==========] Running 1 test from 1 test suite.
[----------] Global test environment set-up.
[----------] 1 test from Print
[ RUN      ] Print.InFileStream
[       OK ] Print.InFileStream (0 ms)
[----------] 1 test from Print (0 ms total)

[----------] Global test environment tear-down
[==========] 1 test from 1 test suite ran. (0 ms total)
[  PASSED  ] 1 test.
```

И запустить ее еще немного другим способом, добавив аргумент `--verbose`
```
Running tests...
UpdateCTestConfiguration  from :/home/univer/Desktop/timp/Mimocake/workspace/projects/lab05/build/DartConfiguration.tcl
UpdateCTestConfiguration  from :/home/univer/Desktop/timp/Mimocake/workspace/projects/lab05/build/DartConfiguration.tcl
Test project /home/univer/Desktop/timp/Mimocake/workspace/projects/lab05/build
Constructing a list of tests
Done constructing a list of tests
Updating test list for fixtures
Added 0 tests to meet fixture requirements
Checking test dependency graph...
Checking test dependency graph end
test 1
    Start 1: check

1: Test command: /home/univer/Desktop/timp/Mimocake/workspace/projects/lab05/build/check
1: Test timeout computed to be: 10000000
1: Running main() from /home/univer/Desktop/timp/Mimocake/workspace/projects/lab05/third-party/gtest/googletest/src/gtest_main.cc
1: [==========] Running 1 test from 1 test suite.
1: [----------] Global test environment set-up.
1: [----------] 1 test from Print
1: [ RUN      ] Print.InFileStream
1: [       OK ] Print.InFileStream (0 ms)
1: [----------] 1 test from Print (0 ms total)
1: 
1: [----------] Global test environment tear-down
1: [==========] 1 test from 1 test suite ran. (0 ms total)
1: [  PASSED  ] 1 test.
1/1 Test #1: check ............................   Passed    0.01 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.01 sec
```