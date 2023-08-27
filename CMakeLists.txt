cmake_minimum_required(VERSION 3.21)
project(stb VERSION 0.0.0)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)         # Generate compile_commands.json for ccls
set_property(GLOBAL PROPERTY USE_FOLDERS ON)  # Group CMake targets inside a folder

# Install directory setup
include(GNUInstallDirs)
set(STB_INC_DIR ${CMAKE_INSTALL_INCLUDEDIR})

set(HEADER_FILES
    "${PROJECT_SOURCE_DIR}/stb_image_write.h"
    "${PROJECT_SOURCE_DIR}/stb_image.h"
)
set(SOURCE_FILES
    "stb_image.cpp"
)
add_library(stb STATIC ${HEADER_FILES} ${SOURCE_FILES})
target_include_directories(stb
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${STB_INC_DIR}>
)
target_compile_features(stb PRIVATE cxx_std_20)
add_library(stb::stb ALIAS stb)

# Setup install targets
include(CMakePackageConfigHelpers)
set(STB_CMAKE_DIR ${CMAKE_INSTALL_LIBDIR}/cmake/stb)
set(version_config "${PROJECT_BINARY_DIR}/stb-config-version.cmake")
set(project_config "${PROJECT_BINARY_DIR}/stb-config.cmake")
set(pkgconfig ${PROJECT_BINARY_DIR}/stb.pc)
set(targets_export_name stb-targets)
set(STB_LIB_NAME stb)

write_basic_package_version_file(
    ${version_config}
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

set(STB_LIB_DIR ${CMAKE_INSTALL_LIBDIR})
set(STB_PKGCONFIG_DIR ${CMAKE_INSTALL_LIBDIR}/pkgconfig)

# Create path
string(CONCAT libdir_for_pc_file "\${exec_prefix}" "/" "${STB_LIB_DIR}")
string(CONCAT includedir_for_pc_file "\${prefix}" "/" "${STB_INC_DIR}")

# pkg-config support
configure_file(
    ${PROJECT_SOURCE_DIR}/support/cmake/stb.pc.in  # Input file
    ${pkgconfig}
    @ONLY
)

# CMake package config
configure_package_config_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/support/cmake/stb.cmake.in
    ${project_config}
    INSTALL_DESTINATION ${STB_CMAKE_DIR}
)

# Setup install targets
set(INSTALL_TARGETS stb)

# Install library and headers
install(TARGETS ${INSTALL_TARGETS} EXPORT ${targets_export_name}
        LIBRARY DESTINATION ${STB_LIB_DIR}
        ARCHIVE DESTINATION ${STB_LIB_DIR}
        PUBLIC_HEADER DESTINATION "${STB_INC_DIR}"
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
install(FILES ${HEADER_FILES} DESTINATION "${STB_INC_DIR}")

export(TARGETS ${INSTALL_TARGETS} NAMESPACE stb::
       FILE ${PROJECT_BINARY_DIR}/${targets_export_name}.cmake)

# Install version, config and target files.
install(
    FILES ${project_config} ${version_config}
    DESTINATION ${STB_CMAKE_DIR}
)
install(EXPORT ${targets_export_name} DESTINATION ${STB_CMAKE_DIR}
        NAMESPACE stb::)

# TODO: pkg-config doesn't work as expected on Windows
install(FILES "${pkgconfig}" DESTINATION "${STB_PKGCONFIG_DIR}")
