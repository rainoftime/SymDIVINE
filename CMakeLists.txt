cmake_minimum_required(VERSION 2.8)
project(SymDIVINE CXX C)

# Setup packages
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# Setup directories
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/ltl_parser)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)

# Import packages
find_package(BISON REQUIRED)
find_package(FLEX REQUIRED)
find_package(Threads REQUIRED)
find_package(LLVM REQUIRED)
find_package(Boost REQUIRED COMPONENTS regex graph)
find_package(LIBDL REQUIRED)
find_package(Curses REQUIRED)
find_package(Z3 REQUIRED)

# Check LLVM version
if (NOT ${LLVM_STRING_VERSION} MATCHES "3[.][45]([^0-9]*|[.].*)")
    message(FATAL_ERROR "LLVM 3.4 or 3.5 is required! Version ${LLVM_STRING_VERSION} found.")
endif()


# Setup C++ sources
file(GLOB BIN_SOURCES
	src/llvmsym/*.cpp
	src/llvmsym/formula/*.cpp
	src/llvmsym/llvmwrap/*.cpp
	src/llvmsym/programutils/*.cpp
	src/llvmsym/stanalysis/*.cpp
	src/toolkit/*.cpp
	)

set(DOCOPT_SOURCES libs/docopt/docopt.cpp)
set(Q3B_SOURCES libs/q3b/ExprSimplifier.cpp libs/q3b/UnconstrainedVariableSimplifier.cpp)


# Setup Bison/Flex sources
bison_target(LTL_PARSER src/parsers/ltl_parser.y ${CMAKE_CURRENT_BINARY_DIR}/ltl_parser/parser.cpp)
flex_target(LTL_SCANNER src/parsers/ltl_tokens.l ${CMAKE_CURRENT_BINARY_DIR}/ltl_parser/tokens.cpp COMPILE_FLAGS --header-file=${CMAKE_CURRENT_BINARY_DIR}/ltl_parser/ltl_tokens.h)
add_flex_bison_dependency(LTL_SCANNER LTL_PARSER)

# Include directories
include_directories(src libs)
include_directories(${LLVM_INCLUDE_DIRS})
include_directories(${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_BINARY_DIR}/ltl_parser)
include_directories(${Boost_INCLUDE_DIRS})
include_directories(${LIBDL_INCLUDE_DIRS})
include_directories(${Z3_INCLUDE_DIRS})

# Link directories
link_directories(${LLVM_LIBRARY_DIRS})

# Compiler COMPILE_FLAGS
add_definitions(${LLVM_COMPILE_FLAGS})

# Setup main binary
add_executable(symdivine src/symdivine.cpp ${BIN_SOURCES} ${DOCOPT_SOURCES}
	${BISON_LTL_PARSER_OUTPUTS} ${FLEX_LTL_SCANNER_OUTPUTS} ${Q3B_SOURCES})

# Setup libraries
# target_link_libraries(symdivine ${FLEX_LIBRARIES} ${BISON_LIBRARIES})
target_link_libraries(symdivine ${LLVM_LIBRARIES})
target_link_libraries(symdivine ${CMAKE_THREAD_LIBS_INIT})
target_link_libraries(symdivine ${Boost_LIBRARIES})
target_link_libraries(symdivine ${LIBDL_LIBRARIES})
target_link_libraries(symdivine ${CURSES_LIBRARIES})
target_link_libraries(symdivine ${Z3_LIBRARIES})

#set_target_properties(symdivine PROPERTIES LINK_SEARCH_START_STATIC 1)
#set_target_properties(symdivine PROPERTIES LINK_SEARCH_END_STATIC 1)
#set(CMAKE_FIND_LIBRARY_SUFFIXES ".a")

# Set C++11 standard
add_definitions("-std=c++11")
set_property(TARGET symdivine PROPERTY CXX_STANDARD 11)
set_property(TARGET symdivine PROPERTY CXX_STANDARD_REQUIRED ON)
set_property(TARGET symdivine APPEND_STRING PROPERTY COMPILE_FLAGS -Wall)
#set(CMAKE_EXE_LINKER_FLAGS "-static -static-libgcc -static-libstdc++")

#set(CMAKE_BUILD_TYPE Release)

message(STATUS "Configuration complete. To build SymDIVINE run 'cd build; make -j2'")
