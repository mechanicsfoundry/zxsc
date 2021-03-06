cmake_minimum_required(VERSION 3.8)

set(PROJECT_NAME zxsc)
set(PROJECT_files_NAME zxsc-files)

project(${PROJECT_NAME})

set(CMAKE_CXX_STANDARD 14)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

option(EMSCRIPTEN "Web Compilation" OFF)

set(SOURCES
    "src/Main.cpp"
    "src/GUI.cpp")

set(HEADERS
    "src/Main.hpp"
    "src/GUI.hpp")

set(SOURCES_SDL
    src/sdl/SDL.cpp
    src/sdl/SDLMain.cpp
    src/sdl/SDLMainWeb.cpp
    src/sdl/SDLFile.cpp
    src/sdl/SDLImgui.cpp)

set(HEADERS_SDL
    src/sdl/SDL.hpp
    src/sdl/SDLMain.hpp
    src/sdl/SDLMainWeb.hpp
    src/sdl/SDLFile.hpp
    src/sdl/SDLImgui.hpp)

set(SOURCES_GL
    src/gl/GL.cpp
    src/gl/Math.cpp)

set(HEADERS_GL
    src/gl/GL.hpp
    src/gl/Math.hpp)

set(SOURCES_SPECCY
    "src/speccy/Z80.c"
    "src/speccy/Rom.c"
    "src/speccy/Speccy.c"
    "src/speccy/Clock.c"
    "src/speccy/Memory.c"
    "src/speccy/Keyboard.c"
    "src/speccy/Render.cpp")

set(HEADERS_SPECCY
    "src/speccy/Z80.h"
    "src/speccy/Rom.h"
    "src/speccy/Speccy.h"
    "src/speccy/Clock.h"
    "src/speccy/Memory.h"
    "src/speccy/Keyboard.h"
    "src/speccy/Render.hpp")

set(SOURCES_IMGUI
    lib/imgui/imgui/imgui.cpp
    lib/imgui/imgui/imgui_draw.cpp
    lib/imgui/imgui/imgui_demo.cpp
    lib/imgui/imgui/imgui_widgets.cpp)

set(HEADERS_IMGUI
    lib/imgui/imgui/imconfig.h
    lib/imgui/imgui/imgui.h
    lib/imgui/imgui/imgui_internal.h
    lib/imgui/imgui/imstb_rectpack.h
    lib/imgui/imgui/imstb_textedit.h
    lib/imgui/imgui/imstb_truetype.h)

SOURCE_GROUP("Source" FILES ${SOURCES})
SOURCE_GROUP("Source" FILES ${HEADERS})

SOURCE_GROUP("Source\\sdl" FILES ${SOURCES_SDL})
SOURCE_GROUP("Source\\sdl" FILES ${HEADERS_SDL})

SOURCE_GROUP("Source\\gl" FILES ${SOURCES_GL})
SOURCE_GROUP("Source\\gl" FILES ${HEADERS_GL})

SOURCE_GROUP("Source\\speccy" FILES ${SOURCES_SPECCY})
SOURCE_GROUP("Source\\speccy" FILES ${HEADERS_SPECCY})

SOURCE_GROUP("Source\\imgui" FILES ${SOURCES_IMGUI})
SOURCE_GROUP("Source\\imgui" FILES ${HEADERS_IMGUI})

include_directories(
    ${PROJECT_SOURCE_DIR}/lib/glm
    ${PROJECT_SOURCE_DIR}/lib/imgui)

if (WIN32)
    message(STATUS "Platform: Windows")
    add_compile_options("/EHsc")

    set(SDL2_PATH ${PROJECT_SOURCE_DIR}/lib/sdl/win/sdl2)
    set(SDL2_LIB_PATH ${SDL2_PATH}/lib/x64)

    set(SDL2_IMAGE_PATH ${PROJECT_SOURCE_DIR}/lib/sdl/win/sdl2-image)
    set(SDL2_IMAGE_LIB_PATH ${SDL2_IMAGE_PATH}/lib/x64)

    set(PVR_PATH ${PROJECT_SOURCE_DIR}/lib/pvr)
    set(PVR_LIB_PATH ${PVR_PATH}/Windows/x86_64)

    include_directories(${SDL2_PATH}/include)
    link_directories(${SDL2_LIB_PATH})

    include_directories(${SDL2_IMAGE_PATH}/include)
    link_directories(${SDL2_IMAGE_LIB_PATH})

    include_directories(${PVR_PATH}/Include)
    link_directories(${PVR_LIB_PATH})

elseif (EMSCRIPTEN)
    message(STATUS "Platform: HTML5")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14 -stdlib=libc++")
    set(CMAKE_CXX_FLAGS_DEBUG "-g")
    set(CMAKE_CXX_FLAGS_RELEASE "-O3")

endif ()

add_executable(
    ${PROJECT_NAME}
    ${SOURCES}
    ${HEADERS}
    ${SOURCES_SDL}
    ${HEADERS_SDL}
    ${SOURCES_GL}
    ${HEADERS_GL}
    ${SOURCES_SPECCY}
    ${HEADERS_SPECCY}
    ${SOURCES_IMGUI}
    ${HEADERS_IMGUI})

add_custom_target(
    ${PROJECT_files_NAME} ALL
    COMMENT "Copying Files..."
    COMMAND ${CMAKE_COMMAND} -E copy_directory
    ${CMAKE_CURRENT_SOURCE_DIR}/files/
    ${CMAKE_CURRENT_BINARY_DIR}/files)

add_dependencies(
    ${PROJECT_NAME}
    ${PROJECT_files_NAME})

if (WIN32)
   target_include_directories(
        ${PROJECT_NAME}
        PRIVATE
        ${PROJECT_SOURCE_DIR}/include/
        ${SDL2_PATH}/include
        ${SDL2_IMAGE_PATH}/include
        ${PVR_PATH}/include)

    target_link_libraries(
        ${PROJECT_NAME}
        PRIVATE
        SDL2
        SDL2main
        SDL2_image
        libEGL
        libGLESv2
        libGLES_CM)

    add_custom_command(
        TARGET ${PROJECT_NAME}
        POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_directory
        "${SDL2_LIB_PATH}"
        $<TARGET_FILE_DIR:${PROJECT_NAME}>)

    add_custom_command(
        TARGET ${PROJECT_NAME}
        POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_directory
        "${SDL2_IMAGE_LIB_PATH}"
        $<TARGET_FILE_DIR:${PROJECT_NAME}>)

    add_custom_command(
        TARGET ${PROJECT_NAME}
        POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_directory
        "${PVR_LIB_PATH}"
        $<TARGET_FILE_DIR:${PROJECT_NAME}>)

elseif (EMSCRIPTEN)

    set_target_properties(
        ${PROJECT_NAME}
        PROPERTIES SUFFIX ".html"
        LINK_FLAGS "--bind -s USE_SDL=2 -s USE_SDL_IMAGE=2 -s USE_WEBGL2=1 -s WASM=1 -s NO_EXIT_RUNTIME=1 -s ALLOW_MEMORY_GROWTH=1 -s --emrun -std=c++11 -O3 --preload-file files --shell-file \"${PROJECT_SOURCE_DIR}/shell.html\" --use-preload-plugins --no-heap-copy")

endif ()
