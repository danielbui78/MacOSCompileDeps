# yaluxplug-LuxCore Mac Dependency Files #

This README contains build instructions for building yaluxplug-LuxCore for MacOS Big Sur.  

This repository itself contains the source dependencies for the Mac OS version of LuxCoreRender version 2.5, which is the version compatible with yaluxplug-LuxCore.  WARNING: I absolutely do NOT recommend you build the dependencies yourself unless you absolutely have no other choice.  Instead, please use the precompiled MacOS dependencies made for LuxCore 2.5 that are available here: https://github.com/LuxCoreRender/MacOSCompileDeps/releases/tag/luxcorerender_v2.5rc1 under the download link:  MacDistFiles.tar.gz (114 MB).  Then use this README for instructions on setting up brew and pyenv which are necessary to properly generate the Xcode project files.

Prerequisites:
* Big Sur 11.4 - 11.6
* CMake 3.20+
* zsh as default shell

# macOS 11.4 - 11.6 BIG SUR instructions (assumes CMake 3.20+, zsh) #
---------------------

# Step 1. Installing Brew #
These instructions based on the original macOS 11+ guide for Compiling LuxCore:
  https://wiki.luxcorerender.org/Compiling_LuxCore#macOS_11.2B

```
brew install pyenv cmake bison gnu-sed gnu-tar xz libtool autoconf automake ispc bzip2 jpeg
```

---------------------

# Step 2. Installing and setting up pyenv #
After installing pyenv, the following steps to create or modify .zshrc must be performed in order for pyenv to correctly set python and pip version:

* https://gist.github.com/josemarimanio/9e0c177c90dee97808bad163587e80f8

Be sure to follow extra steps for 11.4+, i.e. add this to .zshrc : `eval "$(pyenv init --path)"`
Once .zshrc is created or modified, you must restart Terminal program for changes to take effect.

```
PATH="/usr/local/opt/gnu-sed/libexec/gnubin:$PATH"
PATH="/usr/local/opt/gnu-tar/libexec/gnubin:$PATH"
pyenv init
eval "$(pyenv init -)"
eval "$(pyenv init --path)"

env PYTHON_CONFIGURE_OPTS="--enable-framework" CFLAGS="-I$(brew --prefix openssl)/include -I$(brew --prefix bzip2)/include -I$(brew --prefix readline)/include -I$(xcrun --show-sdk-path)/usr/include" LDFLAGS="-L$(brew --prefix openssl)/lib -L$(brew --prefix readline)/lib -L$(brew --prefix zlib)/lib -L$(brew --prefix bzip2)/lib" pyenv install --patch 3.7.4 < <(curl -sSL https://github.com/python/cpython/commit/8ea6353.patch\?full_index\=1)

pyenv global 3.7.4
pip install --upgrade pip

curl -o numpy-1.19.4-cp37-cp37m-macosx_11_0_x86_64.whl https://files.pythonhosted.org/packages/46/09/1bae812d4afa67e365d3d1dbdc0e9071ba7678611f52b49353d6104ae8ff/numpy-1.19.4-cp37-cp37m-macosx_10_9_x86_64.whl
pip install numpy-1.19.4-cp37-cp37m-macosx_11_0_x86_64.whl

pip install pillow
pip install pyside2
```
-----------------------------

# Step 3. Download or build MacOS Dependency Libraries #
Building the MacOS dependency libraries yourself is highly discouraged unless absolutely necessary.  I recommend you download the precompiled libraries for LuxCore 2.5, which are compatible with the current version of yaluxplug-LuxCore.  

Direct Download link: MacDistFiles.tar.gz (114 MB): https://github.com/LuxCoreRender/MacOSCompileDeps/releases/download/luxcorerender_v2.5rc1/MacDistFiles.tar.gz

# Compile Dependencies (OPTIONAL) #
If you decide to build the libraries yourself for whatever reason, you must first remove the following libpng14 folder which is contained within the Mono framework: "/Library/Frameworks/Mono.framework/Headers -> libpng14".  The Headers folder is a symlink, and on Big Sur 11.6, the Headers folder is linked to the following folder path: "/Library/Frameworks/Mono.framework/Versions/6.12.0/include".  If you do not remove this and any other conflicting libpng folder in your library header/include path, then luxcore will build successfully but when you run the executables, you will receive a RUNTIME ERROR and segfault when trying to read PNG files.  This is why I recommend you use the precompiled libraries whenever possible.

```
pyenv shell 3.7.4

git clone https://github.com/danielbui78/MacOSCompileDeps.git
cd MacOSCompileDeps
./cut_deps_release
cd ..
```

This creates a MacDistFiles.tar.gz in the MacOSCompileDeps folder.

# Step 4. Build yaluxplug-LuxCore Xcode project files #
If you downloaded the precompiled binaries, then substitute the "../MacOSCompileDeps/" path with the correct path to wherever you downloaded it.

```
git clone https://github.com/danielbui78/LuxCore.git
cd LuxCore
tar xzf ../MacOSCompileDeps/MacDistFiles.tar.gz
```

-------------------------------

NOTE: I have not yet been able to complete commandline build with make successfully, using v2.5 and Big Sur.  If you are interested in figuring out a way to build commandline with make on your own, then please refer to the LuxCore Compile guide here: https://wiki.luxcorerender.org/Compiling_LuxCore#macOS_11.2B

Xcode build is currently the only way I know how to successfully build yaluxplug-LuxCore / LuxCore 2.5.

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

# Step 5. Make the application bundle #

Edit scripts/macos/codesign.sh to match your certificate ( something like A1CD139B9FD66DE9D474D420C1899EA96A622B9A )

For XCode build:
```
./scripts/macos/pack_lux_osx_xcode.sh
```

This should produce a distributable .dmg.
