pkgname=(ncurses ncurses-compat-libs)
pkgbase=ncurses
pkgver=6.5_20250816
pkgrel=2
pkgdesc="System V Release 4.0 curses emulation library"
arch=('x86_64')
url="https://www.gnu.org/software/ncurses/"
license=('MIT-open-group')
depends=(
    'glibc'
    'gcc-libs'
)
makedepends=('autoconf-archive')
source=(https://invisible-mirror.net/archives/ncurses/current/${pkgbase}-${pkgver/_/-}.tgz
    ncurses-6.3-libs.patch
    ncurses-6.3-pkgconfig.patch)
sha256sums=(300008d64f5f066c8757d23f753a936e8b67d7e7302d19682ef5a4d90cfbd3b4
    dc4261b6642058a9df1c0945e2409b24f84673ddc3a665d8a15ed3580e51ee25
    b8544a607dfbeffaba2b087f03b57ed1fa81286afca25df65f61b04b5f3b3738)

prepare() {
    cd ${pkgbase}-${pkgver/_/-}

    # do not link against test libraries
    patch -Np1 -i ${srcdir}/ncurses-6.3-libs.patch

    # do not leak build-time LDFLAGS into the pkgconfig files:
    # https://bugs.archlinux.org/task/68523
    patch -Np1 -i ${srcdir}/ncurses-6.3-pkgconfig.patch

    mkdir  build compat-libs-build
}

build() {
    cd ${pkgbase}-${pkgver/_/-}

    local configure_args=(
        --with-shared
        --without-debug
        --without-normal
        --with-cxx-shared
        ${configure_options}
    )

    (
        cd build

        ../configure                                         \
            --mandir=/usr/share/man                          \
            --enable-pc-files                                \
            --with-pkg-config-libdir=/usr/lib64/pkgconfig    \
            --with-default-terminfo-dir=/usr/share/terminfo  \
            "${configure_args[@]}"

        make
    )

    (
        cd compat-libs-build

        ../configure              \
            --with-cxx-binding    \
            --with-abi-version=5  \
            "${configure_args[@]}"

        make sources libs
    )
}

package_ncurses() {
    cd ${pkgbase}-${pkgver/_/-}/build

    make DESTDIR=$PWD/DESTDIR install

    install -vDm755 DESTDIR/usr/lib64/libncursesw.so.${pkgver%_*} ${pkgdir}/usr/lib64/libncursesw.so.${pkgver%_*}
    
    rm -v  DESTDIR/usr/lib64/libncursesw.so.${pkgver%_*}
    
    sed -e 's/^#if.*XOPEN.*$/#if 1/' \
        -i DESTDIR/usr/include/curses.h
    cp -av DESTDIR/* ${pkgdir}/

    for lib in ncurses form panel menu ; do
        ln -sfv lib${lib}w.so ${pkgdir}/usr/lib64/lib${lib}.so
        ln -sfv ${lib}w.pc    ${pkgdir}/usr/lib64/pkgconfig/${lib}.pc
    done

    ln -sfv libncursesw.so ${pkgdir}/usr/lib64/libcurses.so

    install -vdm755 ${pkgdir}/usr/share/doc/${pkgname}-${pkgver/_/-}
    cp -v -R ../doc -T ${pkgdir}/usr/share/doc/${pkgname}-${pkgver/_/-}
}

package_ncurses-compat-libs() {
    pkgdesc="Ncurses compatibility libraries"
    depends=('ncurses')

    cd ${pkgbase}-${pkgver/_/-}/compat-libs-build

    install -vdm755 ${pkgdir}/usr/lib64
    cp -av lib/lib*.so.5* ${pkgdir}/usr/lib64    
}
