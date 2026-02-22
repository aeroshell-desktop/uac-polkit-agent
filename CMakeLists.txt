cmake_minimum_required(VERSION 3.16)

project(uac-polkit-agent)
set(PROJECT_VERSION "6.5.5")

set(QT_MIN_VERSION "6.9.0")
set(KF6_MIN_VERSION "6.17.0")
set(KDE_COMPILERSETTINGS_LEVEL "5.82")

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(SEVENLIKE true)

# we need some parts of the ECM CMake helpers
find_package(ECM ${KF6_MIN_VERSION} REQUIRED NO_MODULE)
set (CMAKE_MODULE_PATH ${ECM_MODULE_PATH})

include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings NO_POLICY_SCOPE)
include(KDEClangFormat)
include(FeatureSummary)
include(ECMConfiguredInstall)
include(ECMDeprecationSettings)

include(KDEGitCommitHooks)

find_package(Qt6 ${QT_MIN_VERSION} CONFIG REQUIRED Core DBus Widgets Quick Qml)

# Load the frameworks we need
find_package(KF6 ${KF6_MIN_VERSION} REQUIRED COMPONENTS
  I18n
  WindowSystem
  Notifications
  DBusAddons
  CoreAddons
  Crash
)

include(CheckIncludeFile)
include(CheckSymbolExists)
check_include_file("sys/prctl.h" HAVE_SYS_PRCTL_H)
check_symbol_exists(PR_SET_DUMPABLE "sys/prctl.h" HAVE_PR_SET_DUMPABLE)
check_include_file("sys/procctl.h" HAVE_SYS_PROCCTL_H)
check_symbol_exists(PROC_TRACE_CTL "sys/procctl.h" HAVE_PROC_TRACE_CTL)
if (HAVE_PR_SET_DUMPABLE OR HAVE_PROC_TRACE_CTL)
  set(CAN_DISABLE_PTRACE TRUE)
endif ()
add_feature_info("prctl/procctl tracing control"
                 CAN_DISABLE_PTRACE
                 "Required for disallowing ptrace on polkit agent")

# create config file
configure_file (config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h)

find_package(PolkitQt6-1 REQUIRED 0.103.0)
ecm_set_disabled_deprecation_versions(QT 6.8.1
    KF 6.12.0
)

add_definitions(-DQT_NO_KEYWORDS)

qt_add_dbus_adaptor(policykit_SRCS org.kde.Polkit1AuthAgent.xml policykitlistener.h PolicyKitListener)

set(policykit_SRCS
    ${policykit_SRCS}
    policykitlistener.cpp
    main.cpp
    QuickAuthDialog.cpp
    QuickAuthDialog.qrc
    IdentitiesModel.cpp
)

add_executable(uac-polkit-agent ${policykit_SRCS})

target_link_libraries(uac-polkit-agent
    KF6::DBusAddons
    KF6::WindowSystem
    KF6::CoreAddons
    KF6::I18n
    KF6::Crash
    KF6::I18nQml
    KF6::Notifications
    Qt::Qml
    Qt::Quick
    Qt::Widgets
    PolkitQt6-1::Agent
)

configure_file(uac-polkit-agent.desktop.cmake ${CMAKE_BINARY_DIR}/uac-polkit-agent.desktop)

install(TARGETS uac-polkit-agent DESTINATION ${KDE_INSTALL_LIBDIR})

install(FILES ${CMAKE_BINARY_DIR}/uac-polkit-agent.desktop DESTINATION ${KDE_INSTALL_AUTOSTARTDIR})
install(FILES ${CMAKE_BINARY_DIR}/uac-polkit-agent.desktop DESTINATION ${KDE_INSTALL_APPDIR} RENAME io.gitgud.wackyideas.uac-polkit-agent.desktop)

ecm_install_configured_files(INPUT uac-polkit-agent.conf.in DESTINATION ${KDE_INSTALL_SYSCONFDIR}/systemd/user/plasma-polkit-agent.service.d/)
ecm_install_configured_files(INPUT uac-polkit-agent.service.in DESTINATION ${KDE_INSTALL_SYSTEMDUSERUNITDIR})

install(FILES uac-polkit-agent.notifyrc DESTINATION ${KDE_INSTALL_KNOTIFYRCDIR})

# add clang-format target for all our real source files
file(GLOB_RECURSE ALL_CLANG_FORMAT_SOURCE_FILES *.cpp *.h)
kde_clang_format(${ALL_CLANG_FORMAT_SOURCE_FILES})

kde_configure_git_pre_commit_hook(CHECKS CLANG_FORMAT)
feature_summary(WHAT ALL FATAL_ON_MISSING_REQUIRED_PACKAGES)

ki18n_install(po)
