# Maintainer: Sasha Gerrand <alpine-pkgs@sgerrand.com>

pkgname="glibc"
pkgver="2.32"
_pkgrel="2"
pkgrel="0"
pkgdesc="GNU C Library compatibility layer"
arch="all"
url="https://github.com/Prouser123/alpine-pkg-glibc"
license="LGPL"
source="x86_64.tar.gz::https://github.com/Prouser123/docker-glibc-multiarch-builder/releases/download/jcx-$pkgver-r$_pkgrel/glibc-bin-$pkgver-r$_pkgrel-x86_64-linux.tar.gz
pmmx.tar.gz::https://github.com/Prouser123/docker-glibc-multiarch-builder/releases/download/jcx-$pkgver-r$_pkgrel/glibc-bin-$pkgver-r$_pkgrel-i686-linux-gnu.tar.gz
ppc64.tar.gz::https://github.com/Prouser123/docker-glibc-multiarch-builder/releases/download/jcx-$pkgver-r$_pkgrel/glibc-bin-$pkgver-r$_pkgrel-powerpc64-linux-gnu.tar.gz
ppc.tar.gz::https://github.com/Prouser123/docker-glibc-multiarch-builder/releases/download/jcx-$pkgver-r$_pkgrel/glibc-bin-$pkgver-r$_pkgrel-powerpc-linux-gnu.tar.gz
aarch64.tar.gz::https://github.com/Prouser123/docker-glibc-multiarch-builder/releases/download/jcx-$pkgver-r$_pkgrel/glibc-bin-$pkgver-r$_pkgrel-aarch64-linux-gnu.tar.gz
armv7.tar.gz::https://github.com/Prouser123/docker-glibc-multiarch-builder/releases/download/jcx-$pkgver-r$_pkgrel/glibc-bin-$pkgver-r$_pkgrel-arm-linux-gnueabihf.tar.gz
nsswitch.conf
ld.so.conf"
#subpackages="$pkgname-bin $pkgname-dev $pkgname-i18n"
#triggers="$pkgname-bin.trigger=/lib:/usr/lib:/usr/glibc-compat/lib"

unpack() {
    
	local u
	verify || return 1
	initdcheck || return 1
	mkdir -p "$srcdir"
	local gunzip="$(command -v pigz || echo gunzip)"
	[ $gunzip = "/usr/bin/pigz" ] && gunzip="$gunzip -d"

    msg "Found arch: $CARCH"
    export FILE="$CARCH.tar.gz"
    msg "Attempting to use file $FILE"
    export FILE="$CARCH.tar.gz"

    msg "Unpacking $FILE..."
	$gunzip -c "$FILE" | tar -C "$srcdir" -f - -x || return 1

  # zlib download step (cheat and only do amd64 for now...)
  export ZLIB_URL='http://archive.ubuntu.com/ubuntu/pool/main/z/zlib/zlib1g_1.2.11.dfsg-2ubuntu1_amd64.deb'

  # Download zlib
  #wget -O zlib.deb ${ZLIB_URL}
	#ar vx zlib.deb

	mkdir -p $srcdir/zlib_tmp && cd $srcdir/zlib_tmp
	wget -O zlib.deb ${ZLIB_URL}
	ar vx zlib.deb
	tar xvf data.tar.xz
  #mv lib/$(ls lib)/* $srcdir/usr/glibc-compat/lib/
  msg "gay> debug> ls $srcdir/zlib_tmp"
  ls -lR $srcdir/zlib_tmp
}

package() {
  mkdir -p "$pkgdir/lib" "$pkgdir/lib64" "$pkgdir/usr/glibc-compat/lib/locale"  "$pkgdir"/usr/glibc-compat/lib64 "$pkgdir"/etc
  cp -a "$srcdir"/usr "$pkgdir" # copy main files

  # move ld.so conf and etc nswitch.conf
  cp "$srcdir"/ld.so.conf "$pkgdir"/usr/glibc-compat/etc/ld.so.conf
  cp "$srcdir"/nsswitch.conf "$pkgdir"/etc/nsswitch.conf


  # amd64 cheat for linking
  ln -s /usr/glibc-compat/lib/ld-linux-x86-64.so.2 $pkgdir/lib/ld-linux-x86-64.so.2
	ln -s /usr/glibc-compat/lib/ld-linux-x86-64.so.2 $pkgdir/lib64/ld-linux-x86-64.so.2
	ln -s /usr/glibc-compat/lib/ld-linux-x86-64.so.2 $pkgdir/usr/glibc-compat/lib64/ld-linux-x86-64.so.2
	ln -s /usr/glibc-compat/etc/ld.so.cache $pkgdir/etc/ld.so.cache
	ln -sfn /lib/libc.musl-x86_64.so.1 $pkgdir/usr/glibc-compat/lib

  # extract zlib
  msg "gay> moving zlib"
  cp $srcdir/zlib_tmp/lib/$(ls $srcdir/zlib_tmp/lib)/* "$pkgdir/usr/glibc-compat/lib/"
  msg "gay> zlib ls"
  ls -lR $pkgdir/usr/glibc-compat/lib/

  # strip
  # Run strip on stuff
	strip "$pkgdir"/usr/glibc-compat/sbin/**; \
	#strip "$pkgdir"/usr/glibc-compat/lib64/**; \
	strip "$pkgdir"/usr/glibc-compat/lib/** || echo 'Probably done with errors'; \
	strip "$pkgdir"/usr/glibc-compat/lib/*/* || echo 'Probably done with errors'; \
		
	# Remove unused files (https://github.com/sgerrand/alpine-pkg-glibc/blob/master/APKBUILD)
	rm "$pkgdir"/usr/glibc-compat/etc/rpc; \
	rm -rf "$pkgdir"/usr/glibc-compat/bin; \
	rm -rf "$pkgdir"/usr/glibc-compat/sbin; \
	rm -rf "$pkgdir"/usr/glibc-compat/lib/gconv; \
	rm -rf "$pkgdir"/usr/glibc-compat/lib/getconf; \
	rm -rf "$pkgdir"/usr/glibc-compat/lib/audit; \
	rm -rf "$pkgdir"/usr/glibc-compat/share; \
	rm -rf "$pkgdir"/usr/glibc-compat/var; \
		
	# Remove object files and static libraries. (https://blog.gilliard.lol/2018/11/05/alpine-jdk11-images.html)
	rm -rf "$pkgdir"/usr/glibc-compat/*.o; \
	rm -rf "$pkgdir"/usr/glibc-compat/*.a; \
	rm -rf "$pkgdir"/usr/glibc-compat/*/*.o; \
	rm -rf "$pkgdir"/usr/glibc-compat/*/*.a; \
  
  # STEP 1 | Copy glibc
  #mkdir -p "$pkgdir/lib" "$pkgdir/lib64" "$pkgdir/usr/glibc-compat/lib/locale"  "$pkgdir"/usr/glibc-compat/lib64 "$pkgdir"/etc
  #cp -a "$srcdir"/usr "$pkgdir"
  #cp "$srcdir"/ld.so.conf "$pkgdir"/usr/glibc-compat/etc/ld.so.conf
  #cp "$srcdir"/nsswitch.conf "$pkgdir"/etc/nsswitch.conf
  #rm "$pkgdir"/usr/glibc-compat/etc/rpc
  #rm -rf "$pkgdir"/usr/glibc-compat/bin
  #rm -rf "$pkgdir"/usr/glibc-compat/sbin
  #rm -rf "$pkgdir"/usr/glibc-compat/lib/gconv
  #rm -rf "$pkgdir"/usr/glibc-compat/lib/getconf
  #rm -rf "$pkgdir"/usr/glibc-compat/lib/audit
  #rm -rf "$pkgdir"/usr/glibc-compat/share
  #rm -rf "$pkgdir"/usr/glibc-compat/var
  #ln -s /usr/glibc-compat/lib/ld-linux-x86-64.so.2 ${pkgdir}/lib/ld-linux-x86-64.so.2
  #ln -s /usr/glibc-compat/lib/ld-linux-x86-64.so.2 ${pkgdir}/lib64/ld-linux-x86-64.so.2
  #ln -s /usr/glibc-compat/lib/ld-linux-x86-64.so.2 ${pkgdir}/usr/glibc-compat/lib64/ld-linux-x86-64.so.2
  #ln -s /usr/glibc-compat/etc/ld.so.cache ${pkgdir}/etc/ld.so.cache
	#ln -sfn /lib/libc.musl-x86_64.so.1 ${pkgdir}/usr/glibc-compat/lib
  # jcx hmm???? i dont think this has any effect
  # ln -s /usr/glibc-compat/lib/libc.so.6 ${pkgdir}/lib/libc.so.6
  
}

sha512sums="24b90355c78a10e8432c1707d6fed19a712a8ce14780ea2c77c89c2d4fd601542d24ad8144913aed984a2de130b223305b73a3c94bc78d7e7ab62faa8e1bfdbc  x86_64.tar.gz
c2b0954a685b3abbe0bd5ff52e3d7a6b3c2b434ce5a11942467ece1bf70cbe6e9a9016fd066029726efe383610940c7a525b91bfbda4217f3fbbbb16a737a74e  pmmx.tar.gz
143aaec5a726502b424b5fe363d9757662cc3ac878104189ecc864286e84b8df7eaf95af8705acdc2ba4b7b33e91cc44320185817b15bf19c884fc6ede72b70d  ppc64.tar.gz
ad726718f3238a38218057d2fa3626d2480ce96f7c02e4d43f868952fd95bf6fc024656b3c61fb100ffad890974573551683a7f685eabfbaef3da53501c9fb0c  ppc.tar.gz
c4716593ffcc2254ae42d1c3c35f38e9a475ac03a7eb7bc5dea146b864b9b63238c6452b865ecc3b1a365b00fd81ad51ffe5724fd7a71990cc319ddfcc08f497  aarch64.tar.gz
8804f956837fcf21409d9493decf7b30c97499f9dc9e21cca9462ffa6924979d28eec050ad608a0e6b6728b814d60b015db0e6093353d4b798c17e085e7c70e5  armv7.tar.gz
478bdd9f7da9e6453cca91ce0bd20eec031e7424e967696eb3947e3f21aa86067aaf614784b89a117279d8a939174498210eaaa2f277d3942d1ca7b4809d4b7e  nsswitch.conf
2912f254f8eceed1f384a1035ad0f42f5506c609ec08c361e2c0093506724a6114732db1c67171c8561f25893c0dd5c0c1d62e8a726712216d9b45973585c9f7  ld.so.conf"