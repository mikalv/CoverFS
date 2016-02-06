set(CMAKE_LEGACY_CYGWIN_WIN32 0)
project (CoverFS)
set(PROJECT_VERSION "0.9")
set(ARCHIVE_NAME ${CMAKE_PROJECT_NAME}-${PROJECT_VERSION})



MESSAGE( STATUS "CMAKE_SYSTEM_NAME: " ${CMAKE_SYSTEM_NAME} )

cmake_minimum_required(VERSION 3.1.0)
set( CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake )

add_definitions(-std=c++11 -Wall -O2 -D_FILE_OFFSET_BITS=64)

set(CPP_FILES
    src/CBlockIO.cpp
    src/CCacheIO.cpp
    src/CDirectory.cpp
    src/CEncrypt.cpp
    src/CNetBlockIO.cpp
    src/CSimpleFS.cpp
    src/CWriteRingBuffer.cpp
    src/INode.cpp
    src/main.cpp
)

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)
find_package(Boost COMPONENTS system REQUIRED)
find_package(OpenSSL REQUIRED)
find_package(Libgcrypt REQUIRED)


if (CMAKE_SYSTEM_NAME STREQUAL "Linux")
    MESSAGE( STATUS "Add FUSE Library" )
    set(CPP_FILES ${CPP_FILES} src/fuseoper.cpp)
    find_package(fuse REQUIRED)
    set(FUSE_LIB fuse)
endif ()
if (CMAKE_SYSTEM_NAME STREQUAL "CYGWIN")
    MESSAGE( STATUS "Add Dokan Library")
    set(CPP_FILES ${CPP_FILES} src/dokanoper.cpp)
    add_definitions(-Iresources/dokan/include)
    set(FUSE_LIB dokan)
    LINK_DIRECTORIES(${CMAKE_BINARY_DIR}/resources/dokan)
endif ()

include_directories(${Boost_INCLUDE_DIRS})

add_executable(coverfsserver src/coverfsserver.cpp)
add_executable(coverfs ${CPP_FILES})

target_link_libraries (coverfs ${Boost_SYSTEM_LIBRARY_RELEASE} Threads::Threads ssl crypto gcrypt ${FUSE_LIB})
target_link_libraries (coverfsserver ${Boost_SYSTEM_LIBRARY_RELEASE} Threads::Threads ssl crypto)


add_custom_target(dist
    COMMAND git archive --prefix=${ARCHIVE_NAME}/ HEAD
        | bzip2 > ${CMAKE_BINARY_DIR}/${ARCHIVE_NAME}.tar.bz2
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})