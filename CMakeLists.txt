cmake_minimum_required(VERSION 3.5)
project(coppelia-opengl VERSION 4.1 LANGUAGES C CXX)
# Set external system dependencies for each module

# CMake definitions
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

#set(CMAKE_CONFIGURATION_TYPES "Debug;Release")

# Set header directories for each module
set(COPPELIASIM_DIR $ENV{COPPELIASIM_ROOT_DIR})
set(LIBZMQ $ENV{LIBZMQ})
set(GLAD $ENV{GLAD})
set(GLFW $ENV{GLFW})
string(REPLACE "\\" "/" COPPELIASIM_DIR "${COPPELIASIM_DIR}")
string(REPLACE "\\" "/" LIBZMQ "${LIBZMQ}")
string(REPLACE "\\" "/" GLAD "${GLAD}")
string(REPLACE "\\" "/" GLFW "${GLFW}")


set(XR_RUNTIME_JSON
    "$ENV{XR_RUNTIME_JSON}"
    CACHE PATH
          "Optional location of a specific OpenXR runtime configuration file."
)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../cmake")

# For FetchContent_Declare() and FetchContent_MakeAvailable()
include(FetchContent)
# openxr_loader - From github.com/KhronosGroup
set(BUILD_TESTS
    OFF
    CACHE INTERNAL "Build tests"
)
set(BUILD_API_LAYERS
    ON
    CACHE INTERNAL "Use OpenXR layers"
)
FetchContent_Declare(
    OpenXR
    URL_HASH MD5=924a94a2da0b5ef8e82154c623d88644
    URL https://github.com/KhronosGroup/OpenXR-SDK-Source/archive/refs/tags/release-1.0.34.zip
        SOURCE_DIR
        openxr
)
FetchContent_MakeAvailable(OpenXR)

# Specify source files for each external module
find_package(Boost REQUIRED)
include_directories(${Boost_INCLUDE_DIR})

find_package(OpenGL REQUIRED)
### Default include directories
include_directories(
    ${CMAKE_CURRENT_LIST_DIR}/include 
)
include_directories(
    #${CMAKE_CURRENT_LIST_DIR}/include/CoppeliaSim/zmqRemoteApi/
    ${COPPELIASIM_DIR}/programming/zmqRemoteApi/clients/cpp/
    ${COPPELIASIM_DIR}/programming/zmqRemoteApi/clients/cpp/build/cppzmq
    ${COPPELIASIM_DIR}/programming/zmqRemoteApi/clients/cpp/build/jsoncons/include/
    ${LIBZMQ}/include
    ${GLAD}/include
    ${GLFW}/include
    ${openxr_SOURCE_DIR}/src/common
    ${openxr_SOURCE_DIR}/external/include
)
#add_compile_definitions(WITH_ZEROMQ)

# Collect the whole set of sources to be built
set(SRCS_FILES
    ${SRCS_FILES}
    ${CMAKE_CURRENT_LIST_DIR}/src/*.cpp
    ${GLAD}/src/*.c
    #${CMAKE_CURRENT_LIST_DIR}/src/CoppeliaSim/legacyRemoteApi/*.c
)
file(GLOB SRCS ${SRCS_FILES})

file(COPY assets/ DESTINATION assets/)
message(${SRCS})
# Project compile definitions
add_compile_definitions(PROJECT_NAME)
# Specify exe file to generate
add_executable(${PROJECT_NAME} ${SRCS})
if(NOT "${XR_RUNTIME_JSON}" STREQUAL "")
    set_target_properties(
        ${PROJECT_NAME} PROPERTIES VS_DEBUGGER_ENVIRONMENT
                                   "XR_RUNTIME_JSON=${XR_RUNTIME_JSON}"
    )
endif()
target_compile_definitions(${PROJECT_NAME} PUBLIC -DSIM_REMOTEAPICLIENT_OBJECTS)

# Set compiling and linker options
#set_target_properties(${PROJECT_NAME} PROPERTIES COMMON_LANGUAGE_RUNTIME "")
#target_compile_options(${PROJECT_NAME} PUBLIC "/Gy;/Oi;/sdl;/MP;/ZW;")
#target_compile_options(${PROJECT_NAME} PUBLIC "/AI$ENV{WINDOWS_WINMD_PATH}")
#target_compile_options(${PROJECT_NAME} PUBLIC "/AI$ENV{PLATFORM_WINMD_PATH}")
#message("/AI$ENV{WINDOWS_WINMD_PATH};/AI$ENV{PLATFORM_WINMD_PATH}")
#set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /FORCE:MULTIPLE /SUBSYSTEM:CONSOLE /ENTRY:mainCRTStartup")
#### Collect the external library dependency list
#set(EXT_LIBS ${EXT_LIBS}  
#    ${CMAKE_CURRENT_LIST_DIR}/lib/LWR_Dynamic_Model_Lib.lib)

### Link the list of libraries
message(${CMAKE_CONFIGURATION_TYPES})
target_link_libraries(${PROJECT_NAME} openxr_loader)
target_link_libraries(${PROJECT_NAME} ${EXT_LIBS})
target_link_libraries(${PROJECT_NAME} ${COPPELIASIM_DIR}/programming/zmqRemoteApi/clients/cpp/build/${CMAKE_CONFIGURATION_TYPES}/RemoteAPIClient.lib)
if(${CMAKE_CONFIGURATION_TYPES} STREQUAL "Debug")
    set(LIBZMQ_LIB_NAME libzmq-v143-mt-gd-4_3_5.lib)
elseif(${CMAKE_CONFIGURATION_TYPES} STREQUAL "Release")
    set(LIBZMQ_LIB_NAME libzmq-v143-mt-4_3_5.lib)
endif()
target_link_libraries(${PROJECT_NAME} ${LIBZMQ}/build/lib/${CMAKE_CONFIGURATION_TYPES}/${LIBZMQ_LIB_NAME})
target_link_libraries(${PROJECT_NAME} ${GLFW}/lib-vc2022/glfw3.lib)
target_link_libraries(${PROJECT_NAME} ${OPENGL_gl_LIBRARY})
