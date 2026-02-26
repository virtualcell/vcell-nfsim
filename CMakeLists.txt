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

# -----------------------------------------------------------------------------
# Model tests (generated from models/* READMEs)
# Each test runs BioNetGen (BNG2.pl in repo root) to generate XML, then runs NFsim.
# We set the working directory to each model's folder so relative paths in RNF files work.
# -----------------------------------------------------------------------------
enable_testing()

# Paths to tools
set(BNG2_PL ${CMAKE_SOURCE_DIR}/BNG2.pl)
set(NFSIM_EXE $<TARGET_FILE:NFsim>)

# Helper macro to create a test that first generates XML then runs NFsim
# Usage: add_nfsim_model_test(TestName ModelDir BnglFile [RNF|XML] ArgFile [ExtraArgs...])
macro(add_nfsim_model_test TEST_NAME MODEL_DIR BNGL_FILE MODE ARG_FILE)
    # Compose a single shell command to ensure XML generation precedes NFsim run within one test
    if("${MODE}" STREQUAL "RNF")
        add_test(
            NAME ${TEST_NAME}
            COMMAND ${CMAKE_COMMAND} -E env bash -lc "perl \"${BNG2_PL}\" \"${BNGL_FILE}\" && \"${NFSIM_EXE}\" -rnf \"${ARG_FILE}\""
        )
    else()
        # MODE == XML: run NFsim directly on the generated XML
        add_test(
            NAME ${TEST_NAME}
            COMMAND ${CMAKE_COMMAND} -E env bash -lc "perl \"${BNG2_PL}\" \"${BNGL_FILE}\" && \"${NFSIM_EXE}\" -xml \"${ARG_FILE}\""
        )
    endif()
    set_tests_properties(${TEST_NAME} PROPERTIES WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/${MODEL_DIR})
endmacro()

# actin: use actin_simple + RNF
add_nfsim_model_test(Model_actin_simple models/actin actin_simple.bngl RNF run_actin_simple.rnf)

# tlbr: use tlbr.bngl + tlbr.rnf
add_nfsim_model_test(Model_tlbr models/tlbr tlbr.bngl RNF tlbr.rnf)

# chemotaxisAdaptation: AN.bngl + run_AN.rnf
add_nfsim_model_test(Model_chemotaxis_AN models/chemotaxisAdaptation AN.bngl RNF run_AN.rnf)

# multisite_phos: e1.bngl then run via -xml e1.xml
add_nfsim_model_test(Model_multisite_e1 models/multisite_phos e1.bngl XML e1.xml)

# flagellar_motor: motor.bngl run via -xml motor.xml
add_nfsim_model_test(Model_flagellar_motor models/flagellar_motor motor.bngl XML motor.xml)

# oscillatory_system: oscSystem.bngl -> oscSystem.xml
add_nfsim_model_test(Model_oscillatory_system models/oscillatory_system oscSystem.bngl XML oscSystem.xml)

# simple_PushPull: PushPull.bngl -> PushPull.xml
add_nfsim_model_test(Model_simple_PushPull models/simple_PushPull PushPull.bngl XML PushPull.xml)

# fceRI_compendium: choose a representative small model (fceri_fyn.bngl)
add_nfsim_model_test(Model_fceRI_fyn models/fceRI_compendium fceri_fyn.bngl XML fceri_fyn.xml)

# performance_test_models: push_pull example with modest steps to limit runtime
# Note: Keeping README defaults (100 steps/seconds). Adjust if CI timeouts occur.
add_test(
    NAME Model_performance_egfr
    COMMAND ${CMAKE_COMMAND} -E env bash -lc "perl \"${BNG2_PL}\" egfr_net.bngl && \"${NFSIM_EXE}\" -xml egfr_net.xml -oSteps 100 -sim 100 -notf -utl 2"
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/models/performance_test_models
)


