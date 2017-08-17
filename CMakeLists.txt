cmake_minimum_required(VERSION 3.1.0 FATAL_ERROR)
include(cmake/HunterGate.cmake)

HunterGate(
        URL "https://github.com/ruslo/hunter/archive/v0.19.71.tar.gz"
        SHA1 "8fe144d9490501d398777532e0c0f56bc7bb0ec7"
        LOCAL
)


set(MAJOR 1)
set(MINOR 6)
set(PATCH 1)
set(VERSION "${MAJOR}.${MINOR}.${PATCH}")

PROJECT(SocketIOClient VERSION "${VERSION}")

if (NOT CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 11)
endif ()

hunter_add_package(OpenSSL)
hunter_add_package(Boost COMPONENTS system date_time random)
hunter_add_package(RapidJSON)
hunter_add_package(websocketpp)

find_package(OpenSSL REQUIRED)
find_package(Boost CONFIG REQUIRED COMPONENTS system date_time random)
find_package(RapidJSON CONFIG REQUIRED)
find_package(websocketpp CONFIG REQUIRED)




option(BUILD_SHARED_LIBS "Build the shared library" OFF)


aux_source_directory(${CMAKE_CURRENT_LIST_DIR}/src ALL_SRC)
aux_source_directory(${CMAKE_CURRENT_LIST_DIR}/src/internal ALL_SRC)
file(GLOB ALL_HEADERS ${CMAKE_CURRENT_LIST_DIR}/src/*.h)
set(SIO_INCLUDEDIR ${CMAKE_CURRENT_LIST_DIR})

add_library(sioclient ${ALL_SRC})
target_include_directories(sioclient PRIVATE ${Boost_INCLUDE_DIRS}
        ${CMAKE_CURRENT_LIST_DIR}/src
        )

target_link_libraries(sioclient PUBLIC RapidJSON::rapidjson)

target_link_libraries(sioclient PUBLIC Boost::boost Boost::system Boost::date_time Boost::random)

if (BUILD_SHARED_LIBS)
    set_target_properties(sioclient
            PROPERTIES
            SOVERSION ${MAJOR}
            VERSION ${MAJOR}.${MINOR}.${PATCH}
            )
endif ()

list(APPEND TARGET_LIBRARIES sioclient)

add_library(sioclient_tls ${ALL_SRC})
target_link_libraries(sioclient_tls PUBLIC OpenSSL::SSL OpenSSL::Crypto)
target_link_libraries(sioclient_tls PUBLIC Boost::boost Boost::system Boost::date_time Boost::random)
target_link_libraries(sioclient_tls PUBLIC websocketpp::websocketpp)
target_compile_definitions(sioclient_tls PRIVATE -DSIO_TLS)
if (BUILD_SHARED_LIBS)
    set_target_properties(sioclient_tls
            PROPERTIES
            SOVERSION ${MAJOR}
            VERSION ${MAJOR}.${MINOR}.${PATCH}
            )
endif ()
list(APPEND TARGET_LIBRARIES sioclient_tls)

#
# installation
#

set(include_install_dir "include")
set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(config_export_name "${PROJECT_NAME}Config")
set(targets_export_name "${PROJECT_NAME}Targets")
set(project_config "${CMAKE_CURRENT_BINARY_DIR}/${config_export_name}.cmake")
set(version_config "${CMAKE_CURRENT_BINARY_DIR}/${config_export_name}Version.cmake")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
configure_package_config_file( # Uses target_exports_name
        "cmake/hunter.Config.cmake.in"
        "${project_config}"
        INSTALL_DESTINATION "${config_install_dir}"
        )
write_basic_package_version_file(
        "${version_config}"
        VERSION "${PROJECT_VERSION}"
        COMPATIBILITY SameMajorVersion)

install(TARGETS ${TARGET_LIBRARIES} DESTINATION lib
        EXPORT ${targets_export_name}
        INCLUDES DESTINATION "${include_install_dir}")

install(
        EXPORT "${targets_export_name}"
        NAMESPACE "${namespace}"
        DESTINATION "${config_install_dir}")

install(FILES ${ALL_HEADERS}
        DESTINATION include
        )

install(
        FILES "${project_config}" "${version_config}"
        DESTINATION "${config_install_dir}")
