# Maintainer: Marek Skrobacki <skrobul@skrobul.com>
# Original Maintainer: Sven-Hendrik Haase <sh@lutzhaase.com>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>
# Contributor: Eduardo Romero <eduardo@archlinux.org>
# Contributor: Giovanni Scafora <giovanni@archlinux.org>


pkgname=wine-tlsfix
provides=(wine=1.5.12)
conflicts=(wine)
pkgver=1.5.12
pkgrel=1

_pkgbasever=${pkgver/rc/-rc}

source=(http://prdownloads.sourceforge.net/wine/wine-$_pkgbasever.tar.bz2
0001-wininet-disable-TLS1.1-1.2-by-default.patch
0002-winhttp-disable-TLS1.1-1.2-by-default.patch
)
md5sums=('42d8a0b933768447aa73447c4f0ec2ed'
         'c0b391c156859e6d0f04b25abc449f20'
         'a781fb070ea2004557fb378d2b7daaa9')



pkgdesc="A compatibility layer for running Windows programs"
url="http://www.winehq.com"
arch=(i686 x86_64)
license=(LGPL)
install=wine.install

depends=(
  fontconfig      lib32-fontconfig
  mesa            lib32-mesa
  libxcursor      lib32-libxcursor
  libxrandr       lib32-libxrandr
  libxdamage      lib32-libxdamage
  libxi           lib32-libxi
  gettext         lib32-gettext
  desktop-file-utils
)

makedepends=(autoconf ncurses bison perl fontforge flex prelink
  'gcc>=4.5.0-2'  'gcc-multilib>=4.5.0-2'
  giflib          lib32-giflib
  libpng          lib32-libpng
  gnutls          lib32-gnutls
  libxinerama     lib32-libxinerama
  libxcomposite   lib32-libxcomposite
  libxmu          lib32-libxmu
  libxxf86vm      lib32-libxxf86vm
  libxml2         lib32-libxml2
  libldap         lib32-libldap
  lcms            lib32-lcms
  mpg123          lib32-mpg123
  openal          lib32-openal
  v4l-utils       lib32-v4l-utils
  alsa-lib        lib32-alsa-lib
  oss
  samba
)

optdepends=(
  giflib          lib32-giflib
  libpng          lib32-libpng
  libldap         lib32-libldap
  gnutls          lib32-gnutls
  lcms            lib32-lcms
  libxml2         lib32-libxml2
  mpg123          lib32-mpg123
  openal          lib32-openal
  v4l-utils       lib32-v4l-utils
  libpulse        lib32-libpulse
  alsa-plugins    lib32-alsa-plugins
  alsa-lib        lib32-alsa-lib
  libjpeg-turbo   lib32-libjpeg-turbo
  oss             cups
  samba
)

if [[ $CARCH == i686 ]]; then
  # Strip lib32 etc. on i686
  depends=(${depends[@]/*32-*/})
  makedepends=(${makedepends[@]/*32-*/})
  makedepends=(${makedepends[@]/*-multilib*/})
  optdepends=(${optdepends[@]/*32-*/})
else
  provides=("bin32-wine=$pkgver" "wine-wow64=$pkgver")
  conflicts=('bin32-wine' 'wine-wow64')
  replaces=('bin32-wine')
fi

build() {
  cd "$srcdir"

  # Allow ccache to work
  mv wine-$_pkgbasever wine

  # Get rid of old build dirs
  rm -rf wine-{32,64}-build
  mkdir wine-32-build

  # These additional CFLAGS solve FS#27662
  export CFLAGS="${CFLAGS/-D_FORTIFY_SOURCE=2/} -D_FORTIFY_SOURCE=0"
  export CXXFLAGS="${CXXFLAGS/-D_FORTIFY_SOURCE=2/} -D_FORTIFY_SOURCE=0"
  cd "$srcdir/wine/"
  patch -p1 < $srcdir/0001-wininet-disable-TLS1.1-1.2-by-default.patch
  patch -p1 < $srcdir/0002-winhttp-disable-TLS1.1-1.2-by-default.patch

  cd "$srcdir"

  if [[ $CARCH == x86_64 ]]; then
    msg2 "Building Wine-64..."

    mkdir wine-64-build
    cd "$srcdir/wine-64-build"
    ../wine/configure \
      --prefix=/usr \
      --sysconfdir=/etc \
      --libdir=/usr/lib \
      --with-x \
      --enable-win64

    make

    _wine32opts=(
      --libdir=/usr/lib32
      --with-wine64="$srcdir/wine-64-build"
    )

    export PKG_CONFIG_PATH="/usr/lib32/pkgconfig"
  fi

  msg2 "Building Wine-32..."
  cd "$srcdir/wine-32-build"
  ../wine/configure \
    --prefix=/usr \
    --sysconfdir=/etc \
    --with-x \
    "${_wine32opts[@]}"

  # These additional CFLAGS solve FS#27560 and FS#23277
  make CFLAGS+="-mstackrealign -mincoming-stack-boundary=2" CXXFLAGS+="-mstackrealign -mincoming-stack-boundary=2"
}

package() {
  msg2 "Packaging Wine-32..."
  cd "$srcdir/wine-32-build"

  if [[ $CARCH == i686 ]]; then
    make prefix="$pkgdir/usr" install
  else
    make prefix="$pkgdir/usr" \
      libdir="$pkgdir/usr/lib32" \
      dlldir="$pkgdir/usr/lib32/wine" install

    msg2 "Packaging Wine-64..."
    cd "$srcdir/wine-64-build"
    make prefix="$pkgdir/usr" \
      libdir="$pkgdir/usr/lib" \
      dlldir="$pkgdir/usr/lib/wine" install
  fi
}

# vim:set ts=8 sts=2 sw=2 et:
