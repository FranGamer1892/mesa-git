# Maintainer: Reza Jahanbakhshi <reza.jahanbakhshi at gmail dot com
# Contributor: Lone_Wolf <lone_wolf@klaas-de-kat.nl>
# Contributor: Armin K. <krejzi at email dot com>
# Contributor: Kristian Klausen <klausenbusk@hotmail.com>
# Contributor: Egon Ashrafinia <e.ashrafinia@gmail.com>
# Contributor: Tavian Barnes <tavianator@gmail.com>
# Contributor: Jan de Groot <jgc@archlinux.org>
# Contributor: Andreas Radke <andyrtr@archlinux.org>
# Contributor: Thomas Dziedzic < gostrc at gmail >
# Contributor: Antti "Tera" Oja <antti.bofh@gmail.com>
# Contributor: Diego Jose <diegoxter1006@gmail.com>

pkgname=mesa-git
pkgdesc="an open-source implementation of the OpenGL specification, git version"
pkgver=24.1.0_devel.186023.2cd192f8799.d41d8cd
pkgrel=1
arch=('x86_64')
makedepends=('git' 'python-mako' 'xorgproto' 'libxml2' 'libvdpau' 'libva' 'elfutils' 'libxrandr'
                            'wayland-protocols' 'meson' 'ninja' 'glslang' 'directx-headers' 'python-ply' 'rust' 'rust-bindgen'
)
depends=('libdrm' 'libxxf86vm' 'libxdamage' 'libxshmfence' 'libelf'
         'libomxil-bellagio' 'libunwind' 'libglvnd' 'wayland' 'lm_sensors' 
         'vulkan-icd-loader' 'zstd' 'expat' 'gcc-libs' 'libxfixes' 'libx11' 'systemd-libs' 'libxext' 'libxcb'
         'glibc' 'zlib'
)
optdepends=('opengl-man-pages: for the OpenGL API man pages')
provides=('mesa' 'vulkan-intel' 'vulkan-radeon' 'vulkan-mesa-layers' 'libva-mesa-driver' 'mesa-vdpau' 'vulkan-swrast' 'vulkan-driver' 'mesa-libgl' 'opengl-driver')
conflicts=('mesa' 'opencl-clover-mesa' 'opencl-rusticl-mesa' 'vulkan-intel' 'vulkan-radeon' 'vulkan-mesa-layers' 'libva-mesa-driver' 'mesa-vdpau' 'vulkan-swrast' 'mesa-libgl')
url="https://www.mesa3d.org"
license=('custom')
source=('mesa::git+https://gitlab.freedesktop.org/mesa/mesa.git#commit=2cd192f8799c8cc27cd76e3acae45e807aadd926'
             'LICENSE'
)
md5sums=('SKIP'
         '5c65a0fe315dd347e09b1f2826a1df5a')
sha512sums=('SKIP'
            '25da77914dded10c1f432ebcbf29941124138824ceecaf1367b3deedafaecabc082d463abcfa3d15abff59f177491472b505bcb5ba0c4a51bb6b93b4721a23c2')

# NINJAFLAGS is an env var used to pass commandline options to ninja
# NOTE: It's your responbility to validate the value of $NINJAFLAGS. If unsure, don't set it.

# MESA_WHICH_LLVM is an environment variable that determines which llvm package tree is used to built mesa-git against.
# Adding a line to ~/.bashrc  that sets this value is the simplest way to ensure a specific choice.
#
# NOTE: Aur helpers don't handle this method well, check the sticky comments on mesa-git aur page .
#
# 1: llvm-minimal-git (aur) preferred value
# 2: AUR llvm-git
# 3: llvm-git from LordHeavy unofficial repo 
# 4  llvm (stable from extra) Default value
# 

if [[ ! $MESA_WHICH_LLVM ]] ; then
    MESA_WHICH_LLVM=4
fi

case $MESA_WHICH_LLVM in
    1)
        # aur llvm-minimal-git
        makedepends+=('llvm-minimal-git' 'libclc-minimal-git' 'spirv-llvm-translator-minimal-git' 'clang-minimal-git' 'clang-opencl-headers-minimal-git')
        depends+=('llvm-libs-minimal-git')
        ;;
    2)
        # aur llvm-git
        # depending on aur-llvm-* to avoid mixup with LH llvm-git
        makedepends+=('aur-llvm-git')
        depends+=('aur-llvm-libs-git')
        optdepends+=('aur-llvm-git: opencl')
        ;;
    3)
        # mesa-git/llvm-git (lordheavy unofficial repo)
        makedepends+=('llvm-git' 'clang-git' 'libclc-git' 'spirv-tools' 'spirv-llvm-translator-git')
        depends+=('llvm-libs-git')
        optdepends+=('clang-git: opencl' 'compiler-rt: opencl')
        ;;
    4)
        # extra/llvm
        makedepends+=(llvm=17.0.6 clang=17.0.6 libclc spirv-llvm-translator)
        depends+=(llvm-libs=17.0.6)
        optdepends+=('clang: opencl' 'compiler-rt: opencl')
        ;;
    *)
esac

pkgver() {
    cd mesa
    local _ver
    _ver=$(<VERSION)

    local _patchver
    local _patchfile
    for _patchfile in "${source[@]}"; do
        _patchfile="${_patchfile%%::*}"
        _patchfile="${_patchfile##*/}"
        [[ $_patchfile = *.patch ]] || continue
        _patchver="${_patchver}$(md5sum ${srcdir}/${_patchfile} | cut -c1-32)"
    done
    _patchver="$(echo -n $_patchver | md5sum | cut -c1-7)"

    echo ${_ver/-/_}.$(git rev-list --count HEAD).$(git rev-parse --short HEAD).${_patchver}
}

prepare() {
    # although removing _build folder in build() function feels more natural,
    # that interferes with the spirit of makepkg --noextract
    if [  -d _build ]; then
        rm -rf _build
    fi

    local _patchfile
    for _patchfile in "${source[@]}"; do
        _patchfile="${_patchfile%%::*}"
        _patchfile="${_patchfile##*/}"
        [[ $_patchfile = *.patch ]] || continue
        echo "Applying patch $_patchfile..."
        patch --directory=mesa --forward --strip=1 --input="${srcdir}/${_patchfile}"
    done
}

build () {
    # Auto-download Rust crates for NAK (removes extra code for crate handling)
    _nak_crate="--force-fallback-for=syn"

    # HACK: Remove crate .rlib files before build
    # (This prevents build errors after a Rust update: https://github.com/mesonbuild/meson/issues/10706)
    [ -d build/subprojects ] && find build/subprojects -iname "*.rlib" -delete
    [ -d build/src/nouveau/compiler ] && find build/src/nouveau/compiler -iname "*.rlib" -delete

    meson setup mesa _build \
       -D b_ndebug=true \
       -D b_lto=false \
       -D platforms=x11,wayland \
       -D gallium-drivers=r300,r600,radeonsi,nouveau,virgl,svga,swrast,i915,iris,crocus,zink,d3d12 \
       -D vulkan-drivers=amd,intel,swrast,virtio,intel_hasvk,nouveau \
       -D vulkan-layers=device-select,overlay \
       -D dri3=enabled \
       -D egl=enabled \
       -D gallium-extra-hud=true \
       -D gallium-nine=true \
       -D gallium-omx=bellagio \
       -D gallium-opencl=disabled \
       -D gallium-va=enabled \
       -D gallium-vdpau=enabled \
       -D gallium-xa=enabled \
       -D gbm=enabled \
       -D gles1=disabled \
       -D gles2=enabled \
       -D glvnd=true \
       -D glx=dri \
       -D libunwind=enabled \
       -D llvm=enabled \
       -D lmsensors=enabled \
       -D osmesa=true \
       -D shared-glapi=enabled \
       -D microsoft-clc=disabled \
       -D valgrind=disabled \
       -D tools=[] \
       -D zstd=enabled \
       -D video-codecs=vc1dec,h264dec,h264enc,h265dec,h265enc,av1dec,av1enc,vp9dec \
       -D buildtype=plain \
       --wrap-mode=nofallback \
       ${_nak_crate} \
       -D prefix=/usr \
       -D sysconfdir=/etc
       
    meson configure --no-pager _build
    
    ninja $NINJAFLAGS -C _build
}

package() {
    DESTDIR="${pkgdir}" ninja $NINJAFLAGS -C _build install

    # remove script file from /usr/bin
    # https://gitlab.freedesktop.org/mesa/mesa/issues/2230
    rm "${pkgdir}/usr/bin/mesa-overlay-control.py"
    rmdir "${pkgdir}/usr/bin"

    # indirect rendering
    ln -s /usr/lib/libGLX_mesa.so.0 "${pkgdir}/usr/lib/libGLX_indirect.so.0"
  
    install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" "${srcdir}/LICENSE"
}
