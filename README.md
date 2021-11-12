# LuxCoreRender Mac Files #
This repository contains the source dependencies for the Mac OS version of LuxCoreRender.
The boost compile used pyenv to build the libraries against versions used in blender 2.8.


# macOS 11.4 BIG SUR instructions (assumes CMake 3.20+) #
---------------------

These instructions based on the original macOS 11+ guide for Compiling LuxCore:
  https://wiki.luxcorerender.org/Compiling_LuxCore#macOS_11.2B

```
brew install pyenv cmake bison gnu-sed gnu-tar xz libtool autoconf automake ispc bzip2 jpeg
```

---------------------

After installing pyenv, the following steps to create or modify .zshrc must be performed in order for pyenv to correctly set python and pip version:

* https://gist.github.com/josemarimanio/9e0c177c90dee97808bad163587e80f8

Be sure to follow extra steps for 11.4, i.e. add this to .zshrc : `eval "$(pyenv init --path)"`
Once .zshrc is created or modified, you must restart Terminal program for changes to take effect.

```
PATH="/usr/local/opt/gnu-sed/libexec/gnubin:$PATH"
PATH="/usr/local/opt/gnu-tar/libexec/gnubin:$PATH"
pyenv init
eval "$(pyenv init --path)"
eval "$(pyenv init -)"

env PYTHON_CONFIGURE_OPTS="--enable-framework" CFLAGS="-I$(brew --prefix openssl)/include -I$(brew --prefix bzip2)/include -I$(brew --prefix readline)/include -I$(xcrun --show-sdk-path)/usr/include" LDFLAGS="-L$(brew --prefix openssl)/lib -L$(brew --prefix readline)/lib -L$(brew --prefix zlib)/lib -L$(brew --prefix bzip2)/lib" pyenv install --patch 3.7.4 < <(curl -sSL https://github.com/python/cpython/commit/8ea6353.patch\?full_index\=1)

pyenv global 3.7.4
pip install --upgrade pip

curl -o numpy-1.19.4-cp37-cp37m-macosx_11_0_x86_64.whl https://files.pythonhosted.org/packages/46/09/1bae812d4afa67e365d3d1dbdc0e9071ba7678611f52b49353d6104ae8ff/numpy-1.19.4-cp37-cp37m-macosx_10_9_x86_64.whl
pip install numpy-1.19.4-cp37-cp37m-macosx_11_0_x86_64.whl

pip install pillow
pip install pyside2
```

# Compile Dependencies #
```
pyenv shell 3.7.4

#git clone https://github.com/LuxCoreRender/MacOSCompileDeps.git
git clone https://github.com/danielbui78/MacOSCompileDeps.git
cd MacOSCompileDeps
./cut_deps_release
cd ..
```

This creates a MacDistFiles.tar.gz in the MacOSCompileDeps folder.

# Compile LuxCore #
```
#git clone https://github.com/LuxCoreRender/LuxCore.git
git clone https://github.com/danielbui78/LuxCore.git
cd LuxCore
tar xzf ../MacOSCompileDeps/MacDistFiles.tar.gz
```

-------------------------------

For commandline build with make:
```
export PATH="/usr/local/opt/bison/bin:/usr/local/bin:$PATH"
DEPS_SOURCE=`pwd`/macos
mkdir build
cd  build
cmake -DOSX_DEPENDENCY_ROOT=$DEPS_SOURCE -DCMAKE_BUILD_TYPE=Release ..
make
```

--------------------------------

For Xcode:
```
export PATH="/usr/local/opt/bison/bin:/usr/local/bin:$PATH"
DEPS_SOURCE=`pwd`/macos
mkdir build
cd  build
cmake -G Xcode -DOSX_DEPENDENCY_ROOT=$DEPS_SOURCE -DCMAKE_BUILD_TYPE=Release -T buildsystem=1 ..
cd ..
```
Open Xcode project located in LuxCore/build folder.  Select "ALL_BUILD" as the target and then Product->Build For->Profiling.

# Make bundle #

Edit scripts/macos/codesign.sh to match your certificate ( something like A1CD139B9FD66DE9D474D420C1899EA96A622B9A )

For commandline build:
```
./scripts/macos/pack_lux_osx.sh
```

For XCode build:
```
./scripts/macos/pack_lux_osx_xcode.sh
```

This should produce a distributable .dmg.
