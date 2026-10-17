# FreeCAD RPM

[![Build FreeCAD RPM](https://github.com/MidnightRAT/FreeCAD-rpm/actions/workflows/build-rpm.yml/badge.svg)](https://github.com/MidnightRAT/FreeCAD-rpm/actions/workflows/build-rpm.yml)
[![Latest Release](https://img.shields.io/github/v/release/MidnightRAT/FreeCAD-rpm)](https://github.com/MidnightRAT/FreeCAD-rpm/releases/latest)

RPM packaging for [FreeCAD](https://github.com/FreeCAD/FreeCAD) — open-source parametric 3D CAD modeler, built with **maximum use of system libraries** from Fedora `-devel` packages.

## Donate

[![Ko-fi](https://img.shields.io/badge/Ko--fi-Support%20me%20on%20Ko--fi-ff5e5b?style=for-the-badge&logo=ko-fi)](https://ko-fi.com/redeyesnightrat)

## Dependency strategy

Where possible the package uses **system libraries** from Fedora `-devel` packages:

- Coin4 4.0.10, OpenCASCADE, VTK 9.5+
- Boost 1.90, Eigen3 5.0.1
- PySide6 6.11, Shiboken6, Pivy 0.6.9+
- fmt, yaml-cpp, nlohmann_json, Microsoft GSL
- OndselSolver, libE57Format, libkdtree++, orocos-kdl
- MEDFile, Xerces-C, ICU, OpenMPI, HDF5
- GTest/GMock, libspnav

Only libraries with no compatible version in the distribution (or requiring custom patches) are **bundled**:

- SMESH 7.7.1.0 (custom Salome fork, not in Fedora)
- zipios++ 0.1.5.9 (Fedora version too old, API incompatible)
- lru-cache (header-only, tiny)
- 3Dconnexion (driver SDK, not packaged)

The system-detection logic lives in CMake options (`FREECAD_USE_EXTERNAL_*`) — all enabled where Fedora provides compatible versions.

## Installation

### From COPR (Fedora 44/45, Recommended)

```bash
sudo dnf copr enable chirikrat/FreeCAD-rpm
sudo dnf install freecad
```

### From GitHub Release

Download the latest `freecad-*.x86_64.rpm` from [Releases](https://github.com/MidnightRAT/FreeCAD-rpm/releases) and install:

```bash
sudo dnf install freecad-*.x86_64.rpm
```

### Build from Source

```bash
# Install build tools
sudo dnf install -y rpm-build rpmdevtools git git-lfs wget curl \
  cmake ninja-build gcc gcc-c++ make patch pkgconf \
  swig python3-devel desktop-file-utils libappstream-glib \
  Coin4-devel opencascade-devel vtk-devel \
  boost-devel eigen3-devel pybind11-devel \
  python3-pycxx-devel python3-pyside6-devel shiboken6-devel \
  python3-pivy python3-matplotlib \
  fmt-devel yaml-cpp-devel nlohmann-json-devel ms-gsl-devel \
  ondselsolver-devel libE57Format-devel libkdtree++-devel orocos-kdl-devel \
  gtest-devel gmock-devel hdf5-static openmpi-devel xerces-c-devel \
  libicu-devel libspnav-devel \
  mesa-libGL-devel mesa-libGLU-devel freetype-devel \
  qt6-qtbase-devel qt6-qtsvg-devel qt6-qttools-devel

# Determine version and build RPM
VERSION=1.1.4
rpmdev-setuptree
wget -q "https://github.com/FreeCAD/FreeCAD/releases/download/${VERSION}/freecad_source_${VERSION}.tar.gz" \
  -O ~/rpmbuild/SOURCES/freecad_source_${VERSION}.tar.gz
cp README.md ~/rpmbuild/SOURCES/
cp patches/*.patch ~/rpmbuild/SOURCES/ 2>/dev/null || true
sed -e "s/@VERSION@/${VERSION}/g" -e "s/@RELEASE@/1/g" \
  freecad.spec > ~/rpmbuild/SPECS/freecad.spec
rpmbuild -ba ~/rpmbuild/SPECS/freecad.spec
```

> The `%prep`/`%build` uses system libraries resolved at configure time from the `-devel` packages listed above.
> `rpmbuild -ba` produces **both** the SRPM (`~/rpmbuild/SRPMS/*.src.rpm`) and
> the binary RPM (`~/rpmbuild/RPMS/x86_64/*.x86_64.rpm`). Verify the artifacts with:

```bash
rpmlint ~/rpmbuild/RPMS/x86_64/freecad*.rpm   # 0 real errors
rpm -qlp ~/rpmbuild/RPMS/x86_64/freecad*.rpm  # file list
ldd /usr/lib64/freecad/bin/FreeCAD            # no "not found"
/usr/bin/freecad --help                       # smoke test
```

## Local mock build (Fedora 44/45 with RPMFusion)

```bash
# Install mock and RPMFusion
sudo dnf install -y mock
sudo usermod -a -G mock $USER
newgrp mock

# Build SRPM first
rpmbuild -bs --define "_sourcedir $PWD" --define "_srcrpmdir $PWD" freecad.spec

# Build for Fedora 44 with RPMFusion free
mock -r fedora+rpmfusion_free-44-x86_64 --rebuild freecad-*.src.rpm

# Build for Fedora 45 with RPMFusion free
mock -r fedora+rpmfusion_free-45-x86_64 --rebuild freecad-*.src.rpm
```

Resulting RPMs in `/var/lib/mock/fedora+rpmfusion_free-44-x86_64/result/` (and 45).

## CI/CD

### GitHub Actions

Automatically:
1. Checks for new FreeCAD releases (weekly schedule)
2. Builds src.rpm and x86_64.rpm in Fedora 44/45 containers
3. Uploads artifacts to GitHub Releases

### COPR

Automatically builds for Fedora 44/45 from the latest main branch:
- [COPR Project Page](https://copr.fedorainfracloud.org/coprs/chirikrat/FreeCAD-rpm/)

**Note:** Workflows do not trigger on changes to README.md or `.copr/**`.

## Project Structure

```
FreeCAD-rpm/
├── .copr/Makefile              # COPR SRPM build script
├── .github/workflows/          # GitHub Actions workflow
├── freecad.spec                # RPM spec file
├── patches/                    # Compatibility patches (as needed)
└── README.md
```

## License

LGPL-2.1-or-later AND Zlib AND BSD-3-Clause AND Apache-2.0 AND EUPL-1.2 AND BSL-1.0 AND MIT AND GPL-2.0-or-later AND CC-BY-SA-4.0 (same as FreeCAD)