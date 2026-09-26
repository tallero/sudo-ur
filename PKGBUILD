# Maintainer: Lukas Fleischer <lfleischer@archlinux.org>
# Contributor: Evangelos Foutras <foutrelis@archlinux.org>
# Contributor: Allan McRae <allan@archlinux.org>
# Contributor: Tom Newsom <Jeepster@gmx.co.uk>

if [[ ! -v "_os" ]]; then
  _os="$(
    uname \
      -o)"
fi
_pkg=sudo
if [[ ! -v "_android" ]]; then
  _android="false"
  if [[ "${_os}" == "Android" ]]; then
    _android="true"
  fi
fi
if [[ ! -v "_gnu" ]]; then
  _gnu="true"
  if [[ "${_android}" == "true" ]]; then
    _gnu="false"
  fi
fi
if [[ ! -v "_ns" ]]; then
  _ns="${_pkg}"
  if [[ "${_android}" == "true" ]]; then
    _ns="agnosticapollo"
    _ns="themartiancompany"
  fi
  _ns="themartiancompany"
fi
if [[ ! -v "_git_service" ]]; then
  _git_service="github"
fi
if [[ ! -v "_git" ]]; then
  _git="false"
fi
if [[ ! -v "_release" ]]; then
  _release="false"
fi
if [[ ! -v "_http" ]]; then
  if [[ "${_git}" == "true" ]]; then
    _http="https://${_git_service}.com"
  elif [[ "${_git}" == "true" ]]; then
    _http="https://${_git_service}.com"
    if [[ "${_release}" == "true" ]]; then
      _http="https://www.${_pkg}.ws"
    fi
  fi
fi
pkgbase="${_pkg}"
pkgname=(
)
if [[ "${_android}" == "true" ]]; then
  pkgname+=(
    "${_pkg}-android"
  )
fi
if [[ "${_gnu}" == "true" ]]; then
  pkgname+=(
    "${_pkg}-gnu"
  )
fi
_gnu_ver=1.9.17
_android_ver=1.2.0
pkgver="1000000.g${_gnu_ver}.a${_android_ver}"
pkgrel=1
_pkgdesc=(
  "Give certain users the"
  "ability to run some commands as root."
)
pkgdesc="${_pkgdesc[*]}"
arch=()
if [[ "${_gnu}" == "true" ]]; then
  arch+=(
    "aarch64"
    "arm"
    "armv6l"
    "armv7l"
    "armv8l"
    "i686"
    "mips"
    "pentium4"
    "powerpc"
    "x86_64"
  )
fi
_gnu_url="https://www.${_pkg}.ws/${_pkg}"
url="https://www.${_git_service}.com/${_ns}/${_pkg}"
license=(
  'custom'
)
depends=(
  'glibc'
  'openssl'
  'pam'
  'libldap'
  'zlib'
)
backup=(
  'etc/pam.d/sudo'
  'etc/sudo.conf'
  'etc/sudo_logsrvd.conf'
  'etc/sudoers'
)
source=(
  "${_gnu_url}/${_pkg}/dist/${_pkg}-${_sudover}.tar.gz{,.sig}
  "sudo_logsrvd.service
  "sudo.pam)
sha256sums=('4a38a1ab3adb1199257edc2a7c4a2bd714665eb605b04368843b06dada2cfcfb'
            'SKIP'
            'bd4bc2f5d85cbe14d7e7acc5008cb4fe62c38de7d42dc6876c87bfaa273c0a6e'
            '7ec1c668c10e0f83d00e25f336872212fe04ce2c2563e1d661d34d28852f4649')
validpgpkeys=('59D1E9CCBA2B376704FDD35BA9F4C021CEA470FB')

build() {
  cd "${pkgname}-${_sudover}"

  ./configure \
    --prefix=/usr \
    --sbindir=/usr/bin \
    --libexecdir=/usr/lib \
    --with-rundir=/run/sudo \
    --with-vardir=/var/db/sudo \
    --with-logfac=auth \
    --enable-tmpfiles.d \
    --with-pam \
    --with-sssd \
    --with-ldap \
    --with-ldap-conf-file=/etc/openldap/ldap.conf \
    --with-env-editor \
    --with-passprompt="[sudo] password for %p: " \
    --with-secure-path-value=/usr/local/sbin:/usr/local/bin:/usr/bin \
    --with-all-insults

  # Prevent excessive overlinking due to libtool; for details, please refer to
  # https://gitlab.archlinux.org/archlinux/packaging/packages/sudo/-/merge_requests/3.
  sed -i -e 's/ -shared / -Wl,-O1,--as-needed\0/g' libtool

  make
}

check() {
  make -C "${pkgname}-${_sudover}" check
}

package() {
  depends+=('libcrypto.so' 'libssl.so')

  cd "${pkgname}-${_sudover}"

  make DESTDIR="$pkgdir" install

  # sudo_logsrvd service file (taken from sudo-logsrvd-1.9.0-1.el8.x86_64.rpm)
  install -Dm644 -t "$pkgdir/usr/lib/systemd/system" ../sudo_logsrvd.service

  # Remove sudoers.dist; not needed since pacman manages updates to sudoers
  rm "$pkgdir/etc/sudoers.dist"

  # Remove /run/sudo directory; we create it using systemd-tmpfiles
  rmdir "$pkgdir/run/sudo"
  rmdir "$pkgdir/run"

  install -Dm644 "$srcdir/sudo.pam" "$pkgdir/etc/pam.d/sudo"

  install -Dm644 LICENSE.md -t "$pkgdir/usr/share/licenses/sudo"
}

# vim:set ts=2 sw=2 et:
