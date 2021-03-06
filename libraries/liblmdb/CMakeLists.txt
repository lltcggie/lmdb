cmake_minimum_required(VERSION 2.8)

file(READ "${CMAKE_CURRENT_SOURCE_DIR}/lmdb.h" VERSION_HEADER)
string(REGEX MATCH ".*MDB_VERSION_MAJOR[ \t]+([0-9])" MAJOR_VERSION_MATCH "${VERSION_HEADER}")
set(LMDB_MAJOR_VERSION ${CMAKE_MATCH_1})
string(REGEX MATCH ".*MDB_VERSION_MINOR[ \t]+([0-9])" MINOR_VERSION_MATCH "${VERSION_HEADER}")
set(LMDB_MINOR_VERSION ${CMAKE_MATCH_1})
string(REGEX MATCH ".*MDB_VERSION_PATCH[ \t]+([0-9]+)" PATCH_VERSION_MATCH "${VERSION_HEADER}")
set(LMDB_PATCH_VERSION ${CMAKE_MATCH_1})

set(LMDB_VERSION "${LMDB_MAJOR_VERSION}.${LMDB_MINOR_VERSION}.${LMDB_PATCH_VERSION}")

project(lmdb)

option(BUILD_WITH_STATIC_CRT "Use CRT Static Runtime Library on Visual Studio" OFF)

add_library(lmdb STATIC mdb.c lmdb.h midl.c midl.h)
set_target_properties(lmdb PROPERTIES DEBUG_POSTFIX d)

if(MSVC AND BUILD_WITH_STATIC_CRT)
    foreach(flag_var
            CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
            CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO
            CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
            CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO)
        if(${flag_var} MATCHES "/MD")
            string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
        endif(${flag_var} MATCHES "/MD")
        if(${flag_var} MATCHES "/MDd")
            string(REGEX REPLACE "/MDd" "/MTd" ${flag_var} "${${flag_var}}")
        endif(${flag_var} MATCHES "/MDd")
    endforeach(flag_var)
endif()

install(TARGETS lmdb 
        EXPORT lmdb-targets
        DESTINATION lib)

install(FILES getopt.h unistd.h lmdb.h midl.h DESTINATION include)

include(CMakePackageConfigHelpers)

set(INSTALL_INCLUDE_DIR include)
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/package-config.cmakein ${CMAKE_CURRENT_BINARY_DIR}/lmdb-config.cmake
                              INSTALL_DESTINATION CMake
                              PATH_VARS INSTALL_INCLUDE_DIR
                              )
       
write_basic_package_version_file( ${CMAKE_CURRENT_BINARY_DIR}/lmdb-config-version.cmake VERSION ${LMDB_VERSION} COMPATIBILITY SameMajorVersion )

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/lmdb-config.cmake ${CMAKE_CURRENT_BINARY_DIR}/lmdb-config-version.cmake
        DESTINATION CMake )


install(EXPORT lmdb-targets DESTINATION CMake)


