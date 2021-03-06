cmake_minimum_required(VERSION 2.8)
project(solacekdb CXX)

set (CMAKE_BUILD_TYPE Release)
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -DKXVER=3")
set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -DKXVER=3")

set(MY_LIBRARY_NAME kdbsolace)
file(GLOB SRC_FILES src/*.cpp)
set_source_files_properties(${SRC_FILES} PROPERTIES LANGUAGE CXX)

message(STATUS "Generator : ${CMAKE_GENERATOR}")
message(STATUS "Build Tool : ${CMAKE_BUILD_TOOL}")
message(STATUS "Solace API : $ENV{BUILD_HOME}")

include_directories (
    $ENV{BUILD_HOME}/include
)

find_library(SOLCLIENT_LIBRARY
    NAMES solclient libsolclient
    HINTS "$ENV{BUILD_HOME}/lib/" "$ENV{BUILD_HOME}/lib/Win64"
)

if (CMAKE_GENERATOR MATCHES "Visual Studio")
    set(INSTALL_SCRIPT "install.bat")
    file(DOWNLOAD "https://github.com/KxSystems/kdb/raw/master/w64/q.lib" "${CMAKE_BINARY_DIR}/q.lib" )
    set(LINK_LIBS "${CMAKE_BINARY_DIR}/q.lib")
else()
    set(INSTALL_SCRIPT "install.sh")
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-strict-aliasing")
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-strict-aliasing")
endif()
    
add_library (${MY_LIBRARY_NAME} SHARED ${SRC_FILES})

IF(APPLE)
   set_target_properties(${MY_LIBRARY_NAME} PROPERTIES LINK_FLAGS "-undefined dynamic_lookup")
   set_target_properties(${MY_LIBRARY_NAME} PROPERTIES SUFFIX .so)
endif()

target_link_libraries(${MY_LIBRARY_NAME} ${SOLCLIENT_LIBRARY} ${LINK_LIBS})
set_target_properties(${MY_LIBRARY_NAME} PROPERTIES PREFIX "")

install(FILES README.md LICENSE install.sh install.bat DESTINATION ${PROJECT_BINARY_DIR}/kdbsolace/)
install(DIRECTORY examples DESTINATION ${PROJECT_BINARY_DIR}/kdbsolace)
install(DIRECTORY q DESTINATION ${PROJECT_BINARY_DIR}/kdbsolace)
install(TARGETS ${MY_LIBRARY_NAME} DESTINATION ${PROJECT_BINARY_DIR}/kdbsolace/lib OPTIONAL)
