cmake_minimum_required(VERSION 3.21)
project(hellotext VERSION 0.0.1)
set_property(GLOBAL PROPERTY USE_FOLDERS ON)  # Group CMake targets inside a folder
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)         # Generate compile_commands.json for language servers

include(FetchContent)
FetchContent_Declare(
    fmt
    GIT_REPOSITORY https://github.com/fmtlib/fmt.git
    GIT_TAG        10.1.0
)
list(APPEND FETCH_CONTENTS fmt)
FetchContent_Declare(
    asio
    GIT_REPOSITORY https://github.com/mononerv/asio.git
    GIT_TAG        1f9c5682840d3dc32e6f36f2e0f54a0e854e87d6
)
list(APPEND FETCH_CONTENTS asio)
FetchContent_Declare(
    utf8cpp
    GIT_REPOSITORY https://github.com/nemtrif/utfcpp.git
    GIT_TAG        v3.2.4
)
list(APPEND FETCH_CONTENTS utf8cpp)
FetchContent_Declare(
    ctre
    GIT_REPOSITORY https://github.com/hanickadot/compile-time-regular-expressions.git
    GIT_TAG        v3.8
)
list(APPEND FETCH_CONTENTS ctre)
FetchContent_MakeAvailable(${FETCH_CONTENTS})

# Group dependencies in Visual Studio and Xcode
if (CMAKE_GENERATOR MATCHES "Visual Studio" OR CMAKE_GENERATOR MATCHES "Xcode")
    set_target_properties(fmt PROPERTIES FOLDER deps)
endif()

if (APPLE)
elseif (UNIX AND NOT APPLE AND NOT EMSCRIPTEN)  # Linux, BSD, Solaris, Minix
elseif (WIN32)
elseif(EMSCRIPTEN)
    message(FATAL_ERROR "Emscripten is not supported!")
else()
    message(FATAL_ERROR "Unkown platform!")
endif()

# Compiler specific options
if (NOT MSVC)
    set(BASE_OPTIONS
        "-Wall"
        "-Wextra"
        "-Wconversion"
        "-Wpedantic"
        "-Wshadow"
        "-Werror"
        # fmt warnings
        "-Wno-unknown-attributes"
        # asio
        "-Wno-sign-conversion"
        "-Wno-shadow"
        "-Wno-shorten-64-to-32"
        "-Wno-implicit-int-conversion"
        "-Wno-unused-private-field"
        "-Wno-deprecated-declarations"
    )
else()
    set(BASE_OPTIONS
        "/W4"
        "/WX"
        "/utf-8"
        "/Zc:__cplusplus"
        #"/fsanitize=address"  # Doesn't work without Visual Studio
    )
endif()

set(HEADERS "")
set(SOURCES replfy.cpp)
add_executable(replfy ${HEADERS} ${SOURCES})
target_include_directories(replfy PRIVATE ${PROJECT_SOURCE_DIR})
target_compile_features(replfy PRIVATE cxx_std_20)
target_compile_options(replfy PRIVATE ${BASE_OPTIONS})
target_link_libraries(replfy
    PRIVATE
    asio::asio
    fmt
    utf8::cpp
    ctre::ctre
)
source_group(TREE "${CMAKE_CURRENT_LIST_DIR}" FILES ${HEADERS} ${SOURCES})
