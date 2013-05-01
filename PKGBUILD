# $Id: PKGBUILD 170041 2012-10-31 07:25:00Z tpowa $
# Maintainer: Paul Mattal <paul@archlinux.org>

pkgbase=lirc
pkgname=('lirc' 'lirc-utils')
pkgver=0.9.0
pkgrel=45
epoch=2
_extramodules=extramodules-3.8-ARCH
arch=('i686' 'x86_64')
url="http://www.lirc.org/"
license=('GPL')
### NOTICE don't forget to bump version in depends in package_lirc
makedepends=('help2man' 'linux-headers>=3.6' 'alsa-lib' 'libx11' 'libftdi' 'libirman' 'python2')
options=('!makeflags' '!strip')
source=(http://prdownloads.sourceforge.net/${pkgbase}/${pkgbase}-${pkgver}.tar.bz2
        lirc_wpc8769l.patch
        lircd-handle-large-config.patch
        lirc_atiusb-kfifo.patch
        kernel-2.6.39.patch
        lircd lircmd lirc.logrotate lircd.conf irexec.conf irexecd
        lirc.service lircm.service irexec.service
        lirc.tmpfiles
        linux-3.8.patch
        lirc-0.9.0_ya_usbirv3-3.diff.tar.gz
       )

build() {
  _kernver="$(cat /usr/lib/modules/${_extramodules}/version)"
  cd "${srcdir}/lirc-${pkgver}"
  patch -Np1 -i "${srcdir}/lirc_wpc8769l.patch"
  patch -Np1 -i "${srcdir}/lircd-handle-large-config.patch"
  patch -Np1 -i "${srcdir}/lirc_atiusb-kfifo.patch"
  patch -Np1 -i "${srcdir}/kernel-2.6.39.patch"
  patch -Np1 -i "${srcdir}/linux-3.8.patch"
  patch -Np1 -i "${srcdir}/lirc-0.9.0_ya_usbirv3-3.diff"

  sed -i '/AC_PATH_XTRA/d' configure.ac
  sed -i 's/AM_CONFIG_HEADER/AC_CONFIG_HEADER/' configure.ac
  sed -e 's/@X_CFLAGS@//g' \
      -e 's/@X_LIBS@//g' \
      -e 's/@X_PRE_LIBS@//g' \
      -e 's/@X_EXTRA_LIBS@//g' -i Makefile.am tools/Makefile.am
  libtoolize
  autoreconf

  PYTHON=python2 ./configure --enable-sandboxed --prefix=/usr \
      --with-driver=all --with-kerneldir=/usr/src/linux-${_kernver}/ \
      --with-moduledir=/usr/lib/modules/${_kernver}/kernel/drivers/misc \
      --with-transmitter

  # Remove drivers already in kernel
  sed -e "s:lirc_dev::" -e "s:lirc_bt829::" -e "s:lirc_igorplugusb::" \
      -e "s:lirc_imon::" -e "s:lirc_parallel::" -e "s:lirc_sasem::" \
      -e "s:lirc_serial::" -e "s:lirc_sir::" -e "s:lirc_ttusbir::" \
      -i Makefile drivers/Makefile drivers/*/Makefile tools/Makefile 
  make
}

package_lirc() {
  pkgdesc="Linux Infrared Remote Control kernel modules for stock arch kernel"
  depends=('lirc-utils' 'linux>=3.6' 'linux<3.7')
  replaces=('lirc+pctv')
  install=lirc.install

  cd "${srcdir}/lirc-${pkgver}/drivers"
  make DESTDIR="${pkgdir}" moduledir="/usr/lib/modules/${_extramodules}" install

  # set the kernel we've built for inside the install script
  sed -i -e "s/EXTRAMODULES=.*/EXTRAMODULES=${_extramodules}/g" "${startdir}/lirc.install"
  # gzip -9 modules
  find "${pkgdir}" -name '*.ko' -exec gzip -9 {} \;
}

package_lirc-utils() {
  pkgdesc="Linux Infrared Remote Control utils"
  depends=('alsa-lib' 'libx11' 'libftdi' 'libirman')
  optdepends=('python2: pronto2lirc utility')
  options=('strip' '!libtool')
  backup=('etc/conf.d/lircd.conf' 'etc/conf.d/irexec.conf')
  install=lirc-utils.install

  cd "${srcdir}/lirc-${pkgver}"
  make DESTDIR="${pkgdir}" install
  install -d "${pkgdir}/usr/share/lirc" "${pkgdir}/etc/rc.d"
  cp "${srcdir}"/{lircd,lircmd,irexecd} "${pkgdir}/etc/rc.d"
  install -D -m644 "${srcdir}"/lirc.service "${pkgdir}"/usr/lib/systemd/system/lirc.service
  install -D -m644 "${srcdir}"/lircm.service "${pkgdir}"/usr/lib/systemd/system/lircm.service
  install -D -m644 "${srcdir}"/irexec.service "${pkgdir}"/usr/lib/systemd/system/irexec.service
  install -D -m644 "${srcdir}"/lirc.tmpfiles "${pkgdir}"/usr/lib/tmpfiles.d/lirc.conf
  cp -rp remotes "${pkgdir}/usr/share/lirc"
  chmod -R go-w "${pkgdir}/usr/share/lirc/"

  # install the logrotate config
  install -Dm644 "${srcdir}/lirc.logrotate" "${pkgdir}/etc/logrotate.d/lirc"
    
  # install conf.d file
  install -Dm644 "${srcdir}/lircd.conf" "${pkgdir}/etc/conf.d/lircd.conf"

  # install conf.d file
  install -Dm644 "${srcdir}/irexec.conf" "${pkgdir}/etc/conf.d/irexec.conf"

  install -d "${pkgdir}/etc/lirc"
  
  # remove built modules
  rm -r "${pkgdir}/usr/lib/modules"
}

md5sums=('b232aef26f23fe33ea8305d276637086'
         '1cce37e18e3f6f46044abab29016d18f'
         'b70cc9640505205446ec47b7d4779f38'
         '1f8b104a2365d9f93404b143f499059b'
         '087a7d102e1c96bf1179f38db2b0b237'
         '8d0e238dc0eda95e340fe570605da492'
         '85f7fdac55e5256967241864049bf5e9'
         '3deb02604b37811d41816e9b4385fcc3'
         '5b1f8c9cd788a39a6283f93302ce5c6e'
         'f0c0ac930326168035f0c8e24357ae55'
         '69d099e6deedfa3c1ee2b6e82d9b8bfb'
         'dab8a73bcc5fd5479d8750493d8d97dc'
         'c2e20fe68b034df752dba2773db16ebe'
         '07131d117fcfe9dcd50c453c3a5e9531'
         'febf25c154a7d36f01159e84f26c2d9a'
         '9ee196bd03ea44af5a752fb0cc6ca96a'
         '8f2b480b0392b35e7aff413dc491a228')
