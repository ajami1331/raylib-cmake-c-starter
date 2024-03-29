# Dependencies
set(RAYLIB_VERSION 4.5.0)
find_package(raylib ${RAYLIB_VERSION} QUIET) # QUIET or REQUIRED
if (NOT raylib_FOUND) # If there's none, fetch and build raylib
  include(FetchContent)
  FetchContent_Declare(
    raylib
    URL https://github.com/raysan5/raylib/archive/refs/tags/${RAYLIB_VERSION}.tar.gz
  )
  FetchContent_GetProperties(raylib)
  if (NOT raylib_POPULATED) # Have we downloaded raylib yet?
    set(FETCHCONTENT_QUIET NO)
    FetchContent_Populate(raylib)
    set(BUILD_EXAMPLES OFF CACHE BOOL "" FORCE) # don't build the supplied examples
    add_subdirectory(${raylib_SOURCE_DIR} ${raylib_BINARY_DIR})
  endif()
endif()

file(GLOB_RECURSE SOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *.c;*.h)

include_directories(../shared)

add_library(game SHARED ${SOURCES})

target_link_libraries(game raylib)

if (MSVC)
  add_custom_command(TARGET game PRE_BUILD
                    COMMAND ${CMAKE_COMMAND} -E rm -rf ${CMAKE_BINARY_DIR}/pdb)
  add_custom_command(TARGET game POST_BUILD
                    COMMAND ${CMAKE_COMMAND} -E sleep 0.1)
  add_custom_command(TARGET game POST_BUILD
                    COMMAND ${CMAKE_COMMAND} -E touch -f $<TARGET_FILE_DIR:game>/fl)
  add_compile_definitions(LOG_LEVEL=LOG_DEBUG)
  string(RANDOM LENGTH 10 ALPHABET 0123456789 PROJECT_ID)
  set_target_properties(game PROPERTIES COMPILE_PDB_NAME game-${PROJECT_ID})
  set_target_properties(game PROPERTIES PDB_NAME game-${PROJECT_ID}-1)
  set_target_properties(game PROPERTIES PDB_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/pdb)
  message(STATUS "PDB file name: game-${PROJECT_ID}.pdb")
  add_custom_command(TARGET game POST_BUILD
                    COMMAND ${CMAKE_COMMAND} -E touch ${CMAKE_CURRENT_SOURCE_DIR}/CMakeLists.txt)
endif()