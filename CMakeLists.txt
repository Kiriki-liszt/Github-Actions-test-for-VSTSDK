cmake_minimum_required(VERSION 3.14.0)
set(CMAKE_OSX_DEPLOYMENT_TARGET 10.13 CACHE STRING "")

set(vst3sdk_SOURCE_DIR "./VST_SDK/vst3sdk")
if(NOT vst3sdk_SOURCE_DIR)
    message(FATAL_ERROR "Path to VST3 SDK is empty!")
endif()

project(Github_Actions_test_for_VSTSDK
    # This is your plug-in version number. Change it here only.
    # Version number symbols usable in C++ can be found in
    # source/version.h and ${PROJECT_BINARY_DIR}/projectversion.h.
    VERSION 1.0.0.0 
    DESCRIPTION "Github_Actions_test_for_VSTSDK VST 3 Plug-in"
)

set(SMTG_VSTGUI_ROOT "${vst3sdk_SOURCE_DIR}")

add_subdirectory(${vst3sdk_SOURCE_DIR} ${PROJECT_BINARY_DIR}/vst3sdk)
smtg_enable_vst3_sdk()

smtg_add_vst3plugin(Github_Actions_test_for_VSTSDK
    source/version.h
    source/GAtfV_cids.h
    source/GAtfV_processor.h
    source/GAtfV_processor.cpp
    source/GAtfV_controller.h
    source/GAtfV_controller.cpp
    source/GAtfV_entry.cpp
)

#- VSTGUI Wanted ----
if(SMTG_ENABLE_VSTGUI_SUPPORT)
    target_sources(Github_Actions_test_for_VSTSDK
        PRIVATE
            resource/GAtfV_editor.uidesc
    )
    target_link_libraries(Github_Actions_test_for_VSTSDK
        PRIVATE
            vstgui_support
    )
    smtg_target_add_plugin_resources(Github_Actions_test_for_VSTSDK
        RESOURCES
            "resource/GAtfV_editor.uidesc"
    )
endif(SMTG_ENABLE_VSTGUI_SUPPORT)
# -------------------

smtg_target_add_plugin_snapshots (Github_Actions_test_for_VSTSDK
    RESOURCES
        resource/EEEF404F505254EC84BD143829DE3A20_snapshot.png
        resource/EEEF404F505254EC84BD143829DE3A20_snapshot_2.0x.png
)

target_link_libraries(Github_Actions_test_for_VSTSDK
    PRIVATE
        sdk
)

smtg_target_configure_version_file(Github_Actions_test_for_VSTSDK)

if(SMTG_MAC)
    smtg_target_set_bundle(Github_Actions_test_for_VSTSDK
        BUNDLE_IDENTIFIER Github.Actions.test.for.VSTSDK
        COMPANY_NAME "yg331"
    )
    smtg_target_set_debug_executable(Github_Actions_test_for_VSTSDK
        "/Applications/VST3PluginTestHost.app"
        "--pluginfolder;$(BUILT_PRODUCTS_DIR)"
    )
elseif(SMTG_WIN)
    target_sources(Github_Actions_test_for_VSTSDK PRIVATE 
        resource/win32resource.rc
    )
    if(MSVC)
        set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT Github_Actions_test_for_VSTSDK)

        smtg_target_set_debug_executable(Github_Actions_test_for_VSTSDK
            "$(ProgramW6432)/Steinberg/VST3PluginTestHost/VST3PluginTestHost.exe"
            "--pluginfolder \"$(OutDir)/\""
        )
    endif()
endif(SMTG_MAC)

# Add an AUv2 target
if (SMTG_MAC AND XCODE AND SMTG_COREAUDIO_SDK_PATH)
	smtg_target_add_auv2(Github_Actions_test_for_VSTSDK-au
		BUNDLE_NAME Github_Actions_test_for_VSTSDK
		BUNDLE_IDENTIFIER Github.Actions.test.for.VSTSDK.audiounit
		INFO_PLIST_TEMPLATE resource/au-info.plist
		VST3_PLUGIN_TARGET Github_Actions_test_for_VSTSDK)
endif(SMTG_MAC AND XCODE AND SMTG_COREAUDIO_SDK_PATH)
