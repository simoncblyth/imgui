cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
#[=[
This was added to ImGui to make it work together with Opticks
CMake machinery. It used to be copied into ImGui by imgui-get 
but has now been moved into my ImGui fork for simplicity.
#]=]

set(name ImGui)
project(${name} VERSION 0.1.0)
include(OpticksBuildOptions)

set(OpticksGLEW_VERBOSE ON)
find_package(OpticksGLEW REQUIRED MODULE)
message(STATUS "ImGui.OpticksGLEW_INCLUDE_DIR : ${OpticksGLEW_INCLUDE_DIR}")
message(STATUS "ImGui.OpticksGLEW_LIBRARY     : ${OpticksGLEW_LIBRARY}")

find_package(OpticksGLFW REQUIRED MODULE)
message(STATUS "ImGui.OpticksGLFW_INCLUDE_DIR : ${OpticksGLFW_INCLUDE_DIR}")
message(STATUS "ImGui.OpticksGLFW_LIBRARY     : ${OpticksGLFW_LIBRARY}")


set(IMGUI_DEFINITIONS
)

set(LIBRARIES
   ${OpticksGLEW_LIBRARY} 
   ${OpticksGLFW_LIBRARY}
)

if(APPLE)
    message(STATUS "APPLE")
    find_library( Cocoa_LIBRARY NAMES Cocoa )
    find_library( OpenGL_LIBRARY NAMES OpenGL )
    find_library( IOKit_LIBRARY NAMES IOKit )
    find_library( CoreFoundation_LIBRARY NAMES CoreFoundation )
    find_library( CoreVideo_LIBRARY NAMES CoreVideo )

    set( LIBRARIES 
               ${LIBRARIES} 
               ${Cocoa_LIBRARY}
               ${OpenGL_LIBRARY}
               ${IOKit_LIBRARY}
               ${CoreFoundation_LIBRARY} 
               ${CoreVideo_LIBRARY} )
endif(APPLE)

if(UNIX AND NOT APPLE)
    message(STATUS "UNIX AND NOT APPLE")
    find_library( OpenGL_LIBRARY NAMES GL )

    set( LIBRARIES 
               ${LIBRARIES} 
               ${OpenGL_LIBRARY}
               )
endif(UNIX AND NOT APPLE)


if(WIN32)
   include(GenerateExportHeader)

   # find_library( OpenGL_LIBRARY NAMES opengl32 )
   set( LIBRARIES ${LIBRARIES} opengl32.lib )
   message("WIN32....   LIBS ${LIBRARIES}")
   set(IMGUI_DEFINITIONS  
            ${IMGUI_DEFINITIONS} 
            -DIMGUI_DISABLE_WIN32_DEFAULT_IME_FUNCS 
           )
         #   -DIMGUI_API=IMGUI_EXPORT 
endif(WIN32)


if(WIN64)
   message(STATUS "WIN64....")
endif(WIN64)


if(MSYS)
   message(STATUS "MSYS....")
endif(MSYS)


if(MINGW)
    set( LIBRARIES ${LIBRARIES} -lopengl32 )
    message(STATUS "MINGW....   LIBS ${LIBRARIES}")
    set(IMGUI_DEFINITIONS  ${IMGUI_DEFINITIONS} -DIMGUI_DISABLE_WIN32_DEFAULT_IME_FUNCS )
endif(MINGW)



set(INCLUDE_DIRS
   ${CMAKE_CURRENT_SOURCE_DIR}
   ${CMAKE_CURRENT_BINARY_DIR}

   ${ImGui_SOURCE_DIR}/examples/opengl3_example
   ${OpticksGLEW_INCLUDE_DIR} 
   ${OpticksGLFW_INCLUDE_DIR} 
)

set(DEFINITIONS
   ${OpticksGLEW_DEFINITIONS}
   ${OpticksGLFW_DEFINITIONS}
   ${IMGUI_DEFINITIONS}
)



message(STATUS " DEFINITIONS : ${DEFINITIONS} ")
set(SOURCES
   imgui.cpp
   imgui_draw.cpp
   imgui_demo.cpp
   examples/opengl3_example/imgui_impl_glfw_gl3.cpp
)

SET(HEADERS
   imgui.h
   imconfig.h
   examples/opengl3_example/imgui_impl_glfw_gl3.h
)

include_directories(${INCLUDE_DIRS})
add_definitions(${DEFINITIONS})
add_library(${name} SHARED ${SOURCES})

if(WIN32)
#[=[
    TODO: On revisiting Windows, see if the generate_export_header is actually needed anymore.
#]=]

   generate_export_header(${name}
                  EXPORT_MACRO_NAME IMGUI_API
                  EXPORT_FILE_NAME IMGUI_EXPORTS.h
             )
   set(HEADERS 
             ${HEADERS} 
             ${CMAKE_CURRENT_BINARY_DIR}/IMGUI_EXPORTS.h
      )
endif(WIN32)

target_link_libraries(${name} ${LIBRARIES})

install(TARGETS ${name}  DESTINATION externals/lib)
install(FILES ${HEADERS} DESTINATION externals/include/${name})


#[=[
Getting the example to build entails converting from gl3w to glew 
thats too much hassle whilst operating in external code
TODO: make my own example in OGLRap/tests 
#]=]
if(WITH_IMGUI_EXAMPLE)
   add_executable(${name}Example  examples/opengl3_example/main.cpp)
   target_link_libraries(${name}Example ${LIBRARIES} ${name})
   install(TARGETS ${name}Example DESTINATION bin)
endif(WITH_IMGUI_EXAMPLE)

