## Laboratory work VI

## Homework

После того, как вы настроили взаимодействие с системой непрерывной интеграции,</br>
обеспечив автоматическую сборку и тестирование ваших изменений, стоит задуматься</br>
о создание пакетов для измениний, которые помечаются тэгами.</br>
Пакет должен содержать приложение _solver_ из [предыдущего задания](https://github.com/tp-labs/lab03#задание-1)
Таким образом, каждый новый релиз будет состоять из следующих компонентов:
- архивы с файлами исходного кода (`.tar.gz`, `.zip`)
- пакеты с бинарным файлом _solver_ (`.deb`, `.rpm`, `.msi`, `.dmg`)
Для этого нужно добавить ветвление в конфигурационные файлы для **CI** со следующей логикой:</br>
если **commit** помечен тэгом, то необходимо собрать пакеты (`DEB, RPM, WIX, DragNDrop, ...`) </br>
и разместить их на сервисе **GitHub**.</br>

Дополняем корневой CMakeLists.txt</br>
1) указываем куда положить исполняемые файлы и статические, динамические библиотеки:
```br
install(TARGETS solver
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)
```
2) указываем имя проекта, версию, описание, имя и почту создателя
```br
set(CPACK_PACKAGE_NAME "solver")
set(CPACK_PACKAGE_VERSION "1.0.0")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Quadratic equation solver")
set(CPACK_PACKAGE_VENDOR "bryansevnikolay-netizen")
set(CPACK_PACKAGE_CONTACT "bryansev.nikolay@gmail.com")
```
3) указываем нужные типы пакетов и зависимости для каждого из них
```br
set(CPACK_GENERATOR "DEB;RPM;DragNDrop;WIX;ZIP;TGZ")

set(CPACK_DEBIAN_PACKAGE_DEPENDS "libc6 (>= 2.28)")

set(CPACK_RPM_PACKAGE_REQUIRES "glibc >= 2.28")

set(CPACK_WIX_UPGRADE_GUID "809da3ca-e1c5-4304-9375-31ef0b5d16d3")

set(CPACK_DMG_VOLUME_NAME "Solver Installer")
set(CPACK_DMG_FORMAT "UDBZ")
```
4) настройка архивов с исходниками
```br
set(CPACK_SOURCE_GENERATOR "TGZ;ZIP")
set(CPACK_SOURCE_IGNORE_FILES 
    "/build/"
    "/.git/"
    ".*~$"
    "/.github/"
)
```
5) активация CPack
```br
include(CPack)
```
Создаем файл сборки</br>
1) вводим название сборки и условие работы (только с версиямм)
```br
name: Build, Test and Package
on:
  push:
    branches: [ main, master ]
    tags:
      - 'v*'
  pull_request:
    branches: [ main, master ]
```
2) создание матрицы с различными ОС и указание их параметров: название, типы и расширения пакетов, необходимость RPM
```br
jobs:
  build:
    name: ${{ matrix.name }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        include:
          - name: Linux DEB
            os: ubuntu-latest
            cpack_generator: DEB
            package_ext: deb
            need_rpm: false

          - name: Linux RPM
            os: ubuntu-latest
            cpack_generator: RPM
            package_ext: rpm
            need_rpm: true

          - name: Windows MSI
            os: windows-latest
            cpack_generator: WIX
            package_ext: msi
            need_rpm: false

          - name: macOS DMG
            os: macos-latest
            cpack_generator: DragNDrop
            package_ext: dmg
            need_rpm: false
```
3) установка кода, RPM (при необходимости), конфигурация CMake и сборка проекта с использованием нескольких ядер в зависимости от ОС
```br
    steps:
      - uses: actions/checkout@v4
      - name: Install RPM
        if: matrix.need_rpm
        run: sudo apt-get update && sudo apt-get install -y rpm

      - name: Configure CMake
        run: cmake -B build -S .

      - name: Build
        run: |
          if [[ "${{ runner.os }}" == "Windows" ]]; then
            cmake --build build --config Release -j$(nproc)
          elif [[ "${{ runner.os }}" == "macOS" ]]; then
            cmake --build build --config Release -j$(sysctl -n hw.ncpu)
          else
            cmake --build build --config Release -j$(nproc)
          fi
        shell: bash
```
4) создание пакетов и их загрузка в релиз
```br
      - name: Create package 
        if: startsWith(github.ref, 'refs/tags/')
        run: |
          cd build
          cpack -G ${{ matrix.cpack_generator }}

      - name: Upload to Release
        if: startsWith(github.ref, 'refs/tags/')
        uses: softprops/action-gh-release@v1
        with:
          files: build/*.${{ matrix.package_ext }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
5) задание для исходников
```br
  sources:
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/')
    steps:
      - uses: actions/checkout@v4
      - name: Create source packages
        run: |
          git archive --format=zip --output solver-source.zip HEAD
          git archive --format=tar.gz --output solver-source.tar.gz HEAD
      - name: Upload sources
        uses: softprops/action-gh-release@v1
        with:
          files: |
            solver-source.zip
            solver-source.tar.gz
        env:
            GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
```
Copyright (c) 2015-2021 The ISC Authors
```
