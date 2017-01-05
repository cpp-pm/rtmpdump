cmake_minimum_required(VERSION 3.0)

include(HunterGate.cmake)

HunterGate(
    URL "https://github.com/caseymcc/hunter/archive/v0.17.4.tar.gz"
    SHA1 "e14c24d104c14b604fcff0f1a660114d534c13ab"
)


project(rtmpdump VERSION 2.4)

if(MSVC)
    hunter_add_package(getopt)
    find_package(getopt CONFIG REQUIRED)
    set(rtmp_extra_libraries getopt::getopt)
#    add_definitions(-D_OFF_T_DEFINED) #redefining value in code
endif()

if(NOT WIN32)
    #remove O3 if there, add Wall and O2
    string(REPLACE "-O3" "" CMAKE_CXX_FLAGS_RELEASE ${CMAKE_CXX_FLAGS_RELEASE})
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -Wall -O2")

    #android has pthread functionality in the standard libs
    if(NOT ANDROID)
        set(rtmp_extra_libraries ${rtmp_extra_libraries} 
            pthread
        )
    endif()
endif()

message(STATUS "${CMAKE_CXX_FLAGS_RELEASE}")

add_definitions(-DRTMPDUMP_VERSION="${PROJECT_VERSION}")

include_directories(${CMAKE_CURRENT_BINARY_DIR}/librtmp)

add_executable(rtmpdump rtmpdump.c)
target_link_libraries(rtmpdump librtmp ${rtmp_extra_libraries})

add_executable(rtmpsrv rtmpsrv.c thread.c thread.h)
target_link_libraries(rtmpsrv librtmp ${rtmp_extra_libraries})

add_executable(rtmpsuck rtmpsuck.c thread.c thread.h)
target_link_libraries(rtmpsuck librtmp ${rtmp_extra_libraries})

add_executable(rtmpgw rtmpgw.c thread.c thread.h)
target_link_libraries(rtmpgw librtmp ${rtmp_extra_libraries})

add_subdirectory(librtmp)