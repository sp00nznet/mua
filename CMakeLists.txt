# Marvel: Ultimate Alliance (BLUS30010, Raven Software / Activision) — PPU boot harness.
#
# Same shape as vf5/ and simpsons/: the shared ps3recomp harness loads the real
# EBOOT, registers the lifted functions + HLE NID handlers, and runs real game
# code; every unlifted function, unimplemented import and lv2 syscall logs
# itself. Nothing title-specific lives here yet -- MUA was picked because its OS
# surface is small (126 imports over 11 libraries, no cellSaveData, no cellGame).
#
# Graphics come from the harness: RSX_LIVE_DRAW=1 selects the live NV4097 ->
# D3D12 draw engine, no code of our own.
#
# Toolchain: clang-cl (the harness uses __builtin_bswap + weak symbols MSVC lacks).
#   cmake -S . -B build -G Ninja -DCMAKE_C_COMPILER=clang-cl -DCMAKE_CXX_COMPILER=clang-cl
#   cmake --build build
#   PS3_VFS_ROOT=input RSX_LIVE_DRAW=1 ./build/mua input/EBOOT.elf
cmake_minimum_required(VERSION 3.20)
project(mua CXX C)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_C_STANDARD 17)

set(PS3RECOMP_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../ps3recomp" CACHE PATH "ps3recomp repo root")
set(PS3RECOMP_RUNTIME_LIB "${PS3RECOMP_DIR}/build/ps3recomp_runtime.lib"
    CACHE FILEPATH "prebuilt runtime library")

# Lifted game code + the generated HLE NID table. Both git-ignored; regenerate
# with the commands in README.md.
file(GLOB RECOMP_SOURCES "src/recomp/*.cpp")
file(GLOB GEN_SOURCES    "src/gen/*.cpp" "src/gen/*.c")
file(GLOB_RECURSE SPU_SOURCES "src/spu_gen/*.c")
if(NOT RECOMP_SOURCES)
    message(FATAL_ERROR "src/recomp/ is empty -- lift first (see README.md)")
endif()

add_executable(mua
    "${PS3RECOMP_DIR}/runtime/ppu/tests/boot_main.cpp"   # generic boot harness
    "${PS3RECOMP_DIR}/runtime/ppu/ppu_loader.cpp"        # vm + ELF loader + run
    "${PS3RECOMP_DIR}/runtime/ppu/ppu_hle.cpp"           # NID -> handler dispatch
    "${PS3RECOMP_DIR}/runtime/ppu/ppu_sysprx.cpp"        # boot-critical CRT
    "${PS3RECOMP_DIR}/runtime/ppu/ppu_fs.cpp"            # cellFs VFS
    ${GEN_SOURCES}
    ${SPU_SOURCES}
    ${RECOMP_SOURCES}
)

# Every libs/* subdir (the generated NID table #includes each module's header).
file(GLOB LIB_INC_DIRS LIST_DIRECTORIES true "${PS3RECOMP_DIR}/libs/*")

target_include_directories(mua PRIVATE
    "${CMAKE_CURRENT_SOURCE_DIR}/src/recomp"            # ppu_recomp.h
    "${PS3RECOMP_DIR}/runtime/ppu"
    "${PS3RECOMP_DIR}/runtime/spu"
    "${PS3RECOMP_DIR}/runtime/prx"
    "${PS3RECOMP_DIR}/include"
    ${LIB_INC_DIRS}
)

target_link_libraries(mua PRIVATE "${PS3RECOMP_RUNTIME_LIB}")
if(WIN32)
    target_link_libraries(mua PRIVATE ws2_32 bcrypt xinput ole32 d3d12 dxgi dxguid d3dcompiler)
    # Every guest call is a host call, so recompiled call chains nest deeply.
    target_link_options(mua PRIVATE /STACK:268435456)
endif()
