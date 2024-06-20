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
    GIT_TAG "8455e1031ee264693de30e790867c96abbe87dd3"
    OPTIONS "JUCE_ENABLE_MODULE_SOURCE_GROUPS ON"
)
CPMAddPackage(
    NAME "ARA_SDK"
    GITHUB_REPOSITORY "celemony/ARA_SDK"
    GIT_TAG "ab7a7cd6a26ceb12b940f7630cc3088ff0a41201"
    DOWNLOAD_ONLY ON
)

juce_set_aax_sdk_path(aax_sdk)
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
    FORMATS AAX AU VST3
    PRODUCT_NAME "ARAPluginDemo"
)

juce_generate_juce_header(ARAPluginDemo)

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

    # AudioPluginData           # If we'd created a binary data target, we'd link to it here
    juce::juce_audio_utils
    PUBLIC
    juce::juce_recommended_config_flags
    juce::juce_recommended_lto_flags
    juce::juce_recommended_warning_flags)
