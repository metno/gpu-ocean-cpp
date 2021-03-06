cmake_minimum_required(VERSION 2.8)
PROJECT(gpu_ocean)
INCLUDE(CTest)

SET(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/Modules" CACHE STRING " ")

# We probably don't want this to run on every build.
option(COVERALLS "Generate coveralls data" OFF)

if (COVERALLS)
    include(Coveralls)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O0 -fprofile-arcs -ftest-coverage")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -O0 -fprofile-arcs -ftest-coverage")

    set(COVERAGE_SRCS 
        sim/src/oclutils.cpp sim/src/simbase.cpp sim/src/simulator.cpp sim/src/testsim.cpp)

    # Create the coveralls target.
    coveralls_setup(
        "${COVERAGE_SRCS}" # The source files.
        ON                 # If we should upload.
    )
endif()

IF(NOT CMAKE_BUILD_TYPE)
	SET(CMAKE_BUILD_TYPE "Release" CACHE STRING " " FORCE)
ENDIF()
IF(NOT LIBRARY_OUTPUT_PATH)
  SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
ENDIF()
IF(NOT EXECUTABLE_OUTPUT_PATH)
  SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
ENDIF()

#Set compilation flags
IF(MSVC10)
	#Enable multiprocessor compilation for speed
	IF(NOT CMAKE_CXX_FLAGS MATCHES "/MP")
		SET(CMAKE_CXX_FLAGS " /MP ${CMAKE_CXX_FLAGS}" CACHE STRING " " FORCE)
	ENDIF()
ENDIF()
IF(NOT CMAKE_HOST_WIN32)
	# Enable position independent code for linux build
	IF(NOT CMAKE_CXX_FLAGS MATCHES "-fPIC")
		SET(CMAKE_CXX_FLAGS "-fPIC ${CMAKE_CXX_FLAGS}" CACHE STRING " " FORCE)
	ENDIF()
ENDIF()

# Enable C++11
SET(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}" CACHE STRING " " FORCE)

SET(Boost_USE_MULTITHREADED ON)

#Set the paths we know
FIND_LIBRARY(mgr_LIBRARY_DEBUG
	NAMES "mgrd.lib" "libmgrd.a"
	PATHS "${PROJECT_SOURCE_DIR}/mgr/lib/")
FIND_LIBRARY(mgr_LIBRARY_RELEASE
	NAMES "mgr.lib" "libmgr.a"
	PATHS "${PROJECT_SOURCE_DIR}/mgr/lib/")	
IF(NOT mgr_LIBRARY_DEBUG AND NOT mgr_LIBRARY_RELEASE) 
	SET(mgr_LIBRARIES mgr CACHE FILEPATH " " FORCE)
	LINK_DIRECTORIES(${LIBRARY_OUTPUT_PATH})
ELSE()
	SET(mgr_LIBRARIES debug ${mgr_LIBRARY_DEBUG} optimized ${mgr_LIBRARY_RELEASE} CACHE FILEPATH "Path to mgr libraries")
ENDIF()

SET(mgr_INCLUDE_DIR "${PROJECT_SOURCE_DIR}/mgr/src" CACHE PATH " ")

FIND_LIBRARY(sim_LIBRARY_DEBUG
	NAMES "simd.lib" "libsimd.a"
	PATHS "${PROJECT_SOURCE_DIR}/sim/lib/")
FIND_LIBRARY(sim_LIBRARY_RELEASE
	NAMES "sim.lib" "libsim.a"
	PATHS "${PROJECT_SOURCE_DIR}/sim/lib/")	
IF(NOT sim_LIBRARY_DEBUG AND NOT sim_LIBRARY_RELEASE) 
	SET(sim_LIBRARIES sim CACHE FILEPATH " " FORCE)
	LINK_DIRECTORIES(${LIBRARY_OUTPUT_PATH})
ELSE()
	SET(sim_LIBRARIES debug ${sim_LIBRARY_DEBUG} optimized ${sim_LIBRARY_RELEASE} CACHE FILEPATH "Path to sim libraries")
ENDIF()

SET(sim_INCLUDE_DIR "${PROJECT_SOURCE_DIR}/sim/src" CACHE PATH " ")

ADD_SUBDIRECTORY(sim)
ADD_SUBDIRECTORY(mgr)
ADD_SUBDIRECTORY(apps)

INSTALL(DIRECTORY . DESTINATION .
	PATTERN ".svn" EXCLUDE
	PATTERN "build*" EXCLUDE
	PATTERN "mgr" EXCLUDE
	PATTERN "sim" EXCLUDE)
INSTALL(CODE "FILE(WRITE \"${CMAKE_INSTALL_PREFIX}/mgr/CMakelists.txt\" \" \")")
INSTALL(CODE "FILE(WRITE \"${CMAKE_INSTALL_PREFIX}/sim/CMakelists.txt\" \" \")")
