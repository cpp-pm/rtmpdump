cmake_minimum_required(VERSION 3.0)

include(HunterGate.cmake)

HunterGate(
	URL "https://github.com/ruslo/hunter/archive/v0.16.30.tar.gz"
    SHA1 "c5b159b48a3a842697f08fe2a72464adfd78a55d"
)

include(GenerateExportHeader)

project(librtmp VERSION 2.4)

hunter_add_package(OpenSSL)
find_package(OpenSSL REQUIRED)

hunter_add_package(ZLIB)
find_package(ZLIB REQUIRED)

set(librtmp_source
    amf.c
    hashswf.c
    log.c
    parseurl.c
    rtmp.c
)

set(librtmp_headers
    amf.h
    bytes.h
    dh.h
    handshake.h
    http.h
    log.h
    rtmp.h
)

#including target file location instead as ZLIB::zlib (${ZLIB_LIBRARIES}=="ZLIB::zlib") 
# will not exist as a target when this library is used
get_property(zlib_target_location TARGET ${ZLIB_LIBRARIES} PROPERTY LOCATION)
set(librtmp_libs
    ${OPENSSL_LIBRARIES}
    ${zlib_target_location}
)

if(WIN32)
    set(librtmp_libs ${librtmp_libs} 
        ws2_32
        winmm
        gdi32
    )
endif()

include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(${OPENSSL_INCLUDE_DIR})
include_directories(${ZLIB_INCLUDE_DIRS})

add_library(librtmp
	${librtmp_source}
	${librtmp_headers}
)

    
generate_export_header(librtmp)
target_link_libraries(librtmp ${librtmp_libs})

#install
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

configure_file("Config.cmake.in" "${project_config}" @ONLY)

install(
    TARGETS librtmp
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(FILES ${librtmp_headers} DESTINATION include/librtmp)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/librtmp_export.h" DESTINATION include/librtmp)

if(BUILD_SHARED_LIBS)
    install(FILES $<TARGET_PDB_FILE:${PROJECT_NAME}> DESTINATION bin OPTIONAL)
endif()

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)