cmake_minimum_required(VERSION 3.19)
project("TextIndex" CXX)
SET(SHORT_NAME tindex)

SET(SOURCE_FILES
#    ${CMAKE_SOURCE_DIR}/src/main.cpp
 )

SET(TEST_SOURCE_FILES 
    ${CMAKE_SOURCE_DIR}/src/test/test_main.cpp
    )

SET(INCLUDE_FILES 
    #${CMAKE_SOURCE_DIR}/src/omux/actions.hpp
)

SET(REQUIRE_COMPILE_FEATURES 
    cxx_auto_type
    cxx_generalized_initializers
    cxx_defaulted_move_initializers
    cxx_deleted_functions
    cxx_lambdas
    cxx_long_long_type
    cxx_noexcept
    cxx_nullptr
    cxx_range_for
    cxx_rvalue_references
    cxx_sizeof_member
    cxx_generic_lambdas
    cxx_return_type_deduction
    cxx_std_20
)
if(${CMAKE_VS_PLATFORM_TOOLSET} MATCHES "v[0-9]+_clang_.*")
    message(STATUS "Using MS compatibility for clang" )
    endif()
message(STATUS ${CMAKE_CXX_FLAGS_INIT})

# Define a bunch of interface libraries so targets can link against the options they need
add_library(BUILD_FLAGS INTERFACE)

if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
target_compile_options(BUILD_FLAGS INTERFACE
    "${BUILD_FLAGS}"
    "-Wall"
    "-pedantic"
    "-Wno-c++98-compat"
    "-Wno-c++98-compat-pedantic"
    "-fno-omit-frame-pointer"
    "-mno-omit-leaf-frame-pointer"
    "-fno-inline-functions"
    "-std=c++20"
)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang-cl")
    target_compile_options(BUILD_FLAGS INTERFACE
        "${BUILD_FLAGS}"
        "/Wall"
        "/Wno-c++98-compat"
        "/Wno-c++98-compat-pedantic"
        "/Zi"
    )
endif()

# Test build
option(INSTALL_GTEST OFF)
option(gmock_build_tests OFF)
# Set the version because I was having trouble getting google test to build
# so I directly include gmock instead which requires version be set
set(GOOGLETEST_VERSION 1.10.0)
add_subdirectory(${CMAKE_SOURCE_DIR}/lib/googletest/googlemock)

add_library(TEST_LIBRARIES INTERFACE)
target_link_libraries(TEST_LIBRARIES INTERFACE gmock)
target_include_directories(TEST_LIBRARIES INTERFACE ${CMAKE_SOURCE_DIR}/lib/googletest/googlemock/include)

add_library(UNICODE_DEFINITIONS INTERFACE)
target_compile_definitions(UNICODE_DEFINITIONS INTERFACE UNICODE INTERFACE _UNICODE INTERFACE NOMINMAX)

add_library(CPP_STANDARD INTERFACE)
target_compile_features(CPP_STANDARD INTERFACE ${REQUIRE_COMPILE_FEATURES})

add_library(STANDARD_INCLUDE INTERFACE)
target_include_directories(STANDARD_INCLUDE SYSTEM INTERFACE ${CMAKE_SOURCE_DIR}/lib/include)

add_library(SRC_INCLUDE INTERFACE)
target_include_directories(SRC_INCLUDE INTERFACE ${CMAKE_SOURCE_DIR}/src/)
target_include_directories(SRC_INCLUDE INTERFACE ${CMAKE_SOURCE_DIR}/src/include)

OPTION(CMAKE_EXPORT_COMPILE_COMMANDS ON)

add_executable(${SHORT_NAME} ${SOURCE_FILES} ${CMAKE_SOURCE_DIR}/src/main.cpp)
target_link_libraries(${SHORT_NAME} BUILD_FLAGS)
target_link_libraries(${SHORT_NAME} UNICODE_DEFINITIONS)
target_link_libraries(${SHORT_NAME} CPP_STANDARD)
target_link_libraries(${SHORT_NAME} STANDARD_INCLUDE)
target_link_libraries(${SHORT_NAME} SRC_INCLUDE)
#target_link_libraries(${SHORT_NAME} CONPTY_DEBUG)



# Splitting off ${CMAKE_SOURCE_DIR}/src/test/catch_main.cpp allows us to have multiple test binaries
add_executable(${SHORT_NAME}_test ${CMAKE_SOURCE_DIR}/src/test/catch_main.cpp ${TEST_SOURCE_FILES} ${SOURCE_FILES})
target_link_libraries(${SHORT_NAME}_test BUILD_FLAGS)
target_link_libraries(${SHORT_NAME}_test UNICODE_DEFINITIONS)
target_link_libraries(${SHORT_NAME}_test CPP_STANDARD)
target_link_libraries(${SHORT_NAME}_test STANDARD_INCLUDE)
target_link_libraries(${SHORT_NAME}_test SRC_INCLUDE)
target_link_libraries(${SHORT_NAME}_test TEST_LIBRARIES)


# This just creates some custom targets to run clang tools against our source code
# without it running every time we build
add_custom_target(tidy_check_no_fix 
    clang-tidy -checks=-*,readability-*,cppcoreguidelines-*,concurrency-*,clang-analyzer-*,modernize-*
    ${SOURCE_FILES} ${TEST_FILES} ${CMAKE_SOURCE_DIR}/src/main.cpp ${INCLUDE_FILES}
    -p ${CMAKE_CURRENT_BINARY_DIR}
    
    )
add_custom_target(tidy_check_fix 
    clang-tidy -checks=-*,readability-*,cppcoreguidelines-*,concurrency-*,clang-analyzer-*,modernize-*
    ${SOURCE_FILES} ${TEST_FILES} ${CMAKE_SOURCE_DIR}/src/main.cpp ${INCLUDE_FILES}
    -p ${CMAKE_CURRENT_BINARY_DIR}
    -fix
    )

add_custom_target(format
    clang-format -i
    ${SOURCE_FILES} ${TEST_FILES} ${CMAKE_SOURCE_DIR}/src/main.cpp ${INCLUDE_FILES}
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    )
