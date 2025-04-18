cmake_minimum_required(VERSION 3.16)

set(KF_VERSION "5.115.0") # handled by release scripts
project(KArchive VERSION ${KF_VERSION})

include(FeatureSummary)
find_package(ECM 5.115.0  NO_MODULE)
set_package_properties(ECM PROPERTIES TYPE REQUIRED DESCRIPTION "Extra CMake Modules." URL "https://commits.kde.org/extra-cmake-modules")
feature_summary(WHAT REQUIRED_PACKAGES_NOT_FOUND FATAL_ON_MISSING_REQUIRED_PACKAGES)


set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH})

include(KDEInstallDirs)
include(KDEFrameworkCompilerSettings NO_POLICY_SCOPE)
include(KDECMakeSettings)
include(KDEGitCommitHooks)

include(ECMGenerateExportHeader)

set(REQUIRED_QT_VERSION 5.15.2)
find_package(Qt${QT_MAJOR_VERSION}Core ${REQUIRED_QT_VERSION} REQUIRED NO_MODULE)

find_package(ZLIB)
set_package_properties(ZLIB PROPERTIES
    URL "https://www.zlib.net"
    DESCRIPTION "Support for gzip compressed files and data streams"
    TYPE REQUIRED
    PURPOSE "Support for gzip compressed files and data streams"
)

find_package(BZip2)
set_package_properties(BZip2 PROPERTIES
    URL "https://sourceware.org/bzip2/"
    DESCRIPTION "Support for BZip2 compressed files and data streams"
    TYPE RECOMMENDED
    PURPOSE "Support for BZip2 compressed files and data streams"
)

find_package(LibLZMA)
set_package_properties(LibLZMA PROPERTIES
    URL "https://tukaani.org/xz/"
    DESCRIPTION "Support for xz compressed files and data streams"
    PURPOSE "Support for xz compressed files and data streams"
)

find_package(PkgConfig)
if (PkgConfig_FOUND)
    pkg_check_modules(LibZstd IMPORTED_TARGET "libzstd")
endif()
add_feature_info(LibZstd LibZstd_FOUND
                "Support for zstd compressed files and data streams"
)

include(ECMSetupVersion)
include(ECMGenerateHeaders)
include(ECMQtDeclareLoggingCategory)
include(ECMAddQch)
include(ECMDeprecationSettings)
include(ECMPoQmTools)

set(EXCLUDE_DEPRECATED_BEFORE_AND_AT 0 CACHE STRING "Control the range of deprecated API excluded from the build [default=0].")

option(BUILD_QCH "Build API documentation in QCH format (for e.g. Qt Assistant, Qt Creator & KDevelop)" OFF)
add_feature_info(QCH ${BUILD_QCH} "API documentation in QCH format (for e.g. Qt Assistant, Qt Creator & KDevelop)")

ecm_setup_version(PROJECT
    VARIABLE_PREFIX KARCHIVE
    VERSION_HEADER "${CMAKE_CURRENT_BINARY_DIR}/karchive_version.h"
    PACKAGE_VERSION_FILE "${CMAKE_CURRENT_BINARY_DIR}/KF5ArchiveConfigVersion.cmake"
    SOVERSION 5)

ecm_set_disabled_deprecation_versions(
    QT 5.15.2
)


add_subdirectory(src)
if (BUILD_TESTING)
    add_subdirectory(autotests)
    add_subdirectory(tests)
endif()

ecm_install_po_files_as_qm(poqm)

# create a Config.cmake and a ConfigVersion.cmake file and install them
set(CMAKECONFIG_INSTALL_DIR "${KDE_INSTALL_CMAKEPACKAGEDIR}/KF5Archive")

if (BUILD_QCH)
    ecm_install_qch_export(
        TARGETS KF5Archive_QCH
        FILE KF5ArchiveQchTargets.cmake
        DESTINATION "${CMAKECONFIG_INSTALL_DIR}"
        COMPONENT Devel
    )
    set(PACKAGE_INCLUDE_QCHTARGETS "include(\"\${CMAKE_CURRENT_LIST_DIR}/KF5ArchiveQchTargets.cmake\")")
endif()

include(CMakePackageConfigHelpers)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/KF5ArchiveConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/KF5ArchiveConfig.cmake"
    INSTALL_DESTINATION ${CMAKECONFIG_INSTALL_DIR}
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/karchive_version.h
        DESTINATION ${KDE_INSTALL_INCLUDEDIR_KF}/KArchive
        COMPONENT Devel)

install(FILES
            "${CMAKE_CURRENT_BINARY_DIR}/KF5ArchiveConfig.cmake"
            "${CMAKE_CURRENT_BINARY_DIR}/KF5ArchiveConfigVersion.cmake"
        DESTINATION "${CMAKECONFIG_INSTALL_DIR}"
        COMPONENT Devel)

install(EXPORT KF5ArchiveTargets
        DESTINATION "${CMAKECONFIG_INSTALL_DIR}"
        FILE KF5ArchiveTargets.cmake
        NAMESPACE KF5::)

feature_summary(WHAT ALL   FATAL_ON_MISSING_REQUIRED_PACKAGES)

kde_configure_git_pre_commit_hook(CHECKS CLANG_FORMAT)
