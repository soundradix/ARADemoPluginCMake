cmake_minimum_required(VERSION 3.22.1)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

if(APPLE)
    if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
        execute_process(COMMAND xcrun --sdk macosx --show-sdk-path OUTPUT_VARIABLE CMAKE_OSX_SYSROOT)
    endif()

    set(CMAKE_XCODE_ATTRIBUTE_ARCHS "$(ARCHS_STANDARD)")
    set(CMAKE_XCODE_ATTRIBUTE_ONLY_ACTIVE_ARCH $<IF:$<CONFIG:Debug>,YES,NO>)
    set(CMAKE_OSX_DEPLOYMENT_TARGET "10.13" CACHE STRING "Minimum OS X deployment version")
endif()

if(MSVC)
    set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
endif()

project(ARAPluginDemo VERSION 1.0.0)

set(JUCE_ENABLE_MODULE_SOURCE_GROUPS ON CACHE BOOL INTERNAL)
set_property(GLOBAL PROPERTY USE_FOLDERS YES)

include(cmake/CPM.cmake)
CPMAddPackage(
    GITHUB_REPOSITORY "soundradix/JUCE"
    GIT_TAG "7a1801baeca526c4c45d6bd21e8160bc0f31fb97"
    OPTIONS "JUCE_ENABLE_MODULE_SOURCE_GROUPS ON"
)
CPMAddPackage(
    NAME "ARA_SDK"
    GITHUB_REPOSITORY "celemony/ARA_SDK"
    GIT_TAG "78bd940807bd7e318785fabe689fba73f8d9098e"
    DOWNLOAD_ONLY ON
)

set(AAX_SDK_PATH aax_sdk)
if(EXISTS "${AAX_SDK_PATH}")
    set(HAS_AAX TRUE)
    juce_set_aax_sdk_path("${AAX_SDK_PATH}")
else()
    set(HAS_AAX FALSE)
    message(STATUS "AAX SDK not found. AAX format will be excluded.")
endif()

juce_set_ara_sdk_path("${ARA_SDK_SOURCE_DIR}")

juce_add_plugin(ARAPluginDemo
    COMPANY_NAME "JUCE"
    IS_SYNTH FALSE
    NEEDS_MIDI_INPUT FALSE
    NEEDS_MIDI_OUTPUT FALSE
    IS_MIDI_EFFECT FALSE
    IS_ARA_EFFECT TRUE
    MICROPHONE_PERMISSION_ENABLED TRUE
    MICROPHONE_PERMISSION_TEXT "We use audio input in standalone JUCE app"
    COPY_PLUGIN_AFTER_BUILD TRUE
    PLUGIN_MANUFACTURER_CODE Manu
    PLUGIN_CODE AraD
    FORMATS Standalone $<HAS_AAX:AAX> AU AUv3 VST3
    PRODUCT_NAME "ARAPluginDemo"
)

juce_generate_juce_header(ARAPluginDemo)


#TODO: This should be within JUCE itself!
add_library(ARA_IPC_Library ${ARA_LIBRARY_TARGET_TYPE}
    "${ARA_SDK_SOURCE_DIR}/ARA_Library/Dispatch/ARAHostDispatch.cpp"
    "${ARA_SDK_SOURCE_DIR}/ARA_Library/IPC/ARAIPC.h"
    "${ARA_SDK_SOURCE_DIR}/ARA_Library/IPC/ARAIPCEncoding.h"
    "${ARA_SDK_SOURCE_DIR}/ARA_Library/IPC/ARAIPCMessage.h"
    "${ARA_SDK_SOURCE_DIR}/ARA_Library/IPC/ARAIPCMessageChannel.h"
    "${ARA_SDK_SOURCE_DIR}/ARA_Library/IPC/ARAIPCMessageChannel.cpp"
    "${ARA_SDK_SOURCE_DIR}/ARA_Library/IPC/ARAIPCProxyHost.h"
    "${ARA_SDK_SOURCE_DIR}/ARA_Library/IPC/ARAIPCProxyHost.cpp"
    "${ARA_SDK_SOURCE_DIR}/ARA_Library/IPC/ARAIPCProxyPlugIn.cpp"
    "${ARA_SDK_SOURCE_DIR}/ARA_Library/IPC/ARAIPCProxyPlugIn.h"
)

if(APPLE)
    target_sources(ARA_IPC_Library PRIVATE
        "${ARA_SDK_SOURCE_DIR}/ARA_Library/IPC/ARAIPCAudioUnit_v3.h"
        "${ARA_SDK_SOURCE_DIR}/ARA_Library/IPC/ARAIPCAudioUnit_v3.mm"
        "${ARA_SDK_SOURCE_DIR}/ARA_Library/IPC/ARAIPCCFEncoding.h"
        "${ARA_SDK_SOURCE_DIR}/ARA_Library/IPC/ARAIPCCFEncoding.cpp"
    )
    target_link_libraries(ARA_IPC_Library PRIVATE
        "-framework Cocoa"
    )
    target_include_directories(ARA_IPC_Library
    PRIVATE
    "${ARA_SDK_SOURCE_DIR}"
)
endif()

target_include_directories(ARAPluginDemo
    PRIVATE
    Source
)

target_sources(ARAPluginDemo
    PRIVATE
    Source/ARAPluginDemo.h
    Source/Main.cpp
)

target_compile_definitions(ARAPluginDemo
    PUBLIC
    DONT_SET_USING_JUCE_NAMESPACE=0
    JUCE_WEB_BROWSER=0
    JUCE_USE_CURL=0
    JUCE_VST3_CAN_REPLACE_VST2=0
    JUCE_DISPLAY_SPLASH_SCREEN=0
)

# juce_add_binary_data(AudioPluginData SOURCES ...)
target_link_libraries(ARAPluginDemo
    PRIVATE
    ARA_IPC_Library
    # AudioPluginData           # If we'd created a binary data target, we'd link to it here
    juce::juce_audio_utils
    PUBLIC
    juce::juce_recommended_config_flags
    juce::juce_recommended_lto_flags
    juce::juce_recommended_warning_flags)
