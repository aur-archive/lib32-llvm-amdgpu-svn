# Maintainer: Lone_Wolf <lonewolf at xs4all dot nl>
# Contributor: Evangelos Foutras <foutrelis@gmail.com>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>
# Contributor: Sebastian Nowicki <sebnow@gmail.com>
# Contributor: Devin Cofer <ranguvar{AT]archlinux[DOT}us>
# Contributor: Tobias Kieslich <tobias@justdreams.de>
# Contributor: Geoffroy Carrier <geoffroy.carrier@aur.archlinux.org>
# Contributor: Tomas Lindquist Olsen <tomas@famolsen.dk>
# Contributor: Roberto Alsina <ralsina@kde.org>
# Contributor: Gerardo Exequiel Pozzi <vmlinuz386@yahoo.com.ar>

pkgname=lib32-llvm-amdgpu-svn
_svnname=llvm-amdgpu-svn
pkgver=179091 
pkgrel=1
pkgdesc='LIB32 Low Level Virtual Machine with AMDGPU enabled, build from upstream svn trunk version'
arch=('x86_64')
url="http://llvm.org/"
license=('custom:University of Illinois/NCSA')
depends=('lib32-libffi' 'llvm-amdgpu-svn' 'gcc-libs-multilib')
makedepends=('python2' 'subversion')
source=("$_svnname::svn+http://llvm.org/svn/llvm-project/llvm/trunk")
conflicts=('lib32-llvm')
provides=('lib32-llvm')
md5sums=('SKIP')
# _svntrunk='http://llvm.org/svn/llvm-project/llvm/trunk'
# _svnmod='llvm'

pkgver() {
  cd "$SRCDEST/$_svnname"
  svnversion | tr -d [A-z]
}

prepare() {
  cd "$_svnname"
  # Fix installation directories, ./configure doesn't seem to set them right
  sed -i -e 's:\$(PROJ_prefix)/etc/llvm:/etc/llvm:' \
         -e 's:\$(PROJ_prefix)/lib:$(PROJ_prefix)/lib32/llvm:' \
         -e 's:\$(PROJ_prefix)/docs/llvm:$(PROJ_prefix)/share/doc/llvm:' \
    Makefile.config.in
  sed -i '/ActiveLibDir = ActivePrefix/s:lib:lib32/llvm:' \
    tools/llvm-config/llvm-config.cpp
  sed -i 's:LLVM_LIBDIR="${prefix}/lib":LLVM_LIBDIR="${prefix}/lib32/llvm":' \
    autoconf/configure.ac \
    configure

  # Fix insecure rpath (http://bugs.archlinux.org/task/14017)
  sed -i 's:$(RPATH) -Wl,$(\(ToolDir\|LibDir\|ExmplDir\))::g' Makefile.rules
}

build() {
  cd "$_svnname"

  export CC="gcc -m32"
  export CXX="g++ -m32"
  export PKG_CONFIG_PATH="/usr/lib32/pkgconfig"

  # Include location of libffi headers in CPPFLAGS
  export CPPFLAGS="$CPPFLAGS $(pkg-config --cflags libffi)"
  
  # Apply strip option to configure
  _optimized_switch="enable"
  [[ $(check_option strip) == n ]] && _optimized_switch="disable"

  # We had to force host and target to get
  # a proper triplet reported by llvm

  ./configure \
    --with-python=/usr/bin/python2 \
    --prefix=/usr \
    --libdir=/usr/lib32/llvm \
    --sysconfdir=/etc \
    --enable-shared \
    --enable-libffi \
    --enable-targets=all \
    --enable-experimental-targets=R600 \
    --disable-expensive-checks \
    --disable-debug-runtime \
    --disable-assertions \
    --with-binutils-include=/usr/include \
    --host=i386-pc-linux-gnu \
    --target=i386-pc-linux-gnu \
    --$_optimized_switch-optimized

  make REQUIRES_RTTI=1
}

package() {

  cd "$_svnname"
  make DESTDIR="$pkgdir" install

  # Remove duplicate files installed by the OCaml bindings
  rm -rf "$pkgdir"/usr/lib

  # Fix permissions of static libs
  chmod -x "$pkgdir"/usr/lib32/llvm/*.a

  mv "$pkgdir/usr/bin/i386-pc-linux-gnu-llvm-config" "$pkgdir/usr/lib32/llvm-config"

  # Get rid of example Hello transformation
  rm "$pkgdir"/usr/lib32/llvm/*LLVMHello.*

  # Symlink the gold plugin where clang expects it
  ln -s llvm/LLVMgold.so "$pkgdir/usr/lib32/LLVMgold.so"

  # Add ld.so.conf.d entry
  install -d "$pkgdir/etc/ld.so.conf.d"
  echo /usr/lib32/llvm >"$pkgdir/etc/ld.so.conf.d/llvm32.conf"

  mv "$pkgdir"/usr/include/llvm/Config/*config.h "$pkgdir/"
  rm -rf "$pkgdir"/usr/{bin,include,share/{doc,man}}

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
  install -d "$pkgdir/usr/include/llvm/Config"
  mv "$pkgdir/config.h" "$pkgdir/usr/include/llvm/Config/config-32.h"
  mv "$pkgdir/llvm-config.h" "$pkgdir/usr/include/llvm/Config/llvm-config-32.h"

  mkdir "$pkgdir"/usr/bin
  mv "$pkgdir/usr/lib32/llvm-config" "$pkgdir/usr/bin/llvm-config32"
}
