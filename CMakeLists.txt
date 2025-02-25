CMAKE_MINIMUM_REQUIRED(VERSION 3.9)

#---------------------------------------------------------------------------------
# Define path variables
#---------------------------------------------------------------------------------

SET( DIR_1 "basis/include" )
SET( DIR_2 "preconditioner/include" )
SET( DIR_3 "mesh/include" )
SET( DIR_4 "input/include" )
SET( DIR_5 "timestep/include" )
SET( DIR_8 "error_estimator/include" )
SET( DIR_9 "post_process/include" )
SET( DIR_10 "elem_color/include" )
SET( DIR_11 "periodic_bc/include" )
SET( DIR_12 "projection/include" )
SET( DIR_6 ${TRILINOS_DIR}lib )
SET( DIR_7 ${CMAKE_CURRENT_SOURCE_DIR} )

#---------------------------------------------------------------------------------
# Give access to TrilinosConfig.cmake in order to access its variables
#---------------------------------------------------------------------------------

INCLUDE( ${DIR_6}/cmake/Trilinos/TrilinosConfig.cmake )

LIST( REMOVE_DUPLICATES Trilinos_LIBRARIES )
LIST( REMOVE_DUPLICATES Trilinos_TPL_LIBRARIES )

#---------------------------------------------------------------------------------
# Give access to all subdirectories ( basis, basis/include, etc. )
#---------------------------------------------------------------------------------

INCLUDE_DIRECTORIES( ${Trilinos_TPL_INCLUDE_DIRS} ${DIR_1} ${DIR_2} ${DIR_3} 
  ${DIR_4} ${DIR_5} ${DIR_8} ${DIR_9} ${DIR_10} ${DIR_11} ${DIR_12} ${DIR_1}/.. ${DIR_2}/.. 
  ${DIR_3}/.. ${DIR_4}/.. ${DIR_5}/.. ${DIR_8}/.. ${DIR_10}/.. ${DIR_11}/.. ${DIR_12}/.. )

#---------------------------------------------------------------------------------
# Change compiler. This must be done before PROJECT() specification
# We access the compiler variables from TrilinosConfig.cmake
#---------------------------------------------------------------------------------

SET( CMAKE_C_COMPILER ${Trilinos_C_COMPILER} )
SET( CMAKE_CXX_COMPILER ${Trilinos_CXX_COMPILER} )


#---------------------------------------------------------------------------------
# General information. Specify name and executable. 
# Let all contributing files feed into executable by saving them in SOURCES and 
# feeding the variable directly in ADD_EXECUTABLE()
#---------------------------------------------------------------------------------

SET( SOURCES 
${DIR_7}/tusas.cpp 	      		# Include DIR_7 which is the directory      #
${DIR_7}/mesh/Mesh.C			# that we are working in so that the other  #
${DIR_7}/mesh/include/Mesh.h		# paths can be based out of it              #
${DIR_7}/basis/basis.cpp 		
${DIR_7}/basis/include/basis.hpp 
${DIR_7}/input/ReadInput.cpp 
${DIR_7}/input/include/ReadInput.h 
${DIR_7}/error_estimator/error_estimator.cpp 
${DIR_7}/timestep/include/ModelEvaluatorNEMESIS_def.hpp
${DIR_7}/post_process/post_process.cpp 
${DIR_7}/post_process/include/post_process.h 
${DIR_7}/elem_color/elem_color.cpp 
${DIR_7}/periodic_bc/include/periodic_bc.h 
${DIR_7}/periodic_bc/periodic_bc.cpp 
${DIR_7}/projection/include/projection.h 
${DIR_7}/projection/projection.cpp 
${DIR_7}/projection/include/interpfluxavg.h 
${DIR_7}/projection/interpfluxavg.cpp 
${DIR_7}/projection/include/interpflux.h 
${DIR_7}/projection/interpflux.cpp 
)


PROJECT( TUSAS )

ADD_EXECUTABLE( tusas ${SOURCES} )


#see https://cmake.org/cmake/help/git-stage/policy/CMP0069.html
#include(CheckIPOSupported)
#check_ipo_supported(RESULT result)
#if(result)
#  message(STATUS "IPO / LTO enabled")
#  set_property(TARGET tusas PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
#endif()


#---------------------------------------------------------------------------------
# Provide access to the Trilinos Libraries. This is done by first linking the 
# directory that holds the libraries, and using TARGET_LINK_LIBRARIES() to show
# the relationship between the executable and the libraries
#---------------------------------------------------------------------------------

INCLUDE_DIRECTORIES( ${Trilinos_INCLUDE_DIRS} )

TARGET_LINK_LIBRARIES( tusas ${Trilinos_LIBRARIES} ${Trilinos_TPL_LIBRARIES} ${Trilinos_EXTRA_LD_FLAGS} )


  # Include Trilinos_INCLUDE_DIRS because many header files    #
  #  that are called in tusas.cpp exist here.	      	       #

#---------------------------------------------------------------------------------
# Hack for test coverage flags
#---------------------------------------------------------------------------------

#SET(GCC_COVERAGE_COMPILE_FLAGS "-fprofile-arcs -ftest-coverage")
#SET(GCC_COVERAGE_LINK_FLAGS    "-lgcov -fprofile-arcs")
#SET( CMAKE_C_FLAGS "${Trilinos_C_COMPILER_FLAGS} ${GCC_COVERAGE_COMPILE_FLAGS}" )
#SET( CMAKE_CXX_FLAGS "${Trilinos_CXX_COMPILER_FLAGS} ${TUSAS_CXX_FLAGS} ${GCC_COVERAGE_COMPILE_FLAGS}" )
#SET(GCC_COVERAGE_LINK_FLAGS    "${GCC_COVERAGE_LINK_FLAGS}")

#---------------------------------------------------------------------------------
# Change compiler flags. This must be done after  PROJECT() specification
#---------------------------------------------------------------------------------

SET( CMAKE_C_FLAGS ${Trilinos_C_COMPILER_FLAGS} )
SET( CMAKE_CXX_FLAGS "${Trilinos_CXX_COMPILER_FLAGS} ${TUSAS_CXX_FLAGS}" )


#---------------------------------------------------------------------------------
# Write tusas.h for use with Testing 
#---------------------------------------------------------------------------------

FILE ( WRITE ${DIR_7}/tusas.h "std::string TRILINOS_DIR=\"${TRILINOS_DIR}\"; ")





SET( DIR_TEST "Test" )

FILE ( COPY ${TRILINOS_DIR}/bin/exodiff DESTINATION ${CMAKE_CURRENT_SOURCE_DIR}/Test )

enable_testing()

add_test( NAME HeatQuad  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/HeatQuad COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME HeatQuadQ  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/HeatQuadQ COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME HeatHex  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/HeatHex COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME HeatHexNoPrec WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/HeatHexNoPrec COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME HeatTet  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/HeatTet COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME PhaseHeatQuadImp  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/PhaseHeatQuadImp COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME PhaseHeatQuadExp  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/PhaseHeatQuadExp COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME PhaseHeatQuad  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/PhaseHeatQuad COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME PhaseHeatQuadNoPrec  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/PhaseHeatQuadNoPrec COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME PhaseHeatTris  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/PhaseHeatTris COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME PhaseHeatQuadPar  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/PhaseHeatQuadPar COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME PhaseHeatQuadParNoPrec  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/PhaseHeatQuadParNoPrec COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME NeumannQuad  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/NeumannQuad COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME NeumannTriPar  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/NeumannTriPar COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME NeumannQuadQPar  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/NeumannQuadQPar COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME NeumannTriQ  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/NeumannTriQ COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME NeumannHex  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/NeumannHex COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME NeumannTetPar  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/NeumannTetPar COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME RobinQuad  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/RobinQuad COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME LinIsoHex  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/LinIsoHex COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME LinIsoBodyForceHexPrec  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/LinIsoBodyForceHexPrec COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME LinIsoHeatHexPar  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/LinIsoHeatHexPar COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME CoupledStressQuad  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/CoupledStressQuad COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME HeatTri  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/HeatTri COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME Uehara1  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/Uehara1 COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME Farzadi  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/Farzadi COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME CahnHilliardQuadPar  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/CahnHilliardQuadPar COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME CahnHilliardTriQPar  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/CahnHilliardTriQPar COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME RestartPar  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/RestartPar COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME PeriodicQuad  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/PeriodicQuad COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME PeriodicHexPar  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/PeriodicHexPar COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME WriteSkipDecomp  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/WriteSkipDecomp COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )

add_test( NAME HeatHexT  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Test/HeatHexT COMMAND run_test ${CMAKE_CURRENT_BINARY_DIR} )
