cmake_minimum_required(VERSION 3.2)

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.23.32.tar.gz"
    SHA1 "720311022e7419d083ad9513ca0e752004f106e2"
)

project(mshadow C CXX)

option(MSHADOW_USE_F16C "Use f16c" OFF)

hunter_add_package(mkl)

set(mshadow_LINT_DIRS mshadow mshadow-ps)
add_custom_target(mshadow_lint COMMAND ${CMAKE_COMMAND} -DMSVC=${MSVC} -DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE} -DLINT_DIRS=${mshadow_LINT_DIRS} -DPROJECT_SOURCE_DIR=${PROJECT_SOURCE_DIR} -DPROJECT_NAME=mshadow -P ${PROJECT_SOURCE_DIR}/../dmlc-core/cmake/lint.cmake)

add_library(mshadow INTERFACE)

if(MSHADOW_USE_F16C)
  set(value 1)
else()
  set(value 0)
endif()

target_compile_definitions(mshadow INTERFACE MSHADOW_USE_F16C=${value})

# Installation (https://github.com/forexample/package-example) {

# Introduce variables:
# * CMAKE_INSTALL_LIBDIR
# * CMAKE_INSTALL_BINDIR
# * CMAKE_INSTALL_INCLUDEDIR
include(GNUInstallDirs)

set(config_install_dir "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)

# Use:
#   * TARGETS_EXPORT_NAME
#   * PROJECT_NAME
configure_package_config_file(
    "cmake/template/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(
    TARGETS mshadow
    EXPORT "${TARGETS_EXPORT_NAME}"
    INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)

install(
    DIRECTORY "mshadow"
    DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)

install(
    FILES "${project_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)

# }
