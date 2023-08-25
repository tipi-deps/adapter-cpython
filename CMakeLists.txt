

cmake_minimum_required(VERSION 3.17.0)
project(middleware_cpython VERSION "0.0.1")

set(cpython_SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/cpython-src)
set(cpython_INSTALL_DIR ${CMAKE_INSTALL_PREFIX})
set(cpython_INCLUDE_DIR ${cpython_INSTALL_DIR}/include)
set(cpython_CONFIGURE_COMMAND ${cpython_SOURCE_DIR}/configure)


list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}")
include(ExternalProject)
include(ProcessorCount)
include(hunter_autotools_project)
include(hunter_cmake_args)

ProcessorCount(Ncore)

find_package(OpenSSL REQUIRED)

hunter_autotools_project("cpython"
    HUNTER_SELF
      "${CMAKE_CURRENT_LIST_DIR}"
    URL
      "https://github.com/python/cpython/archive/refs/tags/v3.11.5.tar.gz"
    URL_HASH
      SHA1=19d0d9683c2a513ea72a5036d86a81ff6c0d8e78
    DOWNLOAD_DIR
      "${CMAKE_CURRENT_BINARY_DIR}"
    SOURCE_DIR
      "${cpython_SOURCE_DIR}"
    BUILD_DIR
      "${CMAKE_CURRENT_BINARY_DIR}"
    INSTALL_DIR
      "${CMAKE_INSTALL_PREFIX}"
    GLOBAL_INSTALL_DIR
      "${CMAKE_INSTALL_PREFIX}"
    PARALLEL_JOBS
      "${Ncore}"
    PACKAGE_CONFIGURATION_TYPES
      "${CMAKE_BUILD_TYPE}"
    EXTRA_FLAGS 
      --with-openssl=${OPENSSL_INCLUDE_DIR}/.. # openssl root dir is one level up the includes
      --with-pkg-config=no
      #--enable-optimizations # disabled because LLVM Profile Error: Failed to write file "code-7271.profclangr": No such file or directory
    BOOTSTRAP
      ""
)


# Installing
set (EXTERNAL_PROJECT_NAME cpython)

set(include_install_dir "include")


# Layout. This works for all platforms:
#   * <prefix>/lib/cmake/<PROJECT-NAME>
#   * <prefix>/lib/
#   * <prefix>/include/
set(config_install_dir "lib/cmake/${EXTERNAL_PROJECT_NAME}")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

# Configuration
set(version_config "${generated_dir}/${EXTERNAL_PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${EXTERNAL_PROJECT_NAME}Config.cmake")
set(targets_export_name "${EXTERNAL_PROJECT_NAME}Targets")
set(namespace "${EXTERNAL_PROJECT_NAME}::")

# Include module with fuction 'write_basic_package_version_file'
include(CMakePackageConfigHelpers)

# Configure '<PROJECT-NAME>ConfigVersion.cmake'
# Note: PROJECT_VERSION is used as a VERSION
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Configure '<PROJECT-NAME>Config.cmake'
# Use variables:
#   * targets_export_name
#   * EXTERNAL_PROJECT_NAME
configure_package_config_file(
    "${CMAKE_CURRENT_LIST_DIR}/cmake/modules/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)


# Targets:
#install(
#    TARGETS libpq
#    EXPORT "${targets_export_name}"
#    LIBRARY DESTINATION "lib"
#    ARCHIVE DESTINATION "lib"
#    RUNTIME DESTINATION "bin"
#    INCLUDES DESTINATION "${include_install_dir}"
#)

# Config
#   * <prefix>/lib/cmake/cpython/cpythonConfig.cmake
#   * <prefix>/lib/cmake/cpython/cpythonConfigVersion.cmake
#   * <prefix>/lib/cmake/cpython/cpythonTargets.cmake
install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)
#install(
#    EXPORT "${targets_export_name}"
#    NAMESPACE "${namespace}"
#    DESTINATION "${config_install_dir}"
#)
