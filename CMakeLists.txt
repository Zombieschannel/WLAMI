cmake_minimum_required (VERSION 3.15)
project (Project)
set(APP_NAME "Project")
set(APP_VERSION "1.0.0")
set(APP_IDENTIFIER "com.Username.App")

if (CMAKE_SYSTEM_NAME STREQUAL "Windows")
    set(WINDOWS TRUE)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    set(LINUX TRUE)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    set(MACOS TRUE)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Android")
    set(ANDROID TRUE)
elseif(CMAKE_SYSTEM_NAME STREQUAL "iOS")
    set(IOS TRUE)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Emscripten")
    set(EMSCRIPTEN TRUE)
endif()

#SFML SHARED LIBRARY
if (WINDOWS OR IOS OR EMSCRIPTEN)
    set(SFML_STATIC_LIBRARIES TRUE)
elseif(LINUX OR ANDROID OR MACOS)
    set(SFML_STATIC_LIBRARIES FALSE)
endif()

####################################################
# SET YOUR SFML DIR FOR EACH PLATFORM/ARCHITECTURE #
####################################################
if (WINDOWS)
    if (${BUILD_ARCH} STREQUAL "x86")
        set(SFML_ROOT "<insert>/SFML-3.0.0-x86")
    elseif(${BUILD_ARCH} STREQUAL "x64")
        set(SFML_ROOT "<insert>/SFML-3.0.0-x64")
    elseif(${BUILD_ARCH} STREQUAL "ARM64")
        set(SFML_ROOT "<insert>/SFML-3.0.0-ARM64")
    endif()
elseif(LINUX)
    set(SFML_ROOT "<insert>/SFML-3.0.0-Linux")
elseif(MACOS)
    set(SFML_ROOT "<insert>/SFML-3.0.0-MacOS")
elseif(IOS)
    set(SFML_ROOT "<insert>/SFML-3.0.0-iOS")
elseif(EMSCRIPTEN)
    set(SFML_ROOT "<insert>/SFML-3.0.0-Emscripten")
endif()

#FILES
file(GLOB_RECURSE Project_SOURCES "src/*.cpp")
if (NOT EXISTS "assets/")
    file(MAKE_DIRECTORY "assets/")
endif()
if (NOT EXISTS "versions/")
    file(MAKE_DIRECTORY "versions/")
endif()

if (WINDOWS OR LINUX)
    file(COPY "assets/" DESTINATION "${CMAKE_BINARY_DIR}")
endif()

if (WINDOWS)

    #enable_language("RC") #uncoment if using icons
    #set (WIN32_RESOURCES "src/icon.rc")

    find_package(SFML 3 COMPONENTS Graphics Audio Window System Network REQUIRED)

    add_executable (${CMAKE_PROJECT_NAME} ${Project_SOURCES} #[[${WIN32_RESOURCES}]])
    set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES LINK_FLAGS_DEBUG "/SUBSYSTEM:CONSOLE")
    set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES LINK_FLAGS_RELEASE "/SUBSYSTEM:WINDOWS /ENTRY:mainCRTStartup")

elseif(LINUX)
    
    find_package(SFML 3 COMPONENTS Graphics Audio Window System Network REQUIRED)
    add_executable (${CMAKE_PROJECT_NAME} ${Project_SOURCES})

elseif(ANDROID)

    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -u ANativeActivity_onCreate")
    set(CMAKE_MAP_IMPORTED_CONFIG_RELWITHDEBINFO RELEASE)
    set(PATH_TO_SFML ${ANDROID_NDK}/sources/third_party/sfml)
    set(SFML_DIR ${PATH_TO_SFML}/lib/${ANDROID_ABI}/cmake/SFML)
    find_package(SFML 3 COMPONENTS Graphics Audio Window System Network REQUIRED)

    add_library(${CMAKE_PROJECT_NAME} SHARED ${Project_SOURCES})

elseif(MACOS)

    find_package(SFML 3 COMPONENTS Graphics Audio Window System Network REQUIRED)

    set(ICON_NAME "AppLogo.icns")
    set(ICON_PATH "${CMAKE_SOURCE_DIR}/src/${ICON_NAME}")

    add_executable(${CMAKE_PROJECT_NAME} MACOSX_BUNDLE ${Project_SOURCES} ${ICON_PATH} ${Whereami_SOURCES})
    set_source_files_properties( ${ICON_PATH} PROPERTIES MACOSX_PACKAGE_LOCATION Resources)

    set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES
        BUNDLE True
        MACOSX_BUNDLE_ICON_FILE ${ICON_NAME}
        MACOSX_BUNDLE_GUI_IDENTIFIER ${APP_IDENTIFIER}
        MACOSX_BUNDLE_BUNDLE_NAME ${APP_NAME}
        MACOSX_BUNDLE_BUNDLE_VERSION ${APP_VERSION}
        MACOSX_BUNDLE_SHORT_VERSION_STRING ${APP_VERSION}
        MACOSX_BUNDLE_INFO_PLIST ${CMAKE_SOURCE_DIR}/MacOSXBundleInfo.plist.in
    )

    add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.app/Contents/Libs"
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        "${SFML_LOC}/lib/libsfml-audio.2.6.dylib" "${SFML_LOC}/lib/libsfml-audio-d.2.6.dylib"
        "${SFML_LOC}/lib/libsfml-system.2.6.dylib" "${SFML_LOC}/lib/libsfml-system-d.2.6.dylib"
        "${SFML_LOC}/lib/libsfml-window.2.6.dylib" "${SFML_LOC}/lib/libsfml-window-d.2.6.dylib"
        "${SFML_LOC}/lib/libsfml-graphics.2.6.dylib" "${SFML_LOC}/lib/libsfml-graphics-d.2.6.dylib"
        "${SFML_LOC}/lib/libsfml-network.2.6.dylib" "${SFML_LOC}/lib/libsfml-network-d.2.6.dylib"
        "${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.app/Contents/Libs"
        COMMENT "Copying dylibs to the app bundle..."
    )

    add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.app/Contents/Frameworks"
        COMMAND ${CMAKE_COMMAND} -E copy_directory "${SFML_LOC}/Frameworks" "${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.app/Contents/Frameworks"
	    COMMENT "Copying frameworks to target location..."
    )

    add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
        COMMAND install_name_tool -add_rpath "@loader_path/../Libs" "$<TARGET_FILE:${CMAKE_PROJECT_NAME}>"
        COMMENT "Fixing dylib paths in the executable..."
    )

    add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_SOURCE_DIR}/assets" "${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.app/Contents/Resources"
	COMMENT "Copying assets to target location..."
    )

elseif(IOS)

    set(Freetype_DIR ${SFML_ROOT}/lib/cmake/freetype)
    set(Vorbis_DIR ${SFML_ROOT}/lib/cmake/Vorbis)
    set(Ogg_DIR ${SFML_ROOT}/lib/cmake/Ogg)
    set(FLAC_DIR ${SFML_ROOT}/lib/cmake/FLAC)
    set(SFML_DIR ${SFML_ROOT}/lib/cmake/SFML)
    find_package(SFML 3 COMPONENTS Graphics Audio Window System Network REQUIRED)

    set(LAUNCH_SCREEN "${CMAKE_SOURCE_DIR}/LaunchScreen.storyboard")
    set(ICON_NAME "AppLogo.icns")
    set(ICON_PATH "${CMAKE_SOURCE_DIR}/src/${ICON_NAME}")

    add_executable(${CMAKE_PROJECT_NAME} MACOSX_BUNDLE ${Project_SOURCES} ${ICON_PATH} ${Whereami_SOURCES} ${LAUNCH_SCREEN} ${CMAKE_SOURCE_DIR}/assets)
    set_source_files_properties( ${ICON_PATH} PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
    set_source_files_properties( ${LAUNCH_SCREEN} PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
    set_source_files_properties("${CMAKE_SOURCE_DIR}/assets" PROPERTIES MACOSX_PACKAGE_LOCATION Resources)

    set(RESOURCES ${LAUNCH_SCREEN} "${CMAKE_SOURCE_DIR}/assets")
    set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES
        BUNDLE True
        MACOSX_BUNDLE_ICON_FILE ${ICON_NAME}
	    RESOURCE "${RESOURCES}"
        MACOSX_BUNDLE_GUI_IDENTIFIER ${APP_IDENTIFIER}
        MACOSX_BUNDLE_BUNDLE_NAME ${APP_NAME}
        MACOSX_BUNDLE_BUNDLE_VERSION ${APP_VERSION}
        MACOSX_BUNDLE_SHORT_VERSION_STRING ${APP_VERSION}
        MACOSX_BUNDLE_INFO_PLIST ${CMAKE_SOURCE_DIR}/MacOSXBundleInfo.plist.in
    )

    set_property (TARGET ${CMAKE_PROJECT_NAME} PROPERTY XCODE_ATTRIBUTE_CLANG_CXX_LIBRARY "libc++")

elseif(EMSCRIPTEN)

    set(SFML_EMSCRIPTEN_TARGET_COMPILE_OPTIONS_DEBUG -g3 -gsource-map -O0)

    set(SFML_EMSCRIPTEN_TARGET_COMPILE_OPTIONS_RELEASE -O3)

    set(SFML_EMSCRIPTEN_TARGET_COMPILE_OPTIONS
        $<$<CONFIG:Debug>:${SFML_EMSCRIPTEN_TARGET_COMPILE_OPTIONS_DEBUG}>
        $<$<CONFIG:Release>:${SFML_EMSCRIPTEN_TARGET_COMPILE_OPTIONS_RELEASE}>
        #-pthread
    )

    set(SFML_EMSCRIPTEN_TARGET_LINK_OPTIONS_DEBUG -g3 -gsource-map -sASSERTIONS=2 -sCHECK_NULL_WRITES=1 -sSAFE_HEAP=1 -sSTACK_OVERFLOW_CHECK=1)

    set(SFML_EMSCRIPTEN_TARGET_LINK_OPTIONS_RELEASE -O3)

    set(SFML_EMSCRIPTEN_TARGET_LINK_OPTIONS_ASYNCIFY -sASYNCIFY=1)

    set(SFML_EMSCRIPTEN_TARGET_LINK_OPTIONS
        $<$<CONFIG:Debug>:${SFML_EMSCRIPTEN_TARGET_LINK_OPTIONS_DEBUG}>
        $<$<CONFIG:Release>:${SFML_EMSCRIPTEN_TARGET_LINK_OPTIONS_RELEASE}>

        ${SFML_EMSCRIPTEN_TARGET_LINK_OPTIONS_ASYNCIFY}

        -Wno-limited-postlink-optimizations # warning: running limited binaryen optimizations because DWARF info requested (or indirectly required)
        -Wno-pthreads-mem-growth            # warning: -pthread + ALLOW_MEMORY_GROWTH may run non-wasm code slowly, see https://github.com/WebAssembly/design/issues/1271

        -sALLOW_MEMORY_GROWTH=1             # Grow the memory arrays at runtime
        -sEXIT_RUNTIME=1                    # Execute cleanup (e.g. `atexit`) after `main` completes
        -sFETCH=1                           # Enables `emscripten_fetch` API
        -sFORCE_FILESYSTEM=1                # Makes full filesystem support be included
        -sFULL_ES2=1                        # Forces support for all GLES2 features, not just the WebGL-friendly subset
        -sMAX_WEBGL_VERSION=1               # Specifies the highest WebGL version to target
        -sMIN_WEBGL_VERSION=1               # Specifies the lowest WebGL version to target
        -sSTACK_SIZE=4mb                    # Set the total stack size
        #-sUSE_PTHREADS=1                    # Enable threading support
        -sWASM=1                            # Compile code to WebAssembly

        -sGL_EXPLICIT_UNIFORM_LOCATION=1  # TODO P0:
        -sGL_EXPLICIT_UNIFORM_BINDING=1   # TODO P0:

        --emrun                             # Add native support for `emrun` (I/O capture)
        --embed-file ${CMAKE_SOURCE_DIR}/assets@/
        -sTOTAL_MEMORY=32mb

        --shell-file=${CMAKE_SOURCE_DIR}/shell.html
    )
    set(CMAKE_EXECUTABLE_SUFFIX ".html")

    set(Freetype_DIR ${SFML_ROOT}/lib/cmake/freetype)
    set(Vorbis_DIR ${SFML_ROOT}/lib/cmake/Vorbis)
    set(Ogg_DIR ${SFML_ROOT}/lib/cmake/Ogg)
    set(FLAC_DIR ${SFML_ROOT}/lib/cmake/FLAC)
    set(SFML_DIR ${SFML_ROOT}/lib/cmake/SFML)
    find_package(SFML 3 COMPONENTS Graphics Audio Window System Network REQUIRED)

    add_executable (${CMAKE_PROJECT_NAME} ${Project_SOURCES})
    target_compile_options(${CMAKE_PROJECT_NAME} PRIVATE ${SFML_EMSCRIPTEN_TARGET_COMPILE_OPTIONS})
    target_link_options(${CMAKE_PROJECT_NAME} PRIVATE ${SFML_EMSCRIPTEN_TARGET_LINK_OPTIONS})

endif()

#C++ version
target_compile_features(${CMAKE_PROJECT_NAME} PRIVATE cxx_std_17)

if (ANDROID OR IOS)
    target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE SFML_MOBILE)
else()
    target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE SFML_DESKTOP)
endif()

if (ANDROID OR IOS OR EMSCRIPTEN)
    target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE SFML_OPENGL_ES)
endif()

#uncomment if you'd like to check which architecture is active on Windows from the c++ side
#[[
if (WINDOWS)
    if (${BUILD_ARCH} STREQUAL "x86")
        target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE SFML_X86)
    elseif(${BUILD_ARCH} STREQUAL "x64")
        target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE SFML_X64)
    elseif(${BUILD_ARCH} STREQUAL "ARM64")
        target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE SFML_ARM64)
    endif()
endif()
]]

set (LINK_SFML SFML::System SFML::Window SFML::Graphics SFML::Audio SFML::Network)

if (WINDOWS)
    target_link_libraries(${CMAKE_PROJECT_NAME} ${LINK_SFML} SFML::Main)
elseif(LINUX)
    target_link_libraries(${CMAKE_PROJECT_NAME} ${LINK_SFML} -lGL -lX11)
elseif(ANDROID)
    target_link_libraries(${CMAKE_PROJECT_NAME}
        log
        android
        ${LINK_SFML}
        SFML::Main
        EGL
        GLESv1_CM
    )
elseif(MACOS)
    target_link_libraries(${CMAKE_PROJECT_NAME} ${LINK_SFML} "-framework OpenGL")
elseif(IOS)
    target_link_libraries(${CMAKE_PROJECT_NAME} ${LINK_SFML})
elseif(EMSCRIPTEN)
    target_link_libraries(${CMAKE_PROJECT_NAME} ${LINK_SFML})
endif()
