cmake_minimum_required(VERSION 3.0.0)

project(physx VERSION 0.1)

if("${CMAKE_PROJECT_NAME}" STREQUAL "${PROJECT_NAME}")
    include(ExternalProject)
    ExternalProject_Add(physx-build-debug
        GIT_REPOSITORY      https://github.com/NVIDIAGameWorks/PhysX.git
        GIT_TAG             origin/4.1
        SOURCE_DIR          "${CMAKE_CURRENT_LIST_DIR}/physx/src"
        BINARY_DIR          "${CMAKE_CURRENT_LIST_DIR}/physx/build-debug"
        SOURCE_SUBDIR       "../"
        CMAKE_ARGS          "-DCMAKE_BUILD_TYPE=Debug"
        INSTALL_COMMAND     ""
        BUILD_BYPRODUCTS    "${CMAKE_CURRENT_LIST_DIR}/physx/src/physx/include"
    )
    ExternalProject_Add(physx-build-release
        DOWNLOAD_COMMAND    ""
        UPDATE_DISCONNECTED true
        SOURCE_DIR          "${CMAKE_CURRENT_LIST_DIR}/physx/src"
        BINARY_DIR          "${CMAKE_CURRENT_LIST_DIR}/physx/build-release"
        SOURCE_SUBDIR       "../"
        CMAKE_ARGS          "-DCMAKE_BUILD_TYPE=Release"
        INSTALL_COMMAND     ""
        DEPENDS             physx-build-debug
    )

    add_custom_command(TARGET physx-build-debug POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_CURRENT_LIST_DIR}/physx/src/physx/include" "${CMAKE_CURRENT_LIST_DIR}/include"
        BYPRODUCTS "${CMAKE_CURRENT_LIST_DIR}/include"
    )

    add_custom_command(TARGET physx-build-debug POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_CURRENT_LIST_DIR}/physx/src/pxshared/include/foundation" "${CMAKE_CURRENT_LIST_DIR}/include/foundation"
        BYPRODUCTS "${CMAKE_CURRENT_LIST_DIR}/include/foundation"
    )
endif()

set(physx_libraries
    PhysX
    PhysXCharacterKinematic
    PhysXCommon
    PhysXCooking
    PhysXExtensions
    PhysXFoundation
    PhysXPvdSDK
)
foreach(lib_name IN LISTS physx_libraries)
    add_library(${lib_name} STATIC IMPORTED GLOBAL)
    target_include_directories(${lib_name} INTERFACE "${CMAKE_CURRENT_LIST_DIR}/include")

    string(TOLOWER "${CMAKE_BUILD_TYPE}" build_type_lowercase)
    if("${build_type_lowercase}" STREQUAL "relwithdebinfo")
        set(build_type_lowercase "release")
    endif()

    if(UNIX)
        set_target_properties(${lib_name} PROPERTIES IMPORTED_LOCATION "${CMAKE_CURRENT_LIST_DIR}/physx/bin/linux.clang/${build_type_lowercase}/lib${lib_name}_static.a")
    elseif(WIN32)
        set_target_properties(${lib_name} PROPERTIES IMPORTED_LOCATION "${CMAKE_CURRENT_LIST_DIR}/physx/bin/win.x86_64.vc142.mt/${build_type_lowercase}/${lib_name}_static.lib")
    else()
        message(FATAL_ERROR "Unknown target platform")
    endif()
endforeach()

if(UNIX)
    target_link_libraries(PhysX INTERFACE dl)
endif()

# Define physx library inter-dependencies so link ordering is correct.
target_link_libraries(PhysX INTERFACE PhysXPvdSDK)
target_link_libraries(PhysX INTERFACE PhysXCommon)
target_link_libraries(PhysXCharacterKinematic INTERFACE PhysXCommon)
target_link_libraries(PhysXCharacterKinematic INTERFACE PhysXExtensions)
target_link_libraries(PhysXCooking INTERFACE PhysXCommon)
target_link_libraries(PhysXCooking INTERFACE PhysXFoundation)
target_link_libraries(PhysXExtensions INTERFACE PhysXFoundation)
target_link_libraries(PhysXPvdSDK INTERFACE PhysXFoundation)
