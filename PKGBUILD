# Contributor: Olivier Mehani <olivier.mehani@nicta.com.au>
pkgname=oml2-git
pkgver=2.11.0
pkgrel=1
pkgdesc="OML is a measurement library that allows to define measurement points inside applications"
arch=(i686 x86_64)
url="http://oml.mytestbed.net/"
license=('custom:MIT-Nicta')
depends=('libxml2' 'sqlite3' 'popt' 'ruby>=1.8.7' 'postgresql-libs')
makedepends=('git' 'asciidoc')
optdepends=("postgresql: to use as a backend instead of sqlite3")
checkdepends=('check')
provides=(liboml liboml-git oml2)
conflicts=(liboml liboml-git oml2)
replaces=(liboml-git)
source=(
	oml2-server.conf
	oml2-server.logrotate
	oml2-server.rc
	oml2-server.service
)

backup=(etc/conf.d/oml2-server.conf)
install="oml2.install"

_gitroot="git://mytestbed.net/oml-staging.git"
_gitname="oml"
_gitbranch="master"
_builddir=${_gitname}-build

build() {
	cd "${srcdir}"
	if [ -d ${_gitname} ] ; then
		cd ${_gitname} && git pull origin
		msg "The local files are updated."
	else
		git clone ${_gitroot} ${_gitname}
	fi
	if [ ! -z ${_gitbranch} ]; then
		cd ${_gitname} && git checkout ${_gitbranch}
		cd ${_gitname} && git checkout master
	fi

	msg "Starting make..."

	rm -rf "${srcdir}/${_builddir}"
	git clone "${srcdir}/${_gitname}" "${srcdir}/${_builddir}"
	cd "${srcdir}/${_builddir}"
	./autogen.sh
	./configure --prefix=/usr --localstatedir=/var --enable-doc --disable-doxygen-doc --with-pgsql
	make || return 1
}

package() {
	cd "${srcdir}/${_builddir}"
	make DESTDIR="${pkgdir}/" install
	# XXX: We can only build the example application when we can find everything else
	make -C example/liboml2 \
		BINDIR="${pkgdir}/usr/bin" \
		SCAFFOLD="${pkgdir}/usr/bin/oml2-scaffold" \
		CFLAGS="-I${pkgdir}/usr/include" \
		LDFLAGS="-L${pkgdir}/usr/lib" \
		LIBS="-loml2 -locomm -lpopt -lm" \
		install
	mv ${pkgdir}/usr/bin/generator ${pkgdir}/usr/bin/oml2-generator
	install -D -m 0644 ${srcdir}/${_builddir}/COPYING \
		${pkgdir}/usr/share/licenses/${pkgname}/COPYING
	install -D -m 0755 ${startdir}/oml2-server.rc ${pkgdir}/etc/rc.d/oml2-server
	install -D -m 0644 ${startdir}/oml2-server.service ${pkgdir}/usr/lib/systemd/system/oml2-server.service
	install -D -m 0644 ${startdir}/oml2-server.conf ${pkgdir}/etc/conf.d/oml2-server.conf
	install -D -m 0644 ${startdir}/oml2-server.logrotate ${pkgdir}/etc/logrotate.d/oml2-server
}

check() {
	cd "${srcdir}/${_builddir}"
	make check
}
md5sums=('0fcaf350b4c5ac5bc8db2f17de11f9f9'
         '422a04d8bde681e43700d949750f2d99'
         '3d783cf3f03d339d5a1607325ee71b0f'
         '06f39c048d6c3752b167770d991f984c')
