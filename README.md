# Debian Packaging for LLVM/Clang 9

This repository is loosely based on [llvm-toolchain
9](https://salsa.debian.org/pkg-llvm-team/llvm-toolchain/-/tree/9) from Debian
with additional patches necessary for the [Cling](https://root.cern/cling/)
interactive C++ interpreter. It's purpose is to provide shared LLVM/Clang
libraries for Cling and [xeus-cling](https://github.com/jupyter-xeus/xeus-cling)
(a C++ Jupyter kernel built on top of Cling).

Patched versions of [LLVM](https://llvm.org/) and
[Clang](https://clang.llvm.org/) are maintained by the [ROOT
Project](https://root.cern/) here:

  * LLVM:  http://root.cern/git/llvm.git/
  * Clang: http://root.cern/git/clang.git/

And are also mirrored here:

  * LLVM: https://salsa.debian.org/dimitry-ishenko/root-llvm-9
  * Clang: https://salsa.debian.org/dimitry-ishenko/root-clang-9

and here:

  * LLVM: https://github.com/dimitry-ishenko-cpp/root-llvm-9
  * Clang: https://github.com/dimitry-ishenko-cpp/root-clang-9

Precompiled Debian packages (along with Cling and xeus-cling) can be installed
from
[ppa:ppa-verse/xeus-cling](https://launchpad.net/~ppa-verse/+archive/ubuntu/xeus-cling).

Or you can build them yourself:

```bash
p=llvm-clang-9 b=debian # or another branch

# clone this repo
git clone -b ${b} https://salsa.debian.org/dimitry-ishenko/${p}.git
r=$(cd ${p} && git log -n1 --oneline --no-decorate `git describe --tags --abbrev=0` | cut -d/ -f2)
v=${r%-*}
v=${v#*:}

# download and unpack upstream tarballs
for m in clang llvm; do
    f=${p}_${v}.orig-${m}.tar.xz
    wget https://github.com/llvm/llvm-project/releases/download/llvmorg-${v}/${m}-${v}.src.tar.xz -O ${f}

    mkdir ${p}/${m}
    tar -C ${p}/${m} --strip-components=1 -xJf ${f}
done

(cd ${p} && git archive -o ../${p}_${v}.orig.tar.gz HEAD)

# create source package
dpkg-source -b ${p}

# build .deb package locally...
sudo pbuilder build ${p}_${r}.dsc

# ...or generate .changes file and upload it to a PPA
# NB: change -sa to -sd when upgrading
(cd ${p} && dpkg-genchanges -S -sa -O../${p}_${r}_amd64.changes)

debsign ${p}_${r}_amd64.changes
dput ppa:... ${p}_${r}_amd64.changes

# share and enjoy
```
