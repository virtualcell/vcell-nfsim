cmake_minimum_required(VERSION 3.13)
project(NFsim)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_EXTENSIONS OFF)

# Build settings
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Enable CXX and C
enable_language(CXX)
enable_language(C)

# Architecture flags
if (APPLE)
    execute_process(COMMAND uname -m OUTPUT_VARIABLE PROCESSOR_ARCHITECTURE OUTPUT_STRIP_TRAILING_WHITESPACE)
    if (PROCESSOR_ARCHITECTURE STREQUAL "arm64")
        add_definitions(-D__arm64__)
        set(CMAKE_OSX_ARCHITECTURES "arm64")
    else()
        add_definitions(-D__x86_64__)
        set(CMAKE_OSX_ARCHITECTURES "x86_64")
    endif()
elseif (UNIX)
    # Default to 64-bit on Linux/Unix
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -m64")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -m64")
endif()

# Source directories for NFsim
set(SUB_DIRS 
    src 
    src/nauty24 
    src/NFcore 
    src/NFcore/moleculeLists 
    src/NFcore/reactionSelector 
    src/NFfunction 
    src/NFfunction/muParser
    src/NFinput 
    src/NFinput/TinyXML
    src/NFoutput 
    src/NFreactions
    src/NFreactions/mappings
    src/NFreactions/reactantLists
    src/NFreactions/reactions
    src/NFreactions/transformations
    src/NFscheduler 
    src/NFutil 
    src/NFutil/MTrand
    src/NFtest
    src/NFtest/agentcell
    src/NFtest/agentcell/cell
    src/NFtest/simple_system
    src/NFtest/tlbr
    src/NFtest/transcription
)

# Use aux_source_directory for better control if needed, but file(GLOB) is also okay here
set(LIB_SRC_FILES)
foreach(DIR ${SUB_DIRS})
    file(GLOB FILES ${DIR}/*.cpp ${DIR}/*.c ${DIR}/*.hh ${DIR}/*.h)
    list(APPEND LIB_SRC_FILES ${FILES})
endforeach()

# Add a simple main if NFsim.cpp doesn't have one
set(MAIN_SRC ${CMAKE_BINARY_DIR}/nfsim_standalone_main.cpp)
file(WRITE ${MAIN_SRC} "
int NFsimMain(int argc, char *argv[]);
int main(int argc, char *argv[]) {
    return NFsimMain(argc, argv);
}
")

# Compile flags
if (UNIX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fpermissive")
endif()

if (MSVC)
    add_definitions(-DHAVE_UNISTD_H=0)
endif()

# Define the library
add_library(NFsim_lib STATIC ${LIB_SRC_FILES})

# Define the executable
add_executable(NFsim ${MAIN_SRC})
target_link_libraries(NFsim PRIVATE NFsim_lib)

# Include directories
foreach(DIR ${SUB_DIRS})
    target_include_directories(NFsim_lib PUBLIC ${DIR})
endforeach()

# Optional: Add smoke tests if they exist
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/tests/smoke/smoke.py)
    enable_testing()
    set(python_cmd python3)
    set(test_dir ${CMAKE_CURRENT_SOURCE_DIR}/tests/smoke)
    add_test(NAME NFsim_smoke_python COMMAND ${python_cmd} ${test_dir}/smoke.py ${CMAKE_BINARY_DIR}/bin/NFsim WORKING_DIRECTORY ${test_dir})
endif()


