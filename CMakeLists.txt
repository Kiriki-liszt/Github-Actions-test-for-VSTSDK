cmake_minimum_required(VERSION 3.14.0)
set(CMAKE_OSX_DEPLOYMENT_TARGET 10.13 CACHE STRING "")

if(SMTG_MAC)
    set(vst3sdk_SOURCE_DIR "/Users/yoon/VST_SDK/vst3sdk")
elseif(SMTG_WIN)
    set(vst3sdk_SOURCE_DIR "C:/VST_SDK/vst3sdk")
endif(SMTG_MAC)

if(GITHUB_ACTIONS) 
    set(vst3sdk_SOURCE_DIR ${CMAKE_SOURCE_DIR}/vst3sdk)
endif()

if(NOT vst3sdk_SOURCE_DIR)
    message(FATAL_ERROR "Path to VST3 SDK is empty!")
endif()

project(JS_Inflator
    # This is your plug-in version number. Change it here only.
    # Version number symbols usable in C++ can be found in
    # source/version.h and ${PROJECT_BINARY_DIR}/projectversion.h.
    VERSION 2.0.3.0
    DESCRIPTION "JS Inflator VST 3 Plug-in"
)

set(SMTG_VSTGUI_ROOT "${vst3sdk_SOURCE_DIR}")

add_subdirectory(${vst3sdk_SOURCE_DIR} ${PROJECT_BINARY_DIR}/vst3sdk)
smtg_enable_vst3_sdk()

smtg_add_vst3plugin(JS_Inflator
    source/version.h
    source/JSIF_cids.h
    source/JSIF_shared.h
    source/JSIF_dataexchange.h
    source/JSIF_processor.h
    source/JSIF_processor.cpp
    source/JSIF_controller.h
    source/JSIF_controller.cpp
    source/JSIF_entry.cpp
)

#- VSTGUI Wanted ----
if(SMTG_ENABLE_VSTGUI_SUPPORT)
    target_sources(JS_Inflator
        PRIVATE
            resource/JSIF_editor.uidesc
    )
    smtg_target_add_plugin_resources(JS_Inflator
        RESOURCES
            "resource/JSIF_editor.uidesc"
    )
    smtg_target_add_plugin_resources(JS_Inflator
        RESOURCES
            "resource/orig_background.png"
            "resource/orig_button_onoff.png"
            "resource/orig_handle.png"
            "resource/orig_knob_green_.png"
            "resource/orig_knob_red_.png"
            "resource/orig_LED_cover.png"
            "resource/orig_LED_ON.png"
            "resource/orig_LED_OFF.png"
            "resource/orig_LED_FX_ON.png"
            "resource/orig_LED_FX_OFF.png"
    )
    smtg_target_add_plugin_resources(JS_Inflator
        RESOURCES
            "resource/twarch/twarch_background.png"
            "resource/twarch/twarch_btn_CLIP.png"
            "resource/twarch/twarch_btn_IN.png"
            "resource/twarch/twarch_btn_SPLIT.png"
            "resource/twarch/twarch_handle.png"
            "resource/twarch/twarch_knob_red_.png"
            "resource/twarch/twarch_knob_yellow_.png"
            "resource/twarch/twarch_meter_off.png"
            "resource/twarch/twarch_meter_on.png"
            "resource/twarch/twarch_meter_fx_on.png"
    )
    target_link_libraries(JS_Inflator
        PRIVATE
            vstgui_support
    )
endif(SMTG_ENABLE_VSTGUI_SUPPORT)
# -------------------

smtg_target_add_plugin_snapshots (JS_Inflator
    RESOURCES
        resource/A6B5CA9F4C4F5B9388F837774504BD37_snapshot.png
        resource/A6B5CA9F4C4F5B9388F837774504BD37_snapshot_2.0x.png
)

target_link_libraries(JS_Inflator
    PRIVATE
        sdk
)

set(R8B_PATH "${PROJECT_BINARY_DIR}/../libs/r8brain-free-src" CACHE STRING "Path to r8brain-free-src library source tree")

file(WRITE "${R8B_PATH}/CMakeLists.txt"
           [=[
project(r8brain-free-src)
add_library(r8brain-free-src r8bbase.cpp)
]=]
)
    
add_subdirectory(${R8B_PATH} ${PROJECT_BINARY_DIR}/r8brain-free-src-build EXCLUDE_FROM_ALL)
include_directories(${R8B_PATH})
target_link_libraries(JS_Inflator PUBLIC r8brain-free-src)
smtg_target_setup_universal_binary(r8brain-free-src)

smtg_target_configure_version_file(JS_Inflator)

if(SMTG_MAC)
    smtg_target_set_bundle(JS_Inflator
        BUNDLE_IDENTIFIER io.github.yg331.JS.Inflator
        COMPANY_NAME "yg331"
    )
    smtg_target_set_debug_executable(JS_Inflator
        "/Applications/VST3PluginTestHost.app"
        "--pluginfolder;$(BUILT_PRODUCTS_DIR)"
    )
elseif(SMTG_WIN)
    target_sources(JS_Inflator PRIVATE 
        resource/win32resource.rc
    )
    if(MSVC)
        set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT JS_Inflator)

        smtg_target_set_debug_executable(JS_Inflator
            "$(ProgramW6432)/Steinberg/VST3PluginTestHost/VST3PluginTestHost.exe"
            "--pluginfolder \"$(OutDir)/\""
        )
    endif()
endif(SMTG_MAC)

if(SMTG_WIN)
    if(NOT MINGW)
        add_definitions(-D_CRT_SECURE_NO_WARNINGS)
    endif(NOT MINGW)
endif(SMTG_WIN)

# Add an AUv2 target
if (SMTG_MAC AND XCODE AND SMTG_COREAUDIO_SDK_PATH)
	smtg_target_add_auv2(JS_Inflator-au
		BUNDLE_NAME JS_Inflator
		BUNDLE_IDENTIFIER io.github.yg331.JS.Inflator.audiounit
		INFO_PLIST_TEMPLATE resource/au-info.plist
		VST3_PLUGIN_TARGET JS_Inflator)
endif(SMTG_MAC AND XCODE AND SMTG_COREAUDIO_SDK_PATH)
