# Contributor: Carlo Landmeter
# Maintainer:Laurelai Bailey
pkgname=libc6
pkgver=2.14
pkgrel=3
pkgdesc="Embedded GLIBC is a variant of the GNU C Library that is designed to work well on embedded systems"
url="http://www.eglibc.org"
license="GPL"
depends=""
depends_dev="linux-headers=>2.6.32"
replaces=uclibc 
makedepends="gawk"
options="!strip"
triggers="libc6-utils.trigger=/lib:/usr/lib"
subpackages="$pkgname-dev $pkgname-utils $pkgname-gconv $pkgname-locales $pkgname-nscd libthread_db"
source="http://alpine.oneechan.org/eglibc-2.14-r17321.tar.bz2
        nscd.initd ld.so.conf"

arch="all"

_builddir="$srcdir"/libc

prepare() {
	cd "$_builddir"

	# http://sources.redhat.com/bugzilla/show_bug.cgi?id=4781
      #	patch -Np1 -i ${srcdir}/glibc-2.10-bz4781.patch

	# http://sources.redhat.com/bugzilla/show_bug.cgi?id=411
	# http://sourceware.org/ml/libc-alpha/2009-07/msg00072.html
      #	patch -Np1 -i ${srcdir}/glibc-__i686.patch
        
      # patch -Np1 -i ${srcdir}/eglibc-make-382.patch
	# Don not build timezones
	sed -i 's/timezone rt/rt/' Makeconfig

	echo "slibdir=/lib" >> configparms
	
	# we need to build in a seperate dir
	mkdir "$_builddir"/libc6-build
}

build() {
	cd "$_builddir"/libc6-build

	export CFLAGS="$CFLAGS -fno-stack-protector"

	../configure --prefix=/usr \
		--sysconfdir=/etc \
		--mandir=/usr/share/man \
		--infodir=/usr/share/info \
		--mandir=/usr/share/info \
		--infodir=/usr/share/info \
		--libdir=/usr/lib \
		--libexecdir=/usr/lib \
		--with-headers=/usr/include \
		--enable-kernel=2.6.32 \
		--enable-add-ons=nptl,libidn \
		--disable-profile \
		--enable-bind-now \
		--with-tls \
		--with-__thread \
		--without-cvs \
		--without-gd


	make || return 1

}

package() {
	cd "$_builddir"/libc6-build
	
	install -dm755 "$pkgdir"/etc
	install -c -m644 ld.so.conf "$pkgdir"/etc/ld.so.conf
	
	make -j1 install_root="$pkgdir" install
	
	# provided by kernel-headers
  	rm "$pkgdir"/usr/include/scsi/scsi.h
	
	rm "$pkgdir"/etc/ld.so.conf

	# strip all
	for i in ldconfig sln gencat getconf getent iconv locale localedef \
		pcprofiledump rpcgen sprof lddlibc4 iconvconfig nscd rpcinfo; do
			find "$pkgdir" -type f -name "$i" -exec strip --strip-all '{}' \;
	done
	strip --strip-all "$pkgdir"/usr/lib/gconv/*.so
	
	# strip debug
	for i in ld-2.12.1.so libpthread-2.12.1.so libthread_db-1.0.so; do
		strip --strip-debug "$pkgdir"/lib/"$i"
	done
	strip --strip-debug "$pkgdir"/usr/lib/*.a
	
	# strip uneeded
	for i in libanl-2.12.1.so libBrokenLocale-2.12.1.so libc-2.12.1.so \
		libcidn-2.12.1.so libcrypt-2.12.1.so libnss_*-2.12.1.so \
		libdl-2.12.so libm-2.12.so libnsl-2.12.so libresolv-2.12.so \
		librt-2.12.so libutil-2.12.so libmemusage.so libpcprofile.so \
		libSegFault.so pt_chown; do
			find "$pkgdir" -type f -name "$i" -exec strip --strip-unneeded '{}' \;
	done
	strip --strip-unneeded "$pkgdir"/usr/lib/gconv/*.so
	
}	

gconv() {
	pkgdesc="gconv character modules"
	mkdir -p "$subpkgdir"/usr/lib/
	mv "$pkgdir"/usr/lib/gconv "$subpkgdir"/usr/lib/
}

locales() {
	pkgdesc="Common files for locale support"
	mkdir -p "$subpkgdir"/usr/share
	mv "$pkgdir"/usr/share/* "$subpkgdir"/usr/share/
}

nscd() {
	pkgdesc="libc6 name service cache daemon"
        mkdir -p "$subpkgdir"/var/db/nscd "$subpkgdir"/var/run/nscd
	install -Dm 755 "$srcdir"/nscd.initd "$subpkgdir"/etc/init.d/nscd
	install -Dm 644 "$srcdir"/libc/nscd/nscd.conf "$subpkgdir"/etc/nscd.conf
	mkdir -p "$subpkgdir"/usr/sbin
	mv "$pkgdir"/usr/sbin/nscd "$subpkgdir"/usr/sbin/
}


utils() {
        pkgdesc="libc6 utility programs"
        replaces="uclibc-utils libiconv"
        mkdir -p "$subpkgdir"/usr/bin "$subpkgdir"/sbin
        mv "$pkgdir"/sbin/* "$subpkgdir"/sbin/
        mv "$pkgdir"/usr/bin/* "$subpkgdir"/usr/bin/
}


dev() {
        default_dev
        replaces="uclibc-dev linux-headers fts-dev"
        mkdir -p "$subpkgdir"/usr/lib
        mv "$pkgdir"/usr/lib/*.so "$subpkgdir"/usr/lib/
}


libthread_db() {
        pkgdesc="libc6 thread debugging library"
        mkdir -p "$subpkgdir"/lib
        mv "$pkgdir"/lib/libthread_db* "$subpkgdir"/lib/
}

md5sums="9d1efab450cceb5e946b433df5a6385b  eglibc-2.14-r17321.tar.bz2
0cb7dd4e6ce03c4d9b68bff7da52cef0  nscd.initd
a9870f383320e52490becb5d530fb076  ld.so.conf"
