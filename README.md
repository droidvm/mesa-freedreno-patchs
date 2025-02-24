# mesa-freedreno-patches
kgsl patchs for adreno a6xx/7xx mesa3d, running in proot env

# 实测结果
- 目前只在proot-ubuntu-24.04中测试了 mesa-24.3.0 或 mesa-24.3.0-rc2
- 其它版本的mesa3d源码，可能要参照 patch 文件手改修补代码才能编译运行

- vulkan.trunip 测试程序：vulkaninfo、vkcube、vkmark 运行正常

- opengl.zink 测试程序：glmark2 测试正常

- opengl.freedreno(kgsl mode) 测试程序：glmark2 测试正常

# ref
1. vulkan.tunrip patches
- https://github.com/MastaG/mesa-turnip-ppa/tree/main/turnip-patches

2. opengl.freedreno in kgsl mode patches
- https://github.com/xMeM/termux-packages/commit/57b1bb44c9eed341c700105efed93f9fd8bc34a6
- https://github.com/termux/termux-packages/commit/401982b8d9eaef70669762bfff2a963341c65e52
- https://github.com/xMeM/termux-packages/tree/401982b8d9eaef70669762bfff2a963341c65e52/packages/mesa

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
patch -p1 < ${DIR_PATCHS}/0002-linux-fix-for-getprogname.patch
patch -p1 < ${DIR_PATCHS}/0003-linux-fix-for-anon-file.patch
patch -p1 < ${DIR_PATCHS}/0007-linux-dri3.patch
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

# run
```bash
export LD_LIBRARY_PATH=$RLT_DIR/usr/lib/aarch64-linux-gnu:$RLT_DIR/usr/lib/aarch64-linux-gnu/dri
export MESA_LOADER_DRIVER_OVERRIDE=kgsl
export VK_ICD_FILENAMES=$RLT_DIR/usr/share/vulkan/icd.d/freedreno_icd.aarch64.json
export TU_DEBUG=noconform
glmark2

```

# 注意
- 如果将编译好的文件放到别的文件夹，请相应的调整 share/vulkan/icd.d/freedreno_icd.aarch64.json 文件中so库文件的搜索路径
- 编辑 ~/.config/xfce4/xfconf/xfce-perchannel-xml/xfwm4.xml, 将 vblank_mode 参数的值，从auto改为off




