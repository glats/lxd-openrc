# Maintainer: Juan Cuzmar <juan.cuzmar.s@gmail.com>
# based on lxd package https://aur.archlinux.org/packages/lxd/

pkgname=lxd-openrc
_pkgname=lxd
pkgver=3.18
pkgrel=1
pkgdesc="Fast, dense, secure container hypervisor and a new user experience for LXC with OpenRC support"
arch=('x86_64')
url="https://github.com/lxc/lxd"
license=('APACHE')
conflicts=('lxd-git' 'lxd-lts' 'lxd')
provides=('lxd')
depends=('lxc' 'lxcfs' 'squashfs-tools' 'dnsmasq' 'libuv' 'openrc' 'libcgroup-openrc')
makedepends=('go' 'git' 'tcl' 'patchelf')
optdepends=(
    'lvm2: for lvm2 support'
    'thin-provisioning-tools: for thin provisioning support'
    'btrfs-progs: for btrfs storage driver support'
    'ceph: for ceph storage driver support'
)
options=('!strip' '!emptydirs')

_lxd=github.com/lxc/lxd

source=(
    "https://${_lxd}/releases/download/${_pkgname}-${pkgver}/${_pkgname}-${pkgver}.tar.gz"
    "lxd.initd"
    "lxd.confd"
)

md5sums=('4acf701d5903c905b931d800bfbfc6a1'
         '8d463d636e7ef5fadcb0a11e7324f493'
         '9f39af2bcf31dccf356bdddc42b3e2ad')

build() {
  export GOPATH="${srcdir}/${_pkgname}-${pkgver}/_dist"
  cd "${GOPATH}/src/${_lxd}"
  make deps
  export CGO_CFLAGS="-I${GOPATH}/deps/sqlite/ -I${GOPATH}/deps/libco/ -I${GOPATH}/deps/raft/include/ -I${GOPATH}/deps/dqlite/include/"
  export CGO_LDFLAGS="-L${GOPATH}/deps/sqlite/.libs/ -L${GOPATH}/deps/libco/ -L${GOPATH}/deps/raft/.libs -L${GOPATH}/deps/dqlite/.libs/"
  export LD_LIBRARY_PATH="${GOPATH}/deps/sqlite/.libs/:${GOPATH}/deps/libco/:${GOPATH}/deps/raft/.libs/:${GOPATH}/deps/dqlite/.libs/"
  make
}

package() {
  go_path="${srcdir}/${_pkgname}-${pkgver}/_dist"
  go_bin_dir="${go_path}/bin"
  go_deps_dir="${go_path}/deps"
  install=lxd.install
  mkdir -p "${pkgdir}/usr/bin"
  mkdir -p "${pkgdir}/usr/lib/lxd"
  mkdir -p "${pkgdir}/usr/share/doc/lxd"
  mkdir -p "${pkgdir}/usr/share/bash-completion/completions"
  install -p -m755 "${go_bin_dir}/"* "${pkgdir}/usr/bin"
  patchelf --set-rpath "/usr/lib/lxd" "${pkgdir}/usr/bin/lxd"
  cp --no-dereference --preserve=timestamps \
    "${go_deps_dir}/sqlite/.libs/"libsqlite3.so* \
    "${go_deps_dir}/libco/"libco.so* \
    "${go_deps_dir}/raft/.libs/"libraft.so* \
    "${go_deps_dir}/dqlite/.libs/"libdqlite.so* \
    "${pkgdir}/usr/lib/lxd"
  patchelf --set-rpath "/usr/lib/lxd" "${pkgdir}/usr/lib/lxd/libdqlite.so"

  # Package license
  install -Dm644 "${go_path}/src/${_lxd}/COPYING"  "${pkgdir}/usr/share/licenses/${_pkgname}/LICENCE"

  # service/conf files
  install -D -m755 "${srcdir}/lxd.initd" \
    "${pkgdir}/etc/init.d/lxd"
  install -D -m644 "${srcdir}/lxd.confd" \
    "${pkgdir}/etc/conf.d/lxd"

  # documentation
  install -D -m644 "${go_path}/src/${_lxd}/doc/"* \
    "${pkgdir}/usr/share/doc/lxd/"

  # Bash completions
  install -p -m644 "${go_path}/src/${_lxd}/scripts/bash/lxd-client" \
    "${pkgdir}/usr/share/bash-completion/completions/lxd"

  # support systemd based containers on OpenRC hosts
  mkdir -p "${pkgdir}/sys/fs/cgroup/systemd"
}

# vim:set ts=2 sw=2 et:
