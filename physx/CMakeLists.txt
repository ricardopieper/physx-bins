cmake_minimum_required(VERSION 2.8.12)
project(physx-build)

set(PHYSX_ROOT_DIR "${CMAKE_CURRENT_LIST_DIR}/src/physx")

set(PX_OUTPUT_LIB_DIR "${CMAKE_CURRENT_LIST_DIR}")
set(PX_OUTPUT_BIN_DIR "${CMAKE_CURRENT_BINARY_DIR}/bin")

set(PX_GENERATE_STATIC_LIBRARIES ON CACHE BOOL "Enable static library generation" FORCE)

set(ENV{PM_CMakeModules_PATH} ${CMAKE_CURRENT_LIST_DIR}/src/externals/cmakemodules)
set(ENV{PM_PxShared_PATH} ${CMAKE_CURRENT_LIST_DIR}/src/pxshared)

if(UNIX)

    set(TARGET_BUILD_PLATFORM linux)

    if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "11.0.0")
            add_compile_options(-Wno-nonnull -Wno-mismatched-new-delete -Wno-stringop-overread)
        endif()
        if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "10.0.0")
            add_compile_options(-Wno-restrict -Wno-array-bounds -Wno-class-memaccess)
        endif()
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "12.0.0")
            add_compile_options(-Wno-suggest-override -Wno-suggest-destructor-override)
        endif()
        if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "11.0.0")
            add_compile_options(-Wno-dtor-name)
        endif()
    endif()

    if(CMAKE_BUILD_TYPE MATCHES Debug)
        # Output minimal debug info to reduce file size
        add_compile_options(-g1)
    endif()

elseif(WIN32)

    set(TARGET_BUILD_PLATFORM windows)
    set(NV_USE_STATIC_WINCRT ON CACHE BOOL "Enable PhysX static lib generation" FORCE)

    if(CMAKE_BUILD_TYPE MATCHES Debug)
        set(NV_USE_DEBUG_WINCRT TRUE CACHE BOOL "Force PhysX to use Debug WINCRT" FORCE)
    else()
        set(NV_USE_DEBUG_WINCRT FALSE CACHE BOOL "Force PhysX to use Debug WINCRT" FORCE)
    endif()

endif()

add_subdirectory(src/physx/compiler/public)
