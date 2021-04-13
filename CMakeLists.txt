cmake_minimum_required(VERSION 3.20) # FetchContent is available in 3.11+

project(freeglutapp VERSION 0.0.1)

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  #add_definitions(/DYOURDEFINITION)
  message(" DEBUG")
endif()

message("CMAKE_BUILD_TYPE " ${CMAKE_BUILD_TYPE})

set(BUILDMODE Debug)

# https://stackoverflow.com/questions/6594796/how-do-i-make-cmake-output-into-a-bin-dir

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${BUILDMODE})
#set(ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/Debug)
#message("PROJECT_BINARY_DIR:>>> " ${PROJECT_BINARY_DIR})
#message("CMAKE_BINARY_DIR:>>> " ${CMAKE_BINARY_DIR})
#message("CMAKE_LIBRARY_OUTPUT_DIRECTORY:>>> " ${CMAKE_LIBRARY_OUTPUT_DIRECTORY})
#message("ARCHIVE_OUTPUT_DIRECTORY:>>> " ${ARCHIVE_OUTPUT_DIRECTORY})
#message("SRC_LIB_DIR:>>> " ${SRC_LIB_DIR})
#message("DST_LIB_DIR:>>> " ${DST_LIB_DIR})

set(CPP_INCLUDE_DIRS "")
set(CPP_SOURCES_FILES "")
set(CPP_HEADERS_FILES "")

if (NOT freeglutgit_FOUND)
  include(FetchContent)
  FetchContent_Declare(
    freeglutgit
    GIT_REPOSITORY https://github.com/dcnieho/FreeGLUT.git
    GIT_TAG FG_3_2_1
  )
  FetchContent_GetProperties(freeglutgit)
  if (NOT freeglutgit_POPULATED) # Have we downloaded glfw yet?
    set(FETCHCONTENT_QUIET NO)
    FetchContent_Populate(freeglutgit)

    set(INSTALL_PDB OFF CACHE BOOL "" FORCE)
    set(FREEGLUT_BUILD_DEMOS OFF CACHE BOOL "" FORCE)
    
    #include_directories(${glfw_SOURCE_DIR}/include)
    list(APPEND CPP_INCLUDE_DIRS ${freeglutgit_SOURCE_DIR}/include)

    #add_subdirectory(${glfw_SOURCE_DIR})
    message("freeglutgit_SOURCE_DIR " ${freeglutgit_SOURCE_DIR})
    message("freeglutgit_BINARY_DIR " ${freeglutgit_BINARY_DIR})
    add_subdirectory(${freeglutgit_SOURCE_DIR} ${freeglutgit_BINARY_DIR})

  endif()
endif()

if (NOT imgui_FOUND)
  include(FetchContent)
  FetchContent_Declare(
    imgui
    GIT_REPOSITORY https://github.com/ocornut/imgui.git
    GIT_TAG v1.82
  )
  FetchContent_GetProperties(imgui)
  if (NOT imgui_POPULATED) # Have we downloaded glfw yet?
    set(FETCHCONTENT_QUIET NO)
    FetchContent_Populate(imgui)
    #message("imgui_SOURCE_DIR " ${imgui_SOURCE_DIR})
    #include_directories(${imgui_SOURCE_DIR} ${imgui_SOURCE_DIR}/backends)
    list(APPEND CPP_SOURCES_FILES ${imgui_SOURCE_DIR}/imgui.cpp)
    list(APPEND CPP_SOURCES_FILES ${imgui_SOURCE_DIR}/imgui_demo.cpp)
    list(APPEND CPP_SOURCES_FILES ${imgui_SOURCE_DIR}/imgui_draw.cpp)
    list(APPEND CPP_SOURCES_FILES ${imgui_SOURCE_DIR}/imgui_tables.cpp)
    list(APPEND CPP_SOURCES_FILES ${imgui_SOURCE_DIR}/imgui_widgets.cpp)
    list(APPEND CPP_SOURCES_FILES ${imgui_SOURCE_DIR}/backends/imgui_impl_glut.cpp)
    list(APPEND CPP_SOURCES_FILES ${imgui_SOURCE_DIR}/backends/imgui_impl_opengl2.cpp)

    list(APPEND CPP_HEADERS_FILES ${imgui_SOURCE_DIR}/imgui.h)
    #list(APPEND CPP_HEADERS_FILES ${imgui_SOURCE_DIR}/imconfig.h)
    #list(APPEND CPP_HEADERS_FILES ${imgui_SOURCE_DIR}/imgui_internal.h)
    #list(APPEND CPP_HEADERS_FILES ${imgui_SOURCE_DIR}/imstb_rectpack.h)
    #list(APPEND CPP_HEADERS_FILES ${imgui_SOURCE_DIR}/imstb_textedit.h)
    #list(APPEND CPP_HEADERS_FILES ${imgui_SOURCE_DIR}/imstb_truetype.h)
    list(APPEND CPP_HEADERS_FILES ${imgui_SOURCE_DIR}/backends/imgui_impl_glut.h)
    list(APPEND CPP_HEADERS_FILES ${imgui_SOURCE_DIR}/backends/imgui_impl_opengl2.h)

    list(APPEND CPP_INCLUDE_DIRS ${imgui_SOURCE_DIR})
    list(APPEND CPP_INCLUDE_DIRS ${imgui_SOURCE_DIR}/backends)

  endif()
endif()

# freeglutd.dll
# freeglut_staticd

find_library(freeglutstaticd_LIB NAMES freeglut_staticd HINTS ${freeglutgit_BINARY_DIR}/lib/${BUILDMODE}) #lib search
find_library(freeglutd_LIB NAMES freeglutd HINTS ${freeglutgit_BINARY_DIR}/lib/${BUILDMODE}) #lib search

#message("freeglutstatic_LIB >>>>>> " ${freeglut_staticd})
#message("freeglut_LIB >>>>>> " ${freeglut_LIB})

include_directories(${CPP_INCLUDE_DIRS}) #need to be added in else error
#================================================
# APPLICATION
#================================================
add_executable(${PROJECT_NAME} 
  #main.c 
  #sampletest01.cpp
  freeglut_imgui.cpp
  ${CPP_SOURCES_FILES}
  ${CPP_HEADERS_FILES}
)
#target_link_libraries(${PROJECT_NAME} ${freeglutd_LIB} ${freeglutd_LIB})
target_link_libraries(${PROJECT_NAME} 
  #PRIVATE ${freeglutstaticd_LIB} 
  PRIVATE ${freeglutd_LIB}
)
#target_link_libraries(${PROJECT_NAME} PRIVATE freeglut_staticd)
#message("BIN" "${freeglutgit_BINARY_DIR}/bin/${BUILDMODE}")
#message("DLL" "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}")
# https://stackoverflow.com/questions/34799916/copy-file-from-source-directory-to-binary-directory-using-cmake
# copy dll
# pass # should not be used
#file(COPY ${freeglutgit_BINARY_DIR}/bin/${BUILDMODE}/freeglutd.dll
#DESTINATION ${CMAKE_LIBRARY_OUTPUT_DIRECTORY})

# pass # should not be used
#configure_file(
  #${freeglutgit_BINARY_DIR}/bin/${BUILDMODE}/freeglutd.dll 
  #${CMAKE_LIBRARY_OUTPUT_DIRECTORY} 
  #COPYONLY)

message("CMAKE_CURRENT_SOURCE_DIR  " ${CMAKE_CURRENT_SOURCE_DIR})

# pass # should be used when compile finish
# note if copy fail that mean the firewall or security is blocking the call command.
add_custom_command(
  TARGET ${PROJECT_NAME} POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy_if_different 
  ${freeglutgit_BINARY_DIR}/bin/${BUILDMODE}/freeglutd.dll
  $<TARGET_FILE_DIR:${PROJECT_NAME}>
  COMMENT "COPY ${freeglutgit_BINARY_DIR}/bin/${BUILDMODE}/freeglutd.dll to output => ${PROJECT_BINARY_DIR}/Debug"
)