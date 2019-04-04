cmake_minimum_required(VERSION 3.1.0 FATAL_ERROR)
project(sioclient CXX C)

set(MAJOR 1)
set(MINOR 6)
set(PATCH 0)

option(BUILD_SHARED_LIBS "Build the shared library" ON)
option(BUILD_UNIT_TESTS "Build unit test target" ON)

if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "Not define build type, set to Release")
    set(CMAKE_BUILD_TYPE Release)
elseif(NOT (${CMAKE_BUILD_TYPE} STREQUAL "Release" OR ${CMAKE_BUILD_TYPE} STREQUAL "Debug"))
    message(SEND_ERROR "CMAKE_BUILD_TYPE must be either Release or Debug")
    return()
endif()

aux_source_directory("${CMAKE_CURRENT_LIST_DIR}/src" ALL_SRC)
aux_source_directory("${CMAKE_CURRENT_LIST_DIR}/src/internal" ALL_SRC)
file(GLOB ALL_HEADERS "${CMAKE_CURRENT_LIST_DIR}/src/*.h")
set(SIO_INCLUDEDIR "${CMAKE_CURRENT_LIST_DIR}")

add_definitions(
        -DBOOST_DATE_TIME_NO_LIB
        -DBOOST_REGEX_NO_LIB
        -DASIO_STANDALONE
        -D_WEBSOCKETPP_CPP11_STL_
        -D_WEBSOCKETPP_CPP11_FUNCTIONAL_)

add_library(sioclient ${ALL_SRC})

target_include_directories(sioclient PRIVATE
        "${CMAKE_CURRENT_LIST_DIR}/src"
        "${CMAKE_CURRENT_LIST_DIR}/lib/websocketpp"
        "${CMAKE_CURRENT_LIST_DIR}/lib/rapidjson/include"
        "${CMAKE_CURRENT_LIST_DIR}/lib/asio/asio/include")

set_property(TARGET sioclient PROPERTY CXX_STANDARD 11)
set_property(TARGET sioclient PROPERTY CXX_STANDARD_REQUIRED ON)

find_package(Threads REQUIRED)
target_link_libraries(sioclient PRIVATE ${CMAKE_THREAD_LIBS_INIT})

if(BUILD_SHARED_LIBS)
    set_target_properties(sioclient PROPERTIES
            SOVERSION ${MAJOR}
            VERSION ${MAJOR}.${MINOR}.${PATCH})
endif()

list(APPEND TARGET_LIBRARIES sioclient)

find_package(OpenSSL)
if(OPENSSL_FOUND)
    add_library(sioclient_tls ${ALL_SRC})

    target_include_directories(sioclient_tls PRIVATE
            "${CMAKE_CURRENT_LIST_DIR}/src"
            "${CMAKE_CURRENT_LIST_DIR}/lib/websocketpp"
            "${CMAKE_CURRENT_LIST_DIR}/lib/rapidjson/include"
            "${CMAKE_CURRENT_LIST_DIR}/lib/asio/asio/include"
            "${OPENSSL_INCLUDE_DIR}")

    set_property(TARGET sioclient_tls PROPERTY CXX_STANDARD 11)
    set_property(TARGET sioclient_tls PROPERTY CXX_STANDARD_REQUIRED ON)

    target_link_libraries(sioclient_tls PRIVATE ${CMAKE_THREAD_LIBS_INIT} ${OPENSSL_LIBRARIES})
    target_compile_definitions(sioclient_tls PRIVATE -DSIO_TLS)

    if(BUILD_SHARED_LIBS)
        set_target_properties(sioclient_tls PROPERTIES
                SOVERSION ${MAJOR}
                VERSION ${MAJOR}.${MINOR}.${PATCH})
    endif()

    list(APPEND TARGET_LIBRARIES sioclient_tls)
endif()

# install configuration
set(CONFIG_INSTALL_DIR "lib/cmake/${PROJECT_NAME}")
set(INCLUDE_INSTALL_DIR "include/sioclient")
set(VERSION_CONFIG "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
set(PROJECT_CONFIG "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")

# load helper functions
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
        "${VERSION_CONFIG}"
        VERSION ${MAJOR}.${MINOR}.${PATCH}
        COMPATIBILITY SameMajorVersion)
configure_package_config_file(
        "config.cmake.in" "${PROJECT_CONFIG}"
        INSTALL_DESTINATION "${CONFIG_INSTALL_DIR}")

# basic installation settings
install(TARGETS sioclient
        EXPORT "${TARGETS_EXPORT_NAME}"
        LIBRARY DESTINATION "lib"
        ARCHIVE DESTINATION "lib"
        RUNTIME DESTINATION "bin"
        INCLUDES DESTINATION "${INCLUDE_INSTALL_DIR}")

# install .cmake configures
install(FILES "${PROJECT_CONFIG}" "${VERSION_CONFIG}"
        DESTINATION "${CONFIG_INSTALL_DIR}")
install(EXPORT "${TARGETS_EXPORT_NAME}"
        NAMESPACE "${NAMESPACE}"
        DESTINATION "${CONFIG_INSTALL_DIR}")

# install headers
install(DIRECTORY "src/"
        DESTINATION "${INCLUDE_INSTALL_DIR}"
        FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp")

if(BUILD_UNIT_TESTS)
    message(STATUS "Build with unit test support")
    enable_testing()
    add_subdirectory(test)
endif()
