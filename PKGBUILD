pkgname=yukigram-git
pkgver=1.1.16.r18486.147eebbb9d
pkgrel=1
pkgdesc='64gram fork with some enhancements, which itself is Telegram Desktop fork'
arch=('x86_64')
url="https://t.me/sylfngram"
license=('GPL3')
depends=('hunspell' 'ffmpeg' 'hicolor-icon-theme' 'lz4' 'minizip' 'openal' 'ttf-opensans'
         'qt6-imageformats' 'qt6-svg' 'qt6-wayland' 'xxhash'
         'rnnoise' 'pipewire' 'libxtst' 'libxrandr' 'libxcomposite' 'abseil-cpp' 'libdispatch'
         'openssl' 'protobuf' 'glib2' 'libsigc++-3.0' 'glibmm-2.68')
makedepends=('cmake' 'git' 'ninja' 'python' 'range-v3' 'tl-expected' 'microsoft-gsl' 'meson'
             'extra-cmake-modules' 'wayland-protocols' 'plasma-wayland-protocols' 'libtg_owt'
             'gobject-introspection' 'boost' 'fmt' 'mm-common' 'perl-xml-parser')
optdepends=('webkit2gtk: embedded browser features'
            'xdg-desktop-portal: desktop integration')
source=("$pkgname::git+https://github.com/yukigram/yukigram.git"
        "fix-lzma.patch"
        "bundled-tgvoip.patch")
sha512sums=('SKIP'
            'SKIP'
            'SKIP')


pkgver() {
    [[ -z "$HEAD" ]] && HEAD=origin/HEAD
    cd "$pkgname"
    local version="$(grep AppVersionStr Telegram/SourceFiles/core/version.h | sed 's|.;||' | sed 's|constexpr auto AppVersionStr = .||')"
    printf "%s.r%s.%s" "$version" "$(git rev-list --count $HEAD)" "$(git rev-parse --short=10 $HEAD)"
}

prepare() {
    cd "$pkgname"
    git checkout $HEAD
    git submodule deinit --force --all
    git submodule update --init --recursive
    patch -Np1 < ../fix-lzma.patch
    patch -Np1 < ../bundled-tgvoip.patch
}

bail() {
    echo "$@"
    exit 1
}

validate_api() {
    [[ "$API_ID" =~ ^[1-9][0-9]*$ ]] || bail "API_ID must be a positive number"
    [[ "$API_HASH" =~ ^[0-9a-f]{32}$ ]] || bail "API_HASH must contain 32 hex digits [0-9a-f]"
}

build() {
    validate_api

    cmake -B build \
        -S $pkgname \
        -G Ninja \
        -D CMAKE_INSTALL_PREFIX="/usr" \
        -D CMAKE_BUILD_TYPE=Release \
        -D CMAKE_INTERPROCEDURAL_OPTIMIZATION=ON \
        -D CMAKE_C_FLAGS_DEBUG="" \
        -D CMAKE_CXX_FLAGS_DEBUG="" \
        -D CMAKE_C_FLAGS="" \
        -D CMAKE_CXX_FLAGS="" \
        -D CMAKE_EXE_LINKER_FLAGS="-s" \
        -D TDESKTOP_API_ID="$API_ID" \
        -D TDESKTOP_API_HASH="$API_HASH" \
        -D DESKTOP_APP_DISABLE_AUTOUPDATE=ON \
        -Wno-dev
    
    cmake --build build --config Release --parallel
}

package() {
    DESTDIR="$pkgdir" cmake --install build
}

# Based on https://aur.archlinux.org/cgit/aur.git/tree/PKGBUILD?h=telegram-desktop-userfonts (commit 9ce5fd07)
# fix-lzma.patch took from https://aur.archlinux.org/cgit/aur.git/tree/PKGBUILD?h=64gram-desktop (commit 2f6d1aeb)
