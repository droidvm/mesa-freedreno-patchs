# mesa-freedreno-patchs
kgsl patchs for adreno a6xx/7xx mesa3d, running in proot env
目前只测试了 mesa-24.3.0-rc2

# ref
https://github.com/xMeM/termux-packages/commit/57b1bb44c9eed341c700105efed93f9fd8bc34a6
https://github.com/termux/termux-packages/commit/401982b8d9eaef70669762bfff2a963341c65e52
https://github.com/xMeM/termux-packages/tree/401982b8d9eaef70669762bfff2a963341c65e52/packages/mesa

# build
```bash
sudo apt-get install -y git patchelf pip zip meson ninja-build flex bison libarchive-dev glslang-tools python3-mako \
libxcb-randr0-dev  libdrm-dev libxcb-glx0-dev libx11-xcb-dev libxcb-dri3-dev libxcb-present-dev \
build-essential git python3-mako libdrm-dev libexpat1-dev libwayland-dev \
libwayland-egl-backend-dev libx11-xcb-dev libxcb-dri2-0-dev libxcb-dri3-dev libxcb-present-dev \
libxcb-randr0-dev libxcb-shm0-dev libxcb-sync-dev libxcb-xfixes0-dev libxrandr-dev libxshmfence-dev \
libxxf86vm-dev bison flex libssl-dev ninja-build meson pkg-config

cd mesa-24.3.0-rc2/
DIR_PATCHS=~/freedreno-kgsl-patchs
patch -p1 < ${DIR_PATCHS}/0010-fix-zink.patch
patch -p1 < ${DIR_PATCHS}/0011-freedreno-drm-kgsl-Add-KGSL-backend-for-freedreno.patch
patch -p1 < ${DIR_PATCHS}/0012-freedreno-drm-Add-more-APIs-to-per-backend-API.patch
patch -p1 < ${DIR_PATCHS}/0013-fix-bad-syscall.patch
patch -p1 < ${DIR_PATCHS}/0014-freedreno-HACK-GL_ARB_timer_query.patch
patch -p1 < ${DIR_PATCHS}/0015-termux-x11-kgsl.patch

RLT_DIR=~/mesa-out-arm64
DIR_NAME=build-proot-linux-aarch64
OPTIONS="-Dprefix=${RLT_DIR}/usr -Dbuildtype=release -Dplatforms=x11 \
-Dgallium-drivers=zink,freedreno \
-Dvulkan-drivers=freedreno \
-Dvulkan-beta=true -Dfreedreno-kmds=kgsl -Dgbm=enabled \
-Dshared-glapi=enabled -Dglx=dri -Dgallium-xa=enabled \
-Dopengl=true -Degl=enabled -Dglx-direct=true
"


echo "正在对源码进行编译前配置。禁用不需要的选项，可以减少编译量 ..."
rm -rf   ${RLT_DIR}
meson setup  --reconfigure  ${DIR_NAME} ${OPTIONS}

echo "正在编译 ..."
ninja -C ${DIR_NAME} 2>&1

echo "正在复制编译好的文件 ..."
ninja -C ${DIR_NAME} install

echo "编译成功，已将文件复制到: ${RLT_DIR}"
```
