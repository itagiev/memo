### cmake_minimum_required()

Все файлы CMakeLists.txt в проекте должны начинаться с определния минимальной версии cmake используя команду `cmake_minimum_required()`.

```cmake
cmake_minimum_required(VERSION 3.10)
```

### project()

Чтобы начать проект, используется команда `project()`, чтобы задать имя проекта. Это команда обязательна и должна быть вызвана сразу после
`cmake_minimum_required()`. Эта команда также может быть использована, чтобы задать некоторую информацию о проекте, такую как
язык или версию.

```cmake
# set the project name and version
project(Tutorial VERSION 1.0)
```

### add_executable()

Команда `add_executable()` создает исполняемый файл с использованием определенных исходных файлов.

```cmake
# add the executable
add_executable(Tutorial tutorial.cxx)
```

### set()

Используя команду `set()` можно задать переменным окружения значения.

```cmake
# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

### configure_file()

Преобразовывает входной файл в выходной с подстановкой некоторых переменных во входном файле.

```cmake
# configure a header file to pass some of the CMake settings
# to the source code
configure_file(TutorialConfig.h.in TutorialConfig.h)
```

TutorialConfig.h.in

```h
// the configured options and settings for Tutorial
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

### target_include_directories()

Показать исполняемому файлу, где нужно искать include файлы.

```cmake
# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
    "${PROJECT_BINARY_DIR}")
```

### add_library()

Чтобы добавить библиотеку "Классов" в CMake, используй команду `add_library()` и укажи какие исходники должны быть включены в библиотеку.

```cmake
add_library(MathFunctions MathFunctions.cxx)
```

### add_subdirectory()

Чтобы не помещать все файлы в один проект, можно создать дополнительные директории. Для этого можно использовать команду `add_subdirectory()`.

```cmake
# add the MathFunctions library
add_subdirectory(MathFunctions)
```

### target_include_directories()

Созданную библиотеку нужно подключить в проект 

```cmake
# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
    "${PROJECT_BINARY_DIR}"
    "${PROJECT_SOURCE_DIR}/MathFunctions")
```

### target_link_libraries()

Также созданную библиотеку нужно подключить в проект к исполняемому файлу

```cmake
target_link_libraries(Tutorial PUBLIC MathFunctions)
```

### Опции

`option()`, `if()`

```cmake
# Создание переменной
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# Использование условных операторов
if(USE_MYMATH)

    target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")

    add_library(SqrtLibrary STATIC mysqrt.cxx)

    target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()
```

### target_compile_definitions()

Чтобы добавить переменную к проекту/библиотеке можно использовать команду `target_compile_definitions()`.

```cmake
# USE_MYMATH as a precompiled definition to our source files
if(USE_MYMATH)
    target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")
endif()
```

### Команды

#### cmake .
Генерация проекта и системы билда

```bash
mkdir build
cd build
cmake ../.
```

Использование дополнительных параметров при сборке

#### cmake --build .
Компиляция/Линковка проекта

### Переменные
#### CMAKE_CXX_STANDARD
#### CMAKE_CXX_STANDARD_REQUIRED
#### PROJECT_BINARY_DIR
#### PROJECT_SOURCE_DIR