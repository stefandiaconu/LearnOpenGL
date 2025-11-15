cmake_minimum_required (VERSION 3.31)

set(CMAKE_CXX_STANDARD 20)

project(LearnOpenGL)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/modules/")

find_package(GLFW3 REQUIRED)
message(STATUS "Found GLFW3 in ${GLFW3_INCLUDE_DIR}")

if(WIN32)
    set(LIBS glfw3 opengl32 assimp freetype irrKlang)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
elseif(UNIX AND NOT APPLE)
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -Wall")
    find_package(OpenGL REQUIRED)
    add_definitions(${OPENGL_DEFINITIONS})
    find_package(X11 REQUIRED)
    # note that the order is important for setting the libs
    # use pkg-config --libs $(pkg-config --print-requires --print-requires-private glfw3) in a terminal to confirm
    set(LIBS ${GLFW3_LIBRARY} X11 Xrandr Xinerama Xi Xxf86vm Xcursor GL dl pthread freetype ${ASSIMP_LIBRARY})
    set (CMAKE_CXX_LINK_EXECUTABLE "${CMAKE_CXX_LINK_EXECUTABLE} -ldl")
elseif(APPLE)
    INCLUDE_DIRECTORIES(/System/Library/Frameworks)
    FIND_LIBRARY(COCOA_LIBRARY Cocoa)
    FIND_LIBRARY(OpenGL_LIBRARY OpenGL)
    FIND_LIBRARY(IOKit_LIBRARY IOKit)
    FIND_LIBRARY(CoreVideo_LIBRARY CoreVideo)
    MARK_AS_ADVANCED(COCOA_LIBRARY OpenGL_LIBRARY)
    SET(APPLE_LIBS ${COCOA_LIBRARY} ${IOKit_LIBRARY} ${OpenGL_LIBRARY} ${CoreVideo_LIBRARY})
    SET(APPLE_LIBS ${APPLE_LIBS} ${GLFW3_LIBRARY} ${ASSIMP_LIBRARY} ${FREETYPE_LIBRARIES})
    set(LIBS ${LIBS} ${APPLE_LIBS})
else()
    set(LIBS )
endif(WIN32)

set(CHAPTERS
        1_getting_started
)

set(1_getting_started
        1_1_hello_window
        2_1_hello_triangle
        2_2_hello_triangle_indexed
        2_3_hello_triangle_exercise_1
        2_4_hello_triangle_exercise_2
        2_5_hello_triangle_exercise_3
        3_1_shaders_uniform
)

add_library(GLAD "src/glad.c")
set(LIBS ${LIBS} GLAD)

function(create_project_from_sources chapter demo)
    file(GLOB SOURCE
            "src/${chapter}/${demo}/*.h"
            "src/${chapter}/${demo}/*.cpp"
    )
    if (demo STREQUAL "")
        SET(replaced "")
        string(REPLACE "/" "_" replaced ${chapter})
        set(NAME ${replaced})
    else()
        set(NAME "${chapter}__${demo}")
    endif()
    add_executable(${NAME} ${SOURCE})
    target_link_libraries(${NAME} ${LIBS})
#    if(MSVC)
#        target_compile_options(${NAME} PRIVATE /std:c++17 /MP)
#        target_link_options(${NAME} PUBLIC /ignore:4099)
#    endif(MSVC)
#    if(WIN32)
#        set_target_properties(${NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${chapter}")
#        set_target_properties(${NAME} PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${chapter}/Debug")
#    elseif(UNIX AND NOT APPLE)
#        set_target_properties(${NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${chapter}")
#    elseif(APPLE)
#        set_target_properties(${NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${chapter}")
#        set_target_properties(${NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY_DEBUG "${CMAKE_SOURCE_DIR}/bin/${chapter}")
#        set_target_properties(${NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY_RELEASE "${CMAKE_SOURCE_DIR}/bin/${chapter}")
#    endif(WIN32)
endfunction()

# then create a project file per tutorial
foreach(CHAPTER ${CHAPTERS})
    foreach(DEMO ${${CHAPTER}})
        create_project_from_sources(${CHAPTER} ${DEMO})
    endforeach(DEMO)
endforeach(CHAPTER)

include_directories(${CMAKE_SOURCE_DIR}/include)