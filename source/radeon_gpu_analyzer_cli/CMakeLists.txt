cmake_minimum_required (VERSION 3.10)
project (radeon_gpu_analyzer_cli)

# The version number
set (RadeonGPUAnalyzerCLI_VERSION_MAJOR 2)
set (RadeonGPUAnalyzerCLI_VERSION_MINOR 0)

# Windows-specific preprocessor directives
if(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
endif()

# Identify if this script runs on a Debian system
set (IsDebian FALSE)
if(EXISTS "/etc/debian_version")
    set(IsDebian TRUE)
endif()

# Add this definitions if static linkage is used with the other projects
add_definitions(-DRGA_BACKEND_STATIC -DAMDTBASETOOLS_STATIC -DAMDTOSWRAPPERS_STATIC)

# Add update_check_api
add_subdirectory (${CMAKE_CURRENT_SOURCE_DIR}/../../../Common/Src/update_check_api ${CMAKE_CURRENT_BINARY_DIR}/Common/Src/update_check_api)

# Configure a header file to pass some of the CMake settings
# to the source code
configure_file (
  "${PROJECT_SOURCE_DIR}/RadeonGPUAnalyzerCLIConfig.h.in"
  "${PROJECT_BINARY_DIR}/RadeonGPUAnalyzerCLIConfig.h"
  )

# Add the binary tree to the search path for include files
# so that we will find ProjectConfig.h
include_directories("${PROJECT_BINARY_DIR}")

IF (WIN32)
    # get the location of the Visual Studio redist libraries - but don't create an install rule to install them
    # the libs will be copied manually as a post build step
    set(CMAKE_INSTALL_SYSTEM_RUNTIME_LIBS_SKIP TRUE)
    include(InstallRequiredSystemLibraries)
ENDIF(WIN32)

# Include directories
include_directories("${PROJECT_SOURCE_DIR}/../../external/dx11/")
include_directories("${PROJECT_SOURCE_DIR}/../common")
include_directories("${PROJECT_SOURCE_DIR}/../../external")
include_directories("${PROJECT_SOURCE_DIR}/../../external/third-party")
include_directories("${PROJECT_SOURCE_DIR}/../../external/dx11/GSA")
include_directories("${PROJECT_SOURCE_DIR}/../../../Common/Src")
include_directories("${PROJECT_SOURCE_DIR}/../")
include_directories("${PROJECT_SOURCE_DIR}/../../")
include_directories("${PROJECT_SOURCE_DIR}/../../external/celf/Include")
include_directories("${PROJECT_SOURCE_DIR}/../../../Common/Src/DynamicLibraryModule")
include_directories("${PROJECT_SOURCE_DIR}/../../../Common/Lib/AMD/ACL/TOT/include")
include_directories("${PROJECT_SOURCE_DIR}/../../../Common/Lib/AMD/CAL/8.95/include")
include_directories("${PROJECT_SOURCE_DIR}/../../../Common/Lib/AMD/APPSDK/3-0/include")
include_directories("${PROJECT_SOURCE_DIR}/../../../Common/Src/TSingleton")
include_directories("${PROJECT_SOURCE_DIR}/../../../Common/Src/DeviceInfo")
include_directories("${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/")
include_directories(${UPDATECHECKAPI_INC_DIRS})

if (NOT IsDebian)
# On Debian we use the native Boost libraries
include_directories("${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0")
endif()

# Windows-specific include directories
if(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
    include_directories("${PROJECT_SOURCE_DIR}/../../external/dx10_asm/include")
    include_directories("${PROJECT_SOURCE_DIR}/../../../Common/Src/ShaderUtils")
    include_directories("${PROJECT_SOURCE_DIR}/../../../Common/Src/ShaderUtils/DX10")
    include_directories("${PROJECT_SOURCE_DIR}/../../../Common/Src/ShaderUtils/DX11")
endif()

# Source files that are common for Windows and Linux
file(GLOB COMMON_SRC
    "../common/*.cpp"
    "*.cpp"
    "*.h"
    ${UPDATECHECKAPI_SRC}
    ${UPDATECHECKAPI_INC}
)

# Windows-specific include directories
if(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
    file(GLOB  WINDOWS_INC
        "../common/resources/*CLI.rc"
    )
endif()

# Pick up the source files that are relevant to the platform
if(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
    add_executable(radeon_gpu_analyzer_cli ${WINDOWS_INC} ${COMMON_SRC} ${WINDOWS_SRC})
    if (CMAKE_64BIT_TARGET)
        # 64-bit target
        find_library(BOOST_LIBRARY1 libboost_filesystem-vc140-mt-1_59 HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/VC14/x86_64")
        find_library(BOOST_LIBRARY2 libboost_system-vc140-mt-1_59 HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/VC14/x86_64")
        find_library(BOOST_LIBRARY3 libboost_program_options-vc140-mt-1_59 HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/VC14/x86_64")
        find_library(BOOST_LIBRARY4 libboost_regex-vc140-mt-1_59 HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/VC14/x86_64")
        find_library(BOOST_LIBRARY1D libboost_filesystem-vc140-mt-gd-1_59 HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/VC14/x86_64")
        find_library(BOOST_LIBRARY2D libboost_system-vc140-mt-gd-1_59 HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/VC14/x86_64")
        find_library(BOOST_LIBRARY3D libboost_program_options-vc140-mt-gd-1_59 HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/VC14/x86_64")
        find_library(BOOST_LIBRARY4D libboost_regex-vc140-mt-gd-1_59 HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/VC14/x86_64")
    else()
        # 32-bit target
        find_library(BOOST_LIBRARY1 libboost_filesystem-vc140-mt-1_59 HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/VC14/x86")
        find_library(BOOST_LIBRARY2 libboost_system-vc140-mt-1_59 HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/VC14/x86")
        find_library(BOOST_LIBRARY3 libboost_program_options-vc140-mt-1_59 HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/VC14/x86")
        find_library(BOOST_LIBRARY4 libboost_regex-vc140-mt-1_59 HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/VC14/x86")
        find_library(BOOST_LIBRARY1D libboost_filesystem-vc140-mt-gd-1_59 HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/VC14/x86")
        find_library(BOOST_LIBRARY2D libboost_system-vc140-mt-gd-1_59 HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/VC14/x86")
        find_library(BOOST_LIBRARY3D libboost_program_options-vc140-mt-gd-1_59 HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/VC14/x86")
        find_library(BOOST_LIBRARY4D libboost_regex-vc140-mt-gd-1_59 HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/VC14/x86")
    endif()
elseif(${CMAKE_SYSTEM_NAME} STREQUAL "Linux" OR ${CMAKE_SYSTEM_NAME} STREQUAL "Darwin")
    # DeviceInfoLib objects are defined in radeon_gpu_analyzer_backend/CMakeLists.txt
    add_executable(radeon_gpu_analyzer_cli ${COMMON_SRC} ${LINUX_SRC} $<TARGET_OBJECTS:DeviceInfoLib>)
    set_target_properties(radeon_gpu_analyzer_cli PROPERTIES ENABLE_EXPORTS ON)

    if(IsDebian)
        # Debian-specific
        # Configure Boost link settings
        set(Boost_USE_STATIC_LIBS ON)
        set(Boost_USE_MULTITHREADED ON)
        set(Boost_USE_STATIC_RUNTIME OFF)

        # We use 1.55 as the minimum version
        find_package(Boost 1.55.0 COMPONENTS filesystem system regex program_options)

        if(Boost_FOUND)
            include_directories(${Boost_INCLUDE_DIRS})
        endif()
    else()
        # RHEL
        find_library(BOOST_LIBRARY1 libboost_filesystem.a HINTS ${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/RHEL6/x86_64)
        find_library(BOOST_LIBRARY2 libboost_system.a HINTS ${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/RHEL6/x86_64)
        find_library(BOOST_LIBRARY3 libboost_program_options.a HINTS ${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/RHEL6/x86_64)
        find_library(BOOST_LIBRARY4 libboost_regex.a HINTS ${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/Boost/boost_1_59_0/lib/RHEL6/x86_64)
    endif()
endif()

# Other libraries
if(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
    find_library(TINYXML_LIB "tinyxml2.lib" HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/tinyxml2/Lib/VS2015/Win64/Release/MD")
    find_library(YAMLCPP_LIB "libyaml-cppmd.lib" HINTS "${PROJECT_SOURCE_DIR}/../../external/yaml-cpp/lib/win64/Release")
    find_library(SPIRVCROSS_LIB "spirv-cross-core.lib" HINTS "${PROJECT_SOURCE_DIR}/../../external/vulkan/tools/windows/lib/Release")
    find_library(TINYXML_LIBD "tinyxml2-d.lib" HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/tinyxml2/Lib/VS2015/Win64/Debug/MDd")
    find_library(YAMLCPP_LIBD "libyaml-cppmdd.lib" HINTS "${PROJECT_SOURCE_DIR}/../../external/yaml-cpp/lib/win64/Debug")
    find_library(SPIRVCROSS_LIBD "spirv-cross-core.lib" HINTS "${PROJECT_SOURCE_DIR}/../../external/vulkan/tools/windows/lib/Debug")
else()
    find_library(TINYXML_LIB "libtinyxml2.a" HINTS "${PROJECT_SOURCE_DIR}/../../../Common/Lib/Ext/tinyxml2/Lib/Linux/x64/Release")
    if (CMAKE_COMPILER_IS_GNUCC AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER 5.0)
        find_library(YAMLCPP_LIB "libyaml-cpp.a" HINTS "${PROJECT_SOURCE_DIR}/../../external/yaml-cpp/lib/linux/gcc5")
    else()
        find_library(YAMLCPP_LIB "libyaml-cpp.a" HINTS "${PROJECT_SOURCE_DIR}/../../external/yaml-cpp/lib/linux/gcc4")
    endif()
    find_library(SPIRVCROSS_LIB "libspirv-cross-core.a" HINTS "${PROJECT_SOURCE_DIR}/../../external/vulkan/tools/linux/lib")
endif()

# Set the name of the executable
if(AMD_INTERNAL)
    if(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
        set_target_properties(radeon_gpu_analyzer_cli PROPERTIES OUTPUT_NAME rga-internal)
    elseif(${CMAKE_SYSTEM_NAME} STREQUAL "Linux" OR ${CMAKE_SYSTEM_NAME} STREQUAL "Darwin")
        # On Linux we have a launch script that configures the environment prior to launching the executable.
        set_target_properties(radeon_gpu_analyzer_cli PROPERTIES OUTPUT_NAME rga-bin-internal)
    endif()
else()
    if(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
        set_target_properties(radeon_gpu_analyzer_cli PROPERTIES OUTPUT_NAME rga)
    elseif(${CMAKE_SYSTEM_NAME} STREQUAL "Linux" OR ${CMAKE_SYSTEM_NAME} STREQUAL "Darwin")
        # On Linux we have a launch script that configures the environment prior to launching the executable.
        set_target_properties(radeon_gpu_analyzer_cli PROPERTIES OUTPUT_NAME rga-bin)
    endif()
endif()

# Link settings
if(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
    target_link_libraries (radeon_gpu_analyzer_cli radeon_gpu_analyzer_backend AMDTOSWrappers AMDTBaseTools
                           optimized ${BOOST_LIBRARY1}
                           optimized ${BOOST_LIBRARY2}
                           optimized ${BOOST_LIBRARY3}
                           optimized ${BOOST_LIBRARY4}
                           optimized ${TINYXML_LIB}
                           optimized ${YAMLCPP_LIB}
                           optimized ${SPIRVCROSS_LIB}
                           debug ${BOOST_LIBRARY1D}
                           debug ${BOOST_LIBRARY2D}
                           debug ${BOOST_LIBRARY3D}
                           debug ${BOOST_LIBRARY4D}
                           debug ${TINYXML_LIBD}
                           debug ${YAMLCPP_LIBD}
                           debug ${SPIRVCROSS_LIBD}
                           general ${UPDATECHECKAPI_LIBS})
else()
    target_link_libraries (radeon_gpu_analyzer_cli radeon_gpu_analyzer_backend AMDTOSWrappers AMDTBaseTools
                           ${Boost_LIBRARIES} ${TINYXML_LIB} ${YAMLCPP_LIB} ${SPIRVCROSS_LIB} ${UPDATECHECKAPI_LIBS})
endif()

# Configure the post-build actions.
set(INTERNAL_ARG "")
if(AMD_INTERNAL)
    set(INTERNAL_ARG "-internal")
endif()

if(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
    set(DEBUG_ARG "-debug")
    set(REL_ARG "")
    # Use Windows path separater to ensure path string is valid for xcopy commands
    add_custom_command(TARGET radeon_gpu_analyzer_cli
                       POST_BUILD
                       COMMAND ${PROJECT_SOURCE_DIR}/../../build/util/windows/copy_post_build_cli.bat ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}\\ $<$<CONFIG:debug>:-debug> ${INTERNAL_ARG}
                       WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}\\..\\..\\build\\util\\windows
                      )
elseif(${CMAKE_SYSTEM_NAME} STREQUAL "Linux" OR ${CMAKE_SYSTEM_NAME} STREQUAL "Darwin")
    add_custom_command(TARGET radeon_gpu_analyzer_cli
                       POST_BUILD
                       COMMAND /bin/sh ${PROJECT_SOURCE_DIR}/../../build/util/linux/copy_post_build_cli.sh ${CMAKE_RUNTIME_OUTPUT_DIRECTORY} ${INTERNAL_ARG}
                       WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/../../build/util/linux
                      )
endif()

# Copy VC redist files into build folder.
IF(WIN32)
    # Copy the VisualStudio redist files
    # the list of all redist files is contained in the variable CMAKE_INSTALL_SYSTEM_RUNTIME_LIBS. We only want to copy
    # a subset of these so we extract the directory from the first entry and then manually copy the files we want
    foreach(VSLIB ${CMAKE_INSTALL_SYSTEM_RUNTIME_LIBS})
        message("VSLIB is ${VSLIB}")
        get_filename_component(VSREDISTDIR ${VSLIB} DIRECTORY)
        message("Visual Studio redistributable files directory = ${VSREDISTDIR}")
        break()
    endforeach()

    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E echo "copying Visual Studio redist files from ${VSREDISTDIR} to $<TARGET_FILE_DIR:${PROJECT_NAME}>"
        COMMAND ${CMAKE_COMMAND} -E copy_if_different ${VSREDISTDIR}/msvcp140.dll $<TARGET_FILE_DIR:${PROJECT_NAME}>
        COMMAND ${CMAKE_COMMAND} -E copy_if_different ${VSREDISTDIR}/concrt140.dll $<TARGET_FILE_DIR:${PROJECT_NAME}>
        COMMAND ${CMAKE_COMMAND} -E copy_if_different ${VSREDISTDIR}/vccorlib140.dll $<TARGET_FILE_DIR:${PROJECT_NAME}>
        COMMAND ${CMAKE_COMMAND} -E copy_if_different ${VSREDISTDIR}/vcruntime140.dll $<TARGET_FILE_DIR:${PROJECT_NAME}>
        COMMAND ${CMAKE_COMMAND} -E copy_if_different ${VSREDISTDIR}/vcruntime140_1.dll $<TARGET_FILE_DIR:${PROJECT_NAME}>
    )
ENDIF(WIN32)

