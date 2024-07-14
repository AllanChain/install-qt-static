# Install Qt Static Action

This action will install and set up a static version of Qt.

> Many thanks to [@gmh5225](https://github.com/gmh5225)'s work.
> The code is developed based on https://github.com/gmh5225/static-build-qt6

Statically linking Qt can often drastically reduce your binary size, and makes deploying the app easier. However, Qt does not provide official static builds. This repo builds static Qt libraries and provide a GitHub Action to use them. An example usage looks like this:
```yaml
    - uses: AllanChain/install-qt-static@v6.7
    - name: Build app
      run: |
        # Use qt-cmake to automatically use the toolchain
        qt-cmake .
        cmake --build .
```
The version of the action is the same as the version of Qt. See tags and releases for a full list of built Qt libraries. Please note that it is not recommended to use branches such as `v6` directly.

This repo only provide Windows (amd64) and macOS (x86_64 and arm64) builds. Linux builds are not included because Linux Qt apps prefer dynamic linking.

Only `qtbase` is included in the binaries. If you need more Qt libraries, you can fork this repo and add your own.

## Example workflow

```yaml
name: Build

on: [workflow_dispatch, push]

jobs:
  Build:
    strategy:
      fail-fast: false
      matrix:
        os: [windows-latest, macos-latest]
        include:
        - os: windows-latest
          os-caption: windows
        - os: macos-latest
          os-caption: macos

    runs-on: ${{matrix.os}}
    steps:
    - uses: actions/checkout@v4
    - uses: AllanChain/install-qt-static@v6
    - uses: ilammy/msvc-dev-cmd@v1
      if: contains(matrix.os, 'windows')

    - name: Build Project
      run: |
        mkdir build
        cd build
        qt-cmake -DCMAKE_BUILD_TYPE=MinSizeRel -DCMAKE_OSX_ARCHITECTURES="x86_64;arm64" ..
        cmake --build . --parallel

    - name: Packing (Windows)
      if: contains(matrix.os, 'windows')
      run: |
        mkdir release
        move build\Release\myapp.exe release

    - name: Packing (macOs)
      if: contains(matrix.os, 'macos')
      run: |
        mkdir release
        brew install create-dmg
        create-dmg release/myapp.dmg build/myapp.app

    - name: Create Artifact
      uses: actions/upload-artifact@v4
      with:
         name: "myapp (${{matrix.os-caption}})"
         path: ./release/*
```

## Developement

The [build action](.github/workflows/build.yml) builds and releases Qt builds with a tag is pushed. The version is stored in `action.yml`. To update the version, you need to run `./bump <version>`. To rebuild, you need to run `./bump`.
