cmake_minimum_required(VERSION 3.29)

set(KF_VERSION "6.29.0") # handled by release scripts
project(KIdleTime VERSION ${KF_VERSION})

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# ECM setup
include(FeatureSummary)
find_package(ECM 6.28.0  NO_MODULE)
set_package_properties(ECM PROPERTIES TYPE REQUIRED DESCRIPTION "Extra CMake Modules." URL "https://commits.kde.org/extra-cmake-modules")
feature_summary(WHAT REQUIRED_PACKAGES_NOT_FOUND FATAL_ON_MISSING_REQUIRED_PACKAGES)

set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH})

include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDEFrameworkCompilerSettings NO_POLICY_SCOPE)

include(KDEGitCommitHooks)
include(CMakeFindFrameworks)
include(CMakePackageConfigHelpers)
include(ECMGenerateExportHeader)
include(ECMQtDeclareLoggingCategory)
include(ECMSetupVersion)
include(ECMGenerateHeaders)
include(ECMDeprecationSettings)
include(ECMGenerateQDoc)

set(kiconthemes_version_header "${CMAKE_CURRENT_BINARY_DIR}/src/kidletime_version.h")
ecm_setup_version(PROJECT VARIABLE_PREFIX KIDLETIME
                        VERSION_HEADER "${kiconthemes_version_header}"
                        PACKAGE_VERSION_FILE "${CMAKE_CURRENT_BINARY_DIR}/KF6IdleTimeConfigVersion.cmake"
                        SOVERSION 6)

set(REQUIRED_QT_VERSION 6.9.0)
find_package(Qt6 ${REQUIRED_QT_VERSION} CONFIG REQUIRED Gui)

set(KIDLETIME_UNIX_X11 FALSE)
if(CMAKE_SYSTEM_NAME STREQUAL "Linux" OR CMAKE_SYSTEM_NAME STREQUAL "FreeBSD")
    set(KIDLETIME_UNIX_X11 TRUE)
endif()

if(APPLE)
    cmake_find_frameworks(CoreFoundation Carbon)
endif()

# Linux/FreeBSD require the XSync backend
if(KIDLETIME_UNIX_X11)
    find_package(X11 REQUIRED)
    find_package(XCB REQUIRED COMPONENTS XCB SYNC)
    find_package(X11_XCB REQUIRED)

    if(NOT X11_XSync_FOUND)
        message(FATAL_ERROR
            "KIdleTime on Linux/FreeBSD requires the XSync extension (libXext).\n"
            "Please install libxext development headers and libraries.")
    endif()

    add_feature_info("libXext" X11_XSync_FOUND "required for X-Sync based poller")
    add_feature_info("libX11-xcb" X11_XCB_FOUND "required for X-Sync based poller")
    add_feature_info("libxcb" XCB_XCB_FOUND "required for X-Sync based poller")
    add_feature_info("libxcb-sync" XCB_SYNC_FOUND "required for X-Sync based poller")
endif()

set(EXCLUDE_DEPRECATED_BEFORE_AND_AT 0 CACHE STRING "Control the range of deprecated API excluded from the build [default=0].")

ecm_set_disabled_deprecation_versions(
    QT 6.11.0
)

# Subdirectories
add_subdirectory(src)
add_subdirectory(examples)

# create a Config.cmake and a ConfigVersion.cmake file and install them
set(CMAKECONFIG_INSTALL_DIR "${KDE_INSTALL_CMAKEPACKAGEDIR}/KF6IdleTime")

configure_package_config_file("${CMAKE_CURRENT_SOURCE_DIR}/KF6IdleTimeConfig.cmake.in"
                              "${CMAKE_CURRENT_BINARY_DIR}/KF6IdleTimeConfig.cmake"
                              INSTALL_DESTINATION  ${CMAKECONFIG_INSTALL_DIR}
                              )

install(FILES  "${CMAKE_CURRENT_BINARY_DIR}/KF6IdleTimeConfig.cmake"
               "${CMAKE_CURRENT_BINARY_DIR}/KF6IdleTimeConfigVersion.cmake"
        DESTINATION "${CMAKECONFIG_INSTALL_DIR}"
        COMPONENT Devel )

install(EXPORT KF6IdleTimeTargets DESTINATION "${CMAKECONFIG_INSTALL_DIR}" FILE KF6IdleTimeTargets.cmake NAMESPACE KF6:: )

install(FILES ${kiconthemes_version_header}
        DESTINATION ${KDE_INSTALL_INCLUDEDIR_KF}/KIdleTime COMPONENT Devel)

include(ECMFeatureSummary)
ecm_feature_summary(WHAT ALL   FATAL_ON_MISSING_REQUIRED_PACKAGES)

kde_configure_git_pre_commit_hook(CHECKS CLANG_FORMAT)
