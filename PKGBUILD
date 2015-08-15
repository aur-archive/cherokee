# $Id: PKGBUILD 63849 2012-02-06 06:17:28Z foutrelis $
# Maintainer:	Fernando Jiménez Solano <fjim@sdfeu.org>
# Contributor: Evangelos Foutras <evangelos@foutrelis.com>
# Contributor: Link Dupont <link@subpop.net>
# Contributor: Pierre Bourdin <pierre@pi3rrot.net>

pkgname=cherokee
pkgver=1.2.103.d021376
pkgrel=1
pkgdesc="A very fast, flexible and easy to configure Web Server"
arch=('i686' 'x86_64')
url="http://www.cherokee-project.com/"
license=('GPL2')
depends=('openssl' 'pcre')
makedepends=('python' 'gettext' 'libldap' 'pam' 'libmysqlclient'
			 'ffmpeg' 'geoip')
optdepends=('python: cherokee-admin (administrative web interface)'
			'libldap: ldap validator'
			'pam: pam validator'
			'libmysqlclient: mysql validator'
			'ffmpeg: Audio/Video streaming handler'
			'geoip: GeoIP rule module'
			'rrdtool: RRDtool based information collector')
backup=('etc/cherokee/cherokee.conf'
		'etc/logrotate.d/cherokee'
		'etc/pam.d/cherokee')
options=('!libtool')
source=(https://github.com/cherokee/webserver/archive/v1.2.103.d021376.zip
		cherokee.rc
		cherokee.logrotate
#		fix-ctk-path-handler-match.patch
#		cherokee-1.2.101-ffmpeg0.11.patch
		cherokee.service)
sha256sums=('5d8a5d99cb7382e2b851bbdc162bc46c192dc6b589ae34c34a6299ab419efe31'
			'4c06cebfab8b68edd4967c020bfb41b077cfff10d76596d1ed192d0b6cedbd86'
			'20e26d633f8c1cd90eb21f41dd163b73a83846e405b1ce995e072c4efefc522e'
#			'2bd05e0181024c9bd02d828e8329d4d96a779e4870b1fc4f18aa8667d8c6a630'
#			'6bcdcb8eaccb5516478a0c36960fbacc3d68f8bc326b9b526c388e0607a65116'
			'415a2e4cd7d04afe21e502dd0ad76301f85a7087cadbfdab5566bec469679a68')

_dirname=webserver

build() {
	cd "$srcdir/$_dirname-$pkgver"

	# Fix path matching bug in CTK apps (e.g. market)
	#patch -Np1 -i "$srcdir/fix-ctk-path-handler-match.patch"

	# Fix this bug : https://bugs.mageia.org/show_bug.cgi?id=6145
	#patch -Np1 -i "$srcdir/cherokee-1.2.101-ffmpeg0.11.patch"

	# Use subdirectory for logs
	sed -i -r 's|(%localstatedir%/log)|\1/cherokee|' cherokee.conf.sample.pre

	# Use Python 2 in cherokee-admin
	#sed -i 's/"python"/"python2"/' cherokee/main_admin.c

	./autogen.sh \
	--prefix=/usr \
	--sysconfdir=/etc \
	--localstatedir=/var \
	--disable-static \
	--with-wwwroot=/srv/http \
	--with-wwwuser=http \
	--with-wwwgroup=http \
	--with-python=python2 \
	--enable-os-string="Arch Linux"
	make
}

package() {
	cd "$srcdir/$_dirname-$pkgver"

	make -j1 DESTDIR="$pkgdir" install

	# PAM configuration file for cherokee
	install -D -m644 pam.d_cherokee "$pkgdir/etc/pam.d/$pkgname"

	# Fix ownership of /var/lib/cherokee/graphs
	chown -R http:http "$pkgdir/var/lib/$pkgname/graphs"

	# Use Python 2
	sed -i 's/env python$/&2/' \
	"$pkgdir/usr/share/cherokee/admin/"{server,upgrade_config}.py \
	"$pkgdir/usr/bin/"{CTK-run,cherokee-{admin-launcher,tweak}}
	sed -i -r "s/['\"]python/&2/g" \
	"$pkgdir/usr/share/cherokee/admin/wizards/django.py"

	# Compile Python scripts
	python2 -m compileall "$pkgdir"
	python2 -O -m compileall "$pkgdir"

	install -d -o http -g http "$pkgdir/var/log/$pkgname"
	install -D "$srcdir/$pkgname.rc" "$pkgdir/etc/rc.d/$pkgname"
	install -Dm644 "$srcdir/$pkgname.logrotate" "$pkgdir/etc/logrotate.d/$pkgname"
	install -Dm644 "$srcdir/$pkgname.service" "$pkgdir/usr/lib/systemd/system/$pkgname.service"

	# Cleanup
	rm -rf "$pkgdir/srv"
}

# vim:set ts=2 sw=2 et:
