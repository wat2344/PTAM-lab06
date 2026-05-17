# PTAM-lab06

Данная лабораторная работа посвящена изучению средств пакетирования на примере CPack.

## Tutorial

```bash
export GITHUB_USERNAME=wat2344
export GITHUB_EMAIL=pavel.khokhlov.07@inbox.ru
alias edit=nano
alias gsed=sed   
cd ${GITHUB_USERNAME}/workspace
pushd .
source scripts/activate
git clone https://github.com/${GITHUB_USERNAME}/PTAM-lab05 projects/PTAM-lab06
cd projects/PTAM-lab06
git remote remove origin
git remote add origin https://github.com/${GITHUB_USERNAME}/PTAM-lab06
```

### Добавление версионирования в CMakeLists.txt

*Вставим после project(banking) следующие строки:*

```bash
gsed -i '/project(banking)/a\
set(PRINT_VERSION_MAJOR 0)
' CMakeLists.txt

gsed -i '/project(banking)/a\
set(PRINT_VERSION_MINOR 1)
' CMakeLists.txt

gsed -i '/project(banking)/a\
set(PRINT_VERSION_PATCH 0)
' CMakeLists.txt

gsed -i '/project(banking)/a\
set(PRINT_VERSION_TWEAK 0)
' CMakeLists.txt

gsed -i '/project(banking)/a\
set(PRINT_VERSION\
  \${PRINT_VERSION_MAJOR}.\${PRINT_VERSION_MINOR}.\${PRINT_VERSION_PATCH}.\${PRINT_VERSION_TWEAK})
' CMakeLists.txt

gsed -i '/project(banking)/a\
set(PRINT_VERSION_STRING "v\${PRINT_VERSION}")
' CMakeLists.txt
```

### Создание файлов описания

```bash
touch DESCRIPTION && edit DESCRIPTION   # краткое описание библиотеки
touch ChangeLog.md
export DATE="$(LANG=en_US date +'%a %b %d %Y')"
cat > ChangeLog.md <<EOF
* ${DATE} wat2344 <pavel.khokhlov.07@inbox.ru> 0.1.0.0
- Initial CPack release
EOF
```

### Конфигурация CPack (CPackConfig.cmake)

```bash
cat > CPackConfig.cmake <<EOF
include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_CONTACT pavel.khokhlov.07@inbox.ru)
set(CPACK_PACKAGE_VERSION_MAJOR \${PRINT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR \${PRINT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH \${PRINT_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION_TWEAK \${PRINT_VERSION_TWEAK})
set(CPACK_PACKAGE_VERSION \${PRINT_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_FILE \${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Banking library with Account and Transaction classes")

set(CPACK_RESOURCE_FILE_LICENSE \${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
set(CPACK_RESOURCE_FILE_README \${CMAKE_CURRENT_SOURCE_DIR}/README.md)

set(CPACK_RPM_PACKAGE_NAME "banking-devel")
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "Development")
set(CPACK_RPM_CHANGELOG_FILE \${CMAKE_CURRENT_SOURCE_DIR}/ChangeLog.md)
set(CPACK_RPM_PACKAGE_RELEASE 1)

set(CPACK_DEBIAN_PACKAGE_NAME "libbanking-dev")
set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)

include(CPack)
EOF
```

*Подключим CPack в конец CMakeLists.txt:*

```bash
cat >> CMakeLists.txt <<EOF

include(CPackConfig.cmake)
EOF
```

### Локальная сборка пакетов

```bash
cmake -H. -B_build
cmake --build _build
cd _build
cpack -G "TGZ"         
cpack -G "DEB"          
cpack -G "RPM"          
cd ..
```

### Сохранение артефактов

```bash
mkdir artifacts
mv _build/*.tar.gz artifacts/
mv _build/*.deb artifacts/ 2>/dev/null || true
mv _build/*.rpm artifacts/ 2>/dev/null || true
mv _build/*.dmg artifacts/ 2>/dev/null || true
```

### Git-теги и релизы

```bash
git add .
git commit -m "Добавлена конфигурация CPack"
git tag v0.1.0.0
git tag v0.1.0.1
git push origin main --tags
```

*Релизы на GitHub созданы вручную через веб-интерфейс (вкладка Releases) с прикреплением пакетов из папки artifacts.*

### Настройка GitHub Actions для автоматической сборки пакетов по тегам

*Создадим файл .github/workflows/cpack.yml:*

```bash
mkdir -p .github/workflows
cat > .github/workflows/cpack.yml <<EOF
name: CPack

on:
  push:
    tags:
      - 'v*'

jobs:
  build-and-release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install dependencies
        run: sudo apt-get update && sudo apt-get install -y cmake build-essential rpm

      - name: Configure
        run: cmake -B build -DCMAKE_BUILD_TYPE=Release

      - name: Build
        run: cmake --build build

      - name: Package TGZ
        run: cd build && cpack -G TGZ

      - name: Package DEB
        run: cd build && cpack -G DEB

      - name: Package RPM
        run: cd build && cpack -G RPM

      - name: Create Release
        uses: softprops/action-gh-release@v1
        with:
          files: build/*.tar.gz build/*.deb build/*.rpm
        env:
          GITHUB_TOKEN: \${{ secrets.GITHUB_TOKEN }}
EOF
```

*Добавим и запушим workflow:*

```bash
git add .github/workflows/cpack.yml
git commit -m "Добавлен GitHub Actions для автоматической сборки пакетов CPack"
git push origin main
```

## Homework

В рамках домашнего задания настроена автоматическая генерация пакетов (.tar.gz, .deb, .rpm, .dmg) при создании тега, а также их публикация в релизах GitHub. Конфигурация CPack позволяет легко расширить набор пакетов.

### Цель

Научить систему CI (GitHub Actions) при каждом новом теге (например, v0.1.0.0, v0.1.0.1):

-собирать проект,
-создавать пакеты с помощью CPack (.tar.gz, .deb, .rpm)
-загружать эти пакеты в соответствующий релиз на GitHub.

### Результаты

-Созданы пакеты .tar.gz, .deb, .rpm (и .dmg на macOS).
-Теги v0.1.0.0 и v0.1.0.1 присутствуют в репозитории.
-Релизы на GitHub созданы (вручную для первых двух тегов, автоматически для последующих).
-Настроен CI (GitHub Actions), который по тегу собирает пакеты и создаёт релиз.

### Реализация 

#### 1. Создан файл .github/workflows/cpack.yml со следующим содержимым:
```bash
name: CPack

on:
  push:
    tags:
      - 'v*'

jobs:
  build-and-release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install dependencies
        run: sudo apt-get update && sudo apt-get install -y cmake build-essential rpm
      - name: Configure
        run: cmake -B build -DCMAKE_BUILD_TYPE=Release
      - name: Build
        run: cmake --build build
      - name: Package TGZ
        run: cd build && cpack -G TGZ
      - name: Package DEB
        run: cd build && cpack -G DEB
      - name: Package RPM
        run: cd build && cpack -G RPM
      - name: Create Release
        uses: softprops/action-gh-release@v1
        with:
          files: build/*.tar.gz build/*.deb build/*.rpm
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

#### 2. Логика работы:

-Workflow запускается только при пуше тега, начинающегося с v (например, v0.1.0.0).

-Устанавливаются необходимые пакеты (cmake, rpm для сборки .rpm).

-Проект конфигурируется и собирается.

-CPack генерирует пакеты форматов TGZ, DEB, RPM.

-С помощью экшена softprops/action-gh-release создаётся релиз с тем же тегом, и все пакеты автоматически прикрепляются к нему.

#### 3. Ручное создание релизов для первых тегов:

Для тегов v0.1.0.0 и v0.1.0.1 релизы были созданы вручную (через веб-интерфейс GitHub) с прикреплением собранных локально пакетов.

### Вывод

В ходе лабораторной работы освоены возможности CPack для упаковки C++ проектов в различные форматы, а также интеграция с Git и GitHub Actions для автоматического выпуска релизов. Это позволяет упростить распространение библиотеки или приложения в виде готовых бинарных пакетов.
