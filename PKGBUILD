# AArch64 multi-platform
# Maintainer: Kevin Mihelich <kevin@archlinuxarm.org>

buildarch=8

pkgbase=linux-aarch64
_srcname=linux-5.1
_kernelname=${pkgbase#linux}
_desc="AArch64 multi-platform"
pkgver=5.1.7
pkgrel=1
arch=('aarch64')
url="http://www.kernel.org/"
license=('GPL2')
makedepends=('xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc' 'git' 'uboot-tools' 'vboot-utils' 'dtc')
options=('!strip')
source=("http://www.kernel.org/pub/linux/kernel/v5.x/${_srcname}.tar.xz"
        "http://www.kernel.org/pub/linux/kernel/v5.x/patch-${pkgver}.xz"
        '0001-net-smsc95xx-Allow-mac-address-to-be-set-as-a-parame.patch'
        '0002-arm64-dts-rockchip-disable-pwm0-on-rk3399-firefly.patch'
        '0003-arm64-dts-rockchip-add-usb3-controller-node-for-RK33.patch'
        '0004-arm64-dts-rockchip-enable-usb3-nodes-on-rk3328-rock6.patch'
        'config'
        'kernel.its'
        'kernel.keyblock'
        'kernel_data_key.vbprivk'
        'linux.preset'
        '60-linux.hook'
        '90-linux.hook')
md5sums=('15fbdff95ff98483069ac6e215b9f4f9'
         '85886aae897ffbe50810c4fc653479ee'
         '6af957dedb319049ddc903f5730d3ae8'
         'eee2e8059822d48f18086f4b36ed30bf'
         '910651d04b2f5fea9889df596b329035'
         'bd11e72a670134a38a75da3435bf8bcb'
         'becd30f71297bd680c299dedf30639ed'
         '7f1a96e24f5150f790df94398e9525a3'
         '61c5ff73c136ed07a7aadbf58db3d96a'
         '584777ae88bce2c5659960151b64c7d8'
         '41cb5fef62715ead2dd109dbea8413d6'
         'ce6c81ad1ad1f8b333fd6077d47abdaf'
         '3dc88030a8f2f5a5f97266d99b149f77')

prepare() {
sed -i s/'CONFIG_LOCALVERSION="-ARCH"'/'CONFIG_LOCALVERSION="-MANJARO-ARM"'/ config
sed -i s/'# CONFIG_NTFS_FS is not set'/'CONFIG_NTFS_FS=y\nCONFIG_NTFS_RW=y\nCONFIG_NTFS_DEBUG=n'/ config
sed -i s/'CONFIG_AUDIT=y'/'CONFIG_AUDIT=n'/ config
sed -i s/'CONFIG_HAVE_ARCH_AUDITSYSCALL=y'/'CONFIG_HAVE_ARCH_AUDITSYSCALL=n'/ config
sed -i s/'CONFIG_AUDITSYSCALL=y'/'CONFIG_AUDITSYSCALL=n'/ config
sed -i s/'CONFIG_AUDIT_WATCH=y'/'CONFIG_AUDIT_WATCH=n'/ config
sed -i s/'CONFIG_AUDIT_TREE=y'/'CONFIG_AUDIT_TREE=n'/ config
sed -i s/'CONFIG_AUDIT_GENERIC=y'/'CONFIG_AUDIT_GENERIC=n'/ config
sed -i s/'CONFIG_AUDIT_ARCH_COMPAT_GENERIC=y'/'CONFIG_AUDIT_ARCH_COMPAT_GENERIC=n'/ config
sed -i s/'CONFIG_MEDIA_SUPPORT=m'/'CONFIG_MEDIA_SUPPORT=y'/ config
sed -i s/'# CONFIG_MEDIA_CONTROLLER_REQUEST_API is not set'/'CONFIG_MEDIA_CONTROLLER_REQUEST_API=y'/ config
sed -i s/'CONFIG_VIDEO_SUNXI=y'/'CONFIG_VIDEO_SUNXI=y\nCONFIG_VIDEO_SUNXI_CEDRUS=y'/ config
  cd ${_srcname}

  # add upstream patch
  git apply --whitespace=nowarn ../patch-${pkgver}

  # ALARM patches
  git apply ../0001-net-smsc95xx-Allow-mac-address-to-be-set-as-a-parame.patch
  git apply ../0002-arm64-dts-rockchip-disable-pwm0-on-rk3399-firefly.patch
  git apply ../0003-arm64-dts-rockchip-add-usb3-controller-node-for-RK33.patch
  git apply ../0004-arm64-dts-rockchip-enable-usb3-nodes-on-rk3328-rock6.patch

  cat "${srcdir}/config" > ./.config

  # add pkgrel to extraversion
  sed -ri "s|^(EXTRAVERSION =)(.*)|\1 \2-${pkgrel}|" Makefile

  # don't run depmod on 'make install'. We'll do this ourselves in packaging
  sed -i '2iexit 0' scripts/depmod.sh
}

build() {
  cd ${_srcname}

  # get kernel version
  make prepare

  # load configuration
  # Configure the kernel. Replace the line below with one of your choice.
  #make menuconfig # CLI menu for configuration
  #make nconfig # new CLI menu for configuration
  #make xconfig # X-based configuration
  #make oldconfig # using old config from previous kernel version
  # ... or manually edit .config

  # Copy back our configuration (use with new kernel version)
  #cp ./.config ../${pkgbase}.config

  ####################
  # stop here
  # this is useful to configure the kernel
  #msg "Stopping build"
  #return 1
  ####################

  #yes "" | make config

  # build!
  unset LDFLAGS
  make ${MAKEFLAGS} Image Image.gz modules
  # Generate device tree blobs with symbols to support applying device tree overlays in U-Boot
  make ${MAKEFLAGS} DTC_FLAGS="-@" dtbs
}

_package() {
  pkgdesc="The Linux Kernel and modules - ${_desc}"
  depends=('coreutils' 'linux-firmware' 'kmod' 'mkinitcpio>=0.7')
  optdepends=('crda: to set the correct wireless channels of your country')
  provides=('kernel26' "linux=${pkgver}")
  replaces=('linux-armv8')
  conflicts=('linux')
  backup=("etc/mkinitcpio.d/${pkgbase}.preset")
  install=${pkgname}.install

  cd ${_srcname}

  KARCH=arm64

  # get kernel version
  _kernver="$(make kernelrelease)"
  _basekernel=${_kernver%%-*}
  _basekernel=${_basekernel%.*}

  mkdir -p "${pkgdir}"/{boot,usr/lib/modules}
  make INSTALL_MOD_PATH="${pkgdir}/usr" modules_install
  make INSTALL_DTBS_PATH="${pkgdir}/boot/dtbs" dtbs_install
  cp arch/$KARCH/boot/Image{,.gz} "${pkgdir}/boot"

  # make room for external modules
  local _extramodules="extramodules-${_basekernel}${_kernelname}"
  ln -s "../${_extramodules}" "${pkgdir}/usr/lib/modules/${_kernver}/extramodules"

  # add real version for building modules and running depmod from hook
  echo "${_kernver}" |
    install -Dm644 /dev/stdin "${pkgdir}/usr/lib/modules/${_extramodules}/version"

  # remove build and source links
  rm "${pkgdir}"/usr/lib/modules/${_kernver}/{source,build}

  # now we call depmod...
  depmod -b "${pkgdir}/usr" -F System.map "${_kernver}"

  # add vmlinux
  install -Dt "${pkgdir}/usr/lib/modules/${_kernver}/build" -m644 vmlinux

  # sed expression for following substitutions
  local _subst="
    s|%PKGBASE%|${pkgbase}|g
    s|%KERNVER%|${_kernver}|g
    s|%EXTRAMODULES%|${_extramodules}|g
  "

  # install mkinitcpio preset file
  sed "${_subst}" ../linux.preset |
    install -Dm644 /dev/stdin "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

  # install pacman hooks
  sed "${_subst}" ../60-linux.hook |
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/60-${pkgbase}.hook"
  sed "${_subst}" ../90-linux.hook |
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/90-${pkgbase}.hook"
}

_package-headers() {
  pkgdesc="Header files and scripts for building modules for linux kernel - ${_desc}"
  provides=("linux-headers=${pkgver}")
  conflicts=('linux-headers')

  cd ${_srcname}
  local _builddir="${pkgdir}/usr/lib/modules/${_kernver}/build"

  install -Dt "${_builddir}" -m644 Makefile .config Module.symvers
  install -Dt "${_builddir}/kernel" -m644 kernel/Makefile

  mkdir "${_builddir}/.tmp_versions"

  cp -t "${_builddir}" -a include scripts

  install -Dt "${_builddir}/arch/${KARCH}" -m644 arch/${KARCH}/Makefile
  install -Dt "${_builddir}/arch/${KARCH}/kernel" -m644 arch/${KARCH}/kernel/asm-offsets.s arch/$KARCH/kernel/module.lds

  cp -t "${_builddir}/arch/${KARCH}" -a arch/${KARCH}/include
  mkdir -p "${_builddir}/arch/arm"
  cp -t "${_builddir}/arch/arm" -a arch/arm/include

  install -Dt "${_builddir}/drivers/md" -m644 drivers/md/*.h
  install -Dt "${_builddir}/net/mac80211" -m644 net/mac80211/*.h

  # http://bugs.archlinux.org/task/13146
  install -Dt "${_builddir}/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # http://bugs.archlinux.org/task/20402
  install -Dt "${_builddir}/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "${_builddir}/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "${_builddir}/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  # add xfs and shmem for aufs building
  mkdir -p "${_builddir}"/{fs/xfs,mm}

  # copy in Kconfig files
  find . -name Kconfig\* -exec install -Dm644 {} "${_builddir}/{}" \;

  # remove unneeded architectures
  local _arch
  for _arch in "${_builddir}"/arch/*/; do
    [[ ${_arch} == */${KARCH}/ || ${_arch} == */arm/ ]] && continue
    rm -r "${_arch}"
  done

  # remove files already in linux-docs package
  rm -r "${_builddir}/Documentation"

  # remove now broken symlinks
  find -L "${_builddir}" -type l -printf 'Removing %P\n' -delete

  # Fix permissions
  chmod -R u=rwX,go=rX "${_builddir}"

  # strip scripts directory
  local _binary _strip
  while read -rd '' _binary; do
    case "$(file -bi "${_binary}")" in
      *application/x-sharedlib*)  _strip="${STRIP_SHARED}"   ;; # Libraries (.so)
      *application/x-archive*)    _strip="${STRIP_STATIC}"   ;; # Libraries (.a)
      *application/x-executable*) _strip="${STRIP_BINARIES}" ;; # Binaries
      *) continue ;;
    esac
    /usr/bin/strip ${_strip} "${_binary}"
  done < <(find "${_builddir}/scripts" -type f -perm -u+w -print0 2>/dev/null)
}

_package-chromebook() {
  pkgdesc="The Linux Kernel - ${_desc} - Chromebooks"
  depends=('linux-aarch64')
  conflicts=('linux-aarch64-rc-chromebook')
  install=${pkgname}.install

  cd ${_srcname}

  mkdir -p "${pkgdir}/boot"
  cp ../kernel.its .

  mkimage -D "-I dts -O dtb -p 2048" -f kernel.its vmlinux.uimg
  dd if=/dev/zero of=bootloader.bin bs=512 count=1
  echo "console=tty0 console=ttyS2,115200n8 earlyprintk=ttyS2,115200n8 init=/sbin/init root=PARTUUID=%U/PARTNROFF=1 rootwait rw noinitrd" > cmdline
  vbutil_kernel \
    --pack vmlinux.kpart \
    --version 1 \
    --vmlinuz vmlinux.uimg \
    --arch aarch64 \
    --keyblock ../kernel.keyblock \
    --signprivate ../kernel_data_key.vbprivk \
    --config cmdline \
    --bootloader bootloader.bin

  cp vmlinux.kpart "${pkgdir}/boot"
}

pkgname=("${pkgbase}" "${pkgbase}-headers" "${pkgbase}-chromebook")
for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    _package${_p#${pkgbase}}
  }"
done
