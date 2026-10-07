# Maintainer: Copper0SO4
pkgname=mctier-linux-web-git
pkgver=3.9.0.r383.gfe617bc
pkgrel=1
pkgdesc='MCTier Linux Web: virtual networking, chat and browser-based voice/screen sharing'
arch=('x86_64')
url='https://github.com/Copper0SO4/MCTier_Linux_Web'
license=('custom:MCTier-NonCommercial' 'LGPL-3.0-only')
depends=('bash' 'glibc' 'gcc-libs' 'openssl' 'dbus' 'libcap' 'polkit' 'curl' 'xdg-utils' 'coreutils' 'gawk')
makedepends=('git' 'nodejs>=20' 'npm' 'rust>=1.90')
optdepends=(
  'chromium: recommended browser for voice and screen sharing'
  'gnome-keyring: Secret Service provider for encrypted passwords'
  'ufw: optional firewall rule management'
  'firewalld: optional firewall rule management'
)
provides=('mctier-linux-web')
conflicts=('mctier-linux-web')
# EasyTier must retain its upstream hash; makepkg stripping would invalidate it.
options=('!strip' '!debug')
install=mctier-linux-web.install
source=(
  'MCTier::git+https://github.com/Copper0SO4/MCTier_Linux_Web.git#branch=master'
  'https://github.com/EasyTier/EasyTier/releases/download/v2.5.0/easytier-linux-x86_64-v2.5.0.zip'
  'easytier-LICENSE::https://raw.githubusercontent.com/EasyTier/EasyTier/v2.5.0/LICENSE'
  'mctier.sh'
  'README.md'
)
noextract=('easytier-linux-x86_64-v2.5.0.zip')
sha256sums=(
  'SKIP'
  'c715d62ffcdad2578bc5d743bdfbcfe02cda9c12f80bd1bad3b36d4d7fc0eb8b'
  'e3a994d82e644b03a792a930f574002658412f62407f5fee083f2555c5f23118'
  'ba3b7137f5544d63e96c1e4458bb584c4605b29227969c1a54e30a75388b569e'
  '7b175429d1d1c0baecff7903ae88d2b9e54b5f58069d967c8812193733e16bdb'
)

pkgver() {
  cd "$srcdir/MCTier"
  local release revision commit
  release=$(awk -F '"' '/^version = / {print $2; exit}' MCTier-Linux-Web/server/Cargo.toml)
  [[ $release =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]] || return 1
  revision=$(git rev-list --count HEAD)
  commit=$(git rev-parse --short=7 HEAD)
  printf '%s.r%s.g%s' "$release" "$revision" "$commit"
}

_check_rust_toolchain() {
  local details
  if ! details=$(rustc -vV 2>&1); then
    printf '%s\n' \
      '错误：Rust 编译器自身无法启动，还未开始编译 MCTier。' \
      "当前 rustc：$(command -v rustc || true)" \
      "$details" \
      '请先修复系统 Rust/LLVM，或将经过验证的独立 Rust 工具链 bin 目录置于 PATH 最前面。' \
      '排障说明见本仓库 README 的“Rust/LLVM 工具链错误”。' >&2
    return 1
  fi
  if ! cargo --version; then
    printf '%s\n' '错误：Cargo 无法启动，请检查 PATH 中的 Rust 工具链。' >&2
    return 1
  fi
  printf '%s\n' "$details"
}

prepare() {
  _check_rust_toolchain || return 1
  cd "$srcdir/MCTier"
  local target=MCTier-Linux-Web/resources/binaries
  mkdir -p "$target"
  bsdtar -xf "$srcdir/easytier-linux-x86_64-v2.5.0.zip" -C "$target" --strip-components 1 \
    easytier-linux-x86_64/easytier-core easytier-linux-x86_64/easytier-cli
  printf '%s  %s\n' \
    f1bd60be7a50da84f50732ed4b826b70284c84f05dadbd3fe448429dfe184322 "$target/easytier-core" \
    e339aea31943f0c5ced2a5a6ecdd675da3bb25843cf847107744e656f8200838 "$target/easytier-cli" | sha256sum -c -
  chmod 755 "$target/easytier-core" "$target/easytier-cli"
  HUSKY=0 npm ci --no-audit --no-fund
  cargo fetch --locked --manifest-path MCTier-Linux-Web/server/Cargo.toml
}

_portable_arch_flags() {
  sed -E \
    -e 's/(^|[[:space:]])-march=[^[:space:]]+/\1-march=x86-64/g' \
    -e 's/(^|[[:space:]])-mtune=[^[:space:]]+/\1-mtune=generic/g'
}

build() {
  cd "$srcdir/MCTier"
  # Arch hosts may configure native CPU flags globally. Keep every compiled
  # component on the x86-64 baseline for older CPUs and virtual machines.
  local portable_cflags portable_cxxflags
  portable_cflags=$(printf '%s\n' "${CFLAGS:-}" | _portable_arch_flags)
  portable_cxxflags=$(printf '%s\n' "${CXXFLAGS:-}" | _portable_arch_flags)
  printf 'CFLAGS: %s\nCXXFLAGS: %s\n' "$portable_cflags" "$portable_cxxflags"
  printf 'Rust build flags: %s\n' '-C opt-level=3 -C target-cpu=x86-64'
  CFLAGS="$portable_cflags" CXXFLAGS="$portable_cxxflags" \
    RUSTFLAGS="-C opt-level=3 -C target-cpu=x86-64" \
    CARGO_NET_OFFLINE=true ./MCTier-Linux-Web/scripts/build-web-server.sh
}

package() {
  cd "$srcdir/MCTier"
  local app="$pkgdir/usr/lib/mctier-linux-web"
  install -Dm755 MCTier-Linux-Web/scripts/launch-linux-web.sh "$app/launcher"
  install -Dm755 MCTier-Linux-Web/server/target/release/mctier-linux-web "$app/bin/mctier-linux-web-service"
  install -Dm755 MCTier-Linux-Web/resources/binaries/easytier-core "$app/bin/binaries/easytier-core"
  install -Dm755 MCTier-Linux-Web/resources/binaries/easytier-cli "$app/bin/binaries/easytier-cli"
  install -Dm755 "$srcdir/mctier.sh" "$pkgdir/usr/bin/mctier"
  install -Dm644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
  install -Dm644 "$srcdir/easytier-LICENSE" "$pkgdir/usr/share/licenses/$pkgname/EasyTier-LICENSE"
  for doc in README.md ADVANCED-NETWORK.md UFW-P2P-TROUBLESHOOTING.md; do
    install -Dm644 "MCTier-Linux-Web/$doc" "$pkgdir/usr/share/doc/$pkgname/$doc"
  done
  install -Dm644 "$srcdir/README.md" "$pkgdir/usr/share/doc/$pkgname/README-Arch.md"
}
