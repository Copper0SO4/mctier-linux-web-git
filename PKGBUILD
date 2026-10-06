# Maintainer: Copper0SO4
pkgname=mctier-linux-web-git
pkgver=3.9.0.r379.g9f64665
pkgrel=1
pkgdesc='MCTier Linux Web: virtual networking, chat and browser-based voice/screen sharing'
arch=('x86_64')
url='https://github.com/Copper0SO4/MCTier_Linux_Web'
license=('custom:MCTier-NonCommercial' 'LGPL-3.0-only')
depends=('bash' 'glibc' 'gcc-libs' 'openssl' 'dbus' 'libcap' 'polkit' 'curl' 'xdg-utils' 'coreutils' 'gawk')
makedepends=('git' 'nodejs>=20' 'npm' 'rust>=1.90' 'desktop-file-utils')
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
  'mctier.desktop'
  'README.md'
)
noextract=('easytier-linux-x86_64-v2.5.0.zip')
sha256sums=(
  'SKIP'
  'c715d62ffcdad2578bc5d743bdfbcfe02cda9c12f80bd1bad3b36d4d7fc0eb8b'
  'e3a994d82e644b03a792a930f574002658412f62407f5fee083f2555c5f23118'
  'ba3b7137f5544d63e96c1e4458bb584c4605b29227969c1a54e30a75388b569e'
  '38ff3bb75ee54dd69f052e079f82855a13c7ef80312692d521db87bafa5beb32'
  '0a0a27f417adfb5d93b6ddbaea5e218be07b747bf293a75e6d678722cc1146f8'
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

prepare() {
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

build() {
  cd "$srcdir/MCTier"
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
  install -Dm644 "$srcdir/mctier.desktop" "$pkgdir/usr/share/applications/mctier.desktop"
  install -Dm644 public/MCTierIcon.png "$pkgdir/usr/share/pixmaps/mctier.png"
  install -Dm644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
  install -Dm644 "$srcdir/easytier-LICENSE" "$pkgdir/usr/share/licenses/$pkgname/EasyTier-LICENSE"
  for doc in README.md ADVANCED-NETWORK.md UFW-P2P-TROUBLESHOOTING.md; do
    install -Dm644 "MCTier-Linux-Web/$doc" "$pkgdir/usr/share/doc/$pkgname/$doc"
  done
  install -Dm644 "$srcdir/README.md" "$pkgdir/usr/share/doc/$pkgname/README-Arch.md"
  desktop-file-validate "$pkgdir/usr/share/applications/mctier.desktop"
}
