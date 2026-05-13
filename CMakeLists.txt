if(NOT DEFINED NSX_AMBIQSUITE_ROOT)
    message(FATAL_ERROR "NSX_AMBIQSUITE_ROOT must be configured before ns-ambiq-bsp-r4 is added.")
endif()
if(NOT DEFINED NSX_BOARD_FLAGS_TARGET)
    message(FATAL_ERROR "Board must define NSX_BOARD_FLAGS_TARGET before ns-ambiq-bsp-r4 is added.")
endif()
if(NOT DEFINED NSX_AMBIQ_BOARD_NAME OR NOT DEFINED NSX_AMBIQ_BSP_DIR OR NOT DEFINED NSX_AMBIQ_PART_NAME)
    message(FATAL_ERROR "Board must define NSX_AMBIQ_BOARD_NAME, NSX_AMBIQ_BSP_DIR, and NSX_AMBIQ_PART_NAME before ns-ambiq-bsp-r4 is added.")
endif()
if(NOT NSX_SDK_PROVIDER STREQUAL "ambiqsuite-r4")
    message(FATAL_ERROR "ns-ambiq-bsp-r4 requires NSX_SDK_PROVIDER=ambiqsuite-r4.")
endif()

option(NSX_AMBIQSUITE_USE_PREBUILT "Use prebuilt AmbiqSuite static libraries" ON)

set(NSX_AMBIQ_INSTALL_BASE "${CMAKE_INSTALL_INCLUDEDIR}/nsx/ambiqsuite/${NSX_AMBIQSUITE_VERSION}")

# Select the correct prebuilt archive for the active toolchain.
if(NSX_TOOLCHAIN_FAMILY STREQUAL "armclang")
    set(NSX_BSP_LIB_DEFAULT "${NSX_AMBIQSUITE_ROOT}/lib/${NSX_AMBIQ_PART_NAME}/evb/libam_bsp.lib")
elseif(NSX_TOOLCHAIN_FAMILY STREQUAL "atfe")
    # ATFE-specific prebuilts are not yet available; fall back to the
    # GCC-built archive which is AAPCS-compatible.
    set(NSX_BSP_LIB_DEFAULT "${NSX_AMBIQSUITE_ROOT}/lib/${NSX_AMBIQ_PART_NAME}/evb/libam_bsp.a")
    message(STATUS "ns-ambiq-bsp-r4: ATFE prebuilt not available, using GCC variant (AAPCS-compatible)")
else()
    set(NSX_BSP_LIB_DEFAULT "${NSX_AMBIQSUITE_ROOT}/lib/${NSX_AMBIQ_PART_NAME}/evb/libam_bsp.a")
endif()
set(NSX_BSP_LIB "${NSX_BSP_LIB_DEFAULT}" CACHE FILEPATH "Path to AmbiqSuite BSP static library")

add_library(nsx_ambiq_bsp INTERFACE)
set_target_properties(nsx_ambiq_bsp PROPERTIES EXPORT_NAME ambiq_bsp_r4)
add_library(nsx::ambiq_bsp ALIAS nsx_ambiq_bsp)
add_library(nsx::ambiq_bsp_r4 ALIAS nsx_ambiq_bsp)
if(NOT TARGET nsx_ambiq_hal)
    message(FATAL_ERROR "ns-ambiq-bsp-r4 requires nsx_ambiq_hal to be added first.")
endif()
target_link_libraries(nsx_ambiq_bsp INTERFACE ${NSX_BOARD_FLAGS_TARGET} nsx_ambiq_hal)
target_include_directories(nsx_ambiq_bsp INTERFACE
    $<BUILD_INTERFACE:${NSX_AMBIQ_BSP_DIR}>
    $<INSTALL_INTERFACE:${NSX_AMBIQ_INSTALL_BASE}/boards/${NSX_AMBIQ_BOARD_NAME}/bsp>
)

if(NSX_AMBIQSUITE_USE_PREBUILT)
    nsx_validate_prebuilt_abi()
    nsx_assert_file_exists("${NSX_BSP_LIB}")
    target_link_libraries(nsx_ambiq_bsp INTERFACE "${NSX_BSP_LIB}")
else()
    message(FATAL_ERROR "ns-ambiq-bsp-r4 source fallback is not implemented yet.")
endif()

install(DIRECTORY
    ${NSX_AMBIQSUITE_ROOT}/boards/${NSX_AMBIQ_BOARD_NAME}/bsp
    DESTINATION ${NSX_AMBIQ_INSTALL_BASE}/boards/${NSX_AMBIQ_BOARD_NAME}
)

install(TARGETS
    nsx_ambiq_bsp
    EXPORT nsxTargets
)
