## Laboratory work VI

Данная лабораторная работа посвещена изучению средств пакетирования на примере **CPack**

```sh
$ open https://cmake.org/Wiki/CMake:CPackPackageGenerators
```



## Homework

После того, как вы настроили взаимодействие с системой непрерывной интеграции,</br>
обеспечив автоматическую сборку и тестирование ваших изменений, стоит задуматься</br>
о создание пакетов для измениний, которые помечаются тэгами (см. вкладку [releases](https://github.com/tp-labs/lab06/releases)).</br>
Пакет должен содержать приложение _solver_ из [предыдущего задания](https://github.com/tp-labs/lab03#задание-1)
Таким образом, каждый новый релиз будет состоять из следующих компонентов:
- архивы с файлами исходного кода (`.tar.gz`, `.zip`)
- пакеты с бинарным файлом _solver_ (`.deb`, `.rpm`, `.msi`, `.dmg`)


---

Для начала клонируем репозиторий 4 лабораторной работы с уже готовыми файлами CMakeLists.txt для всех библиотек

---


# 1) Создаем файл CMakeLists.txt в корне репозитория lab06, который мы сначала привязываем к гитхабу

```bash
cmake_minimum_required(VERSION 3.4)
project(lab06)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/solver_application)

include(CPack.cmake)
```

Подключение `include(CPack.cmake)` в CMakeLists.txt позволяет использовать функционал CPack для создания пакетов и установочных файлов.


# 2) Создаем файл CPack.cmake, где указываем необходимые параметры нашего проекта

```bash
include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_CONTACT mihail_160505@@mail.ru)
set(CPACK_PACKAGE_VERSION ${PRINT_VERSION})
set(CPACK_PACKAGE_NAME "solver")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "static C++ library for solver")
set(CPACK_PACKAGE_VENDOR "kuznetsovvvv")
set(CPACK_PACKAGE_PACK_NAME "solver-${PRINT_VERSION}")

set(CPACK_SOURCE_INSTALLED_DIRECTORIES 
  "${CMAKE_SOURCE_DIR}/solver_application; solver_application"
  "${CMAKE_SOURCE_DIR}/solver_lib; solver_lib"
  "${CMAKE_SOURCE_DIR}/formatter_ex_lib; formatter_ex_lib"
  "${CMAKE_SOURCE_DIR}/formatter_lib; formatter_lib")

set(CPACK_RESOURCE_FILE_README ${CMAKE_CURRENT_SOURCE_DIR}/README.md)

set(CPACK_SOURCE_GENERATOR "TGZ;ZIP")

set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)

set(CPACK_DEBIAN_PACKAGE_VERSION ${PRINT_VERSION})
set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "all")
set(CPACK_DEBIAN_PACKAGE_DESCRIPTION "solves equations")

set(CPACK_GENERATOR "DEB;RPM")

set(CPACK_RPM_PACKAGE_SUMMARY "solves equations")

include(CPack)
```

# 3) Создаем папку .github/workflows , в которой создаем два файла сборки: CI.yml и CI_release.yml

Назначение 1-го файла:
- Этот файл создает рабочий процесс (workflow) для билдинга проекта в GitHub Actions при каждом push или pull_request в ветку main, он нужен для:
- Выполняет проверку кода репозитория.
- Настраивает сборку проекта с помощью CMake.

CI.yml:
```bash
name: CMake

on:
 push:
  branches: [master]
 pull_request:
  branches: [master]

jobs: 
 build_Linux:

  runs-on: ubuntu-latest

  steps:
  - uses: actions/checkout@v3

  - name: Configure Solver
    run: cmake ${{github.workspace}} -B ${{github.workspace}}/build

  - name: Build Solver
    run: cmake --build ${{github.workspace}}/build
```

Назначение 2-го файла:
- Файл создает рабочий процесс (workflow) для билдинга проекта в GitHub Actions при каждом push нового тэга версии (с тэгом в формате v*.*.*). Он выполняет следующие действия:
- Запускает сборку проекта.
- Создает упакованные файлы для различных платформ.
- Создает и публикует релиз в GitHub с созданными артефактами (пакеты DEB, RPM, TAR.GZ, ZIP).
     
CI_release.yml:
```bash
name: CMake

on:
 push:
   tags:
     - v*.*.*

jobs: 

  build_packages_Linux:

    runs-on: ubuntu-latest
    
    permissions:
      contents: write

    steps:
    - uses: actions/checkout@v3

    - name: Configure Solver
      run: cmake ${{github.workspace}} -B ${{github.workspace}}/build -D PRINT_VERSION=${GITHUB_REF_NAME#v}

    - name: Build Solver
      run: cmake --build ${{github.workspace}}/build

    - name: Build package
      run: cmake --build ${{github.workspace}}/build --target package

    - name: Build source package
      run: cmake --build ${{github.workspace}}/build --target package_source

    - name: Make a release
      uses: ncipollo/release-action@v1.14.0
      with:
        artifacts: "build/*.deb,build/*.rpm,build/*.tar.gz,build/*.zip"
        token: ${{ secrets.GITHUB_TOKEN }}
```

Для создания тэга пишем в терминал:

```bash
Команда:git tag -a v*.*.* -m "v*.*.*"
```
(Пишем версию тэга, например:git tag -a v1.0.0 -m "v1.0.0")
После пушим тэг:
```bash
Команда:git push origin v1.0.0
```
Все тэги можно посмотреть перейдя по кнопке Tags в репозитории справа от кнопки Branch.

