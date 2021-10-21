# Maintainer: Baltazár Radics <baltazar.radics@gmail.com>

_target=xtensa-esp32-elf
pkgname=$_target-gcc
pkgver=12.0.0
_islver=0.24
_overlay_commit=4d8c98d
_snapshot=e5a67458da39de7db8971cc5ec017fa6199a08c1
pkgrel=1
pkgdesc='The GNU Compiler Collection - cross compiler for xtensa esp32 (bare-metal) target'
arch=(x86_64)
url='https://gcc.gnu.org/'
license=(GPL LGPL FDL)
depends=($_target-binutils zlib libmpc)
makedepends=(gmp mpfr $_target-newlib)
optdepends=("$_target-newlib: Standard C library optimized for embedded systems")
options=(!emptydirs !strip)
source=(#https://ftp.gnu.org/gnu/gcc/gcc-$pkgver/gcc-$pkgver.tar.xz{,.sig}
        gcc-$pkgver.zip::https://github.com/D-Programming-GDC/gcc/archive/$_snapshot.zip
        #http://isl.gforge.inria.fr/isl-$_islver.tar.bz2
				https://master.dl.sourceforge.net/project/libisl/isl-$_islver.tar.bz2
        xtensa-overlays-$_overlay_commit.tar.gz::https://codeload.github.com/espressif/xtensa-overlays/tar.gz/$_overlay_commit)
sha256sums=('993e74560e89bd7a1ca19d52ef8b8c50d3fe2db160c275d779f3e04401627e1d'
            'fcf78dd9656c10eb8cf9fbd5f59a0b6b01386205fe1934b3b287a0a1898145c0'
            '88b054b60b8009d02184ed0703b7fe200b8965af5c45268b7e99a11820119344')
validpgpkeys=(F3691687D867B81B51CE07D9BBE43771487328A9  # bpiotrowski@archlinux.org
              86CFFCA918CF3AF47147588051E8B148A9999C34  # evangelos@foutrelis.com
              13975A70E63C361C73AE69EF6EEB81F8981C74C7  # richard.guenther@gmail.com
              33C235A34C46AA3FFB293709A328C3A2C3C45C06) # Jakub Jelinek <jakub@redhat.com>

_basedir=gcc-$_snapshot

prepare() {
	cd $_basedir

	# link isl for in-tree builds
	ln -sf ../isl-$_islver isl

	echo $pkgver > gcc/BASE-VER

	# hack! - some configure tests for header files using "$CPP $CPPFLAGS"
	sed -i "/ac_cpp=/s/\$CPPFLAGS/\$CPPFLAGS -O2/" {libiberty,gcc}/configure
	cp -r ../xtensa-overlays-$_overlay_commit/xtensa_esp32/gcc/* .

	mkdir "$srcdir"/build-{gcc,gcc-nano}
}

_build_gcc() {
	"$srcdir"/$_basedir/configure \
		--libexecdir=/usr/lib \
		--prefix=/usr \
		--target=$_target \
		--with-gmp \
		--with-gnu-as \
		--with-gnu-ld \
		--with-headers=/usr/$_target/include \
		--with-host-libstdcxx='-static-libgcc -Wl,-Bstatic,-lstdc++,-Bdynamic -lm' \
		--with-isl \
		--with-libelf \
		--with-mpc \
		--with-mpfr \
		--with-native-system-header-dir=/include \
		--with-newlib \
		--with-python-dir=share/gcc-$_target \
		--with-sysroot=/usr/$_target \
		--with-system-zlib \
		--without-libffi \
		--disable-__cxa_atexit \
		--disable-decimal-float \
		--disable-libgomp \
		--disable-libmpx \
		--disable-libmudflap \
		--disable-libquadmath \
		--disable-libquadmath-support \
		--disable-libssp \
		--disable-libstdcxx-pch \
		--disable-libstdcxx-verbose \
		--disable-nls \
		--disable-shared \
		--disable-threads \
		--disable-tls \
		--enable-gnu-indirect-function \
		--enable-languages=c,c++,d \
		--enable-lto \
		--enable-target-optspace
	make INHIBIT_LIBC_CFLAGS='-DUSE_TM_CLONE_REGISTRY=0'
}

build() {
	cd "$srcdir"/build-gcc
	export CFLAGS_FOR_TARGET='-g -Os -ffunction-sections -fdata-sections -mlongcalls'
	export CXXFLAGS_FOR_TARGET='-g -Os -ffunction-sections -fdata-sections -mlongcalls'
	_build_gcc

	# Build libstdc++ without exceptions support (the 'nano' variant)
	cd "$srcdir"/build-gcc-nano
	export CFLAGS_FOR_TARGET='-g -Os -ffunction-sections -fdata-sections -fno-exceptions -mlongcalls'
	export CXXFLAGS_FOR_TARGET='-g -Os -ffunction-sections -fdata-sections -fno-exceptions -mlongcalls'  
	_build_gcc
}

package() {
	cd "$srcdir"/build-gcc
	make DESTDIR="$pkgdir" install -j1

	cd "$srcdir"/build-gcc-nano
	make DESTDIR="$pkgdir.nano" install -j1
	# we need only libstdc nano files
	multilibs=( $("$pkgdir"/usr/bin/$_target-gcc -print-multi-lib 2>/dev/null) )
	for multilib in "${multilibs[@]}"; do
		dir="${multilib%%;*}"
		from_dir="$pkgdir".nano/usr/$_target/lib/"$dir"
		to_dir="$pkgdir"/usr/$_target/lib/"$dir"
		cp -f "$from_dir"/libstdc++.a "$to_dir"/libstdc++_nano.a
		cp -f "$from_dir"/libsupc++.a "$to_dir"/libsupc++_nano.a
	done

	# strip target binaries
	find "$pkgdir"/usr/lib/gcc/$_target/$pkgver "$pkgdir"/usr/$_target/lib -type f -and \( -name \*.a -or -name \*.o \) -exec $_target-objcopy -R .comment -R .note -R .debug_info -R .debug_aranges -R .debug_pubnames -R .debug_pubtypes -R .debug_abbrev -R .debug_line -R .debug_str -R .debug_ranges -R .debug_loc '{}' \;

	# strip host binaries
	find "$pkgdir"/usr/bin/ "$pkgdir"/usr/lib/gcc/$_target/$pkgver -type f -and \( -executable \) -exec strip '{}' \;

	# Remove files that conflict with host gcc package
	rm -r "$pkgdir"/usr/share/man/man7
	rm -r "$pkgdir"/usr/share/info
	rm "$pkgdir"/usr/lib/libcc1.*
}

# vim: ts=2 sw=0 noet
