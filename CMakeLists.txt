project( Pisces_Project C CXX Fortran )
cmake_minimum_required(VERSION 2.6)

# 

# ==============================================================================
#   FIND LIBRARIES 
# ==============================================================================
set( CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/site-macros/Modules )
find_package( BLAS REQUIRED )
find_package( LAPACK REQUIRED )
#find_package( FORTRANRTL REQUIRED )
find_package( Threads )  # for lapack on some platforms
# todo: cross platform and compiler check this:
list(APPEND LAPACK_LIBRARIES ${CMAKE_THREAD_LIBS_INIT} )


# ==============================================================================
#  SOURCE AND TARGET
# ==============================================================================

add_library( piscesf
  src/lbfgs.f
  )

# add source
set( src 
#  src/amoeba.c
  src/C60.cpp
  src/ChargeDipPol.cpp
  src/ClusterAnion.cpp
  src/cm_dvr.cpp
  src/DPP.cpp
  src/DVR.cpp
  src/davdriver.cpp
  src/Davidson.cpp
  src/DerivDDTensor.cpp
  src/GetInput.cpp
  src/GeoAux.cpp
  src/Gradients.cpp
  src/fulldiag.cpp
  src/ho_dvr.cpp
  src/KE_diag.cpp
  src/larnoldi.cpp
  src/Model_pot.cpp
  src/Molecule.cpp
  src/MolPolAux.cpp
  src/mtx.cpp
  src/NaCl.cpp
  src/optimize.cpp
  src/Parameters.cpp
  src/polplot.cpp
  src/Potential.cpp
  src/pisces.cpp
  src/potfit.cpp
#  src/Powell.cpp
  src/ReadCubeFile.cpp
  src/sine_dvr.cpp
  src/Small2Large.cpp
  src/tsin.cpp
  src/vtx_FFT.cpp
  src/VectorFFTW.cpp
  src/Water.cpp
  src/WriteCubeFile.cpp
  )

# add headers for VisualStudio and friends
if ( WIN32 ) 
  file(GLOB headers src/*.h*)
endif()
  
# Add main target

add_executable( pisces
  ${headers}
  ${src}
)
  
include_directories( ${PROJECT_SOURCE_DIR}/ext )
#target_link_libraries( pisces piscesf ${FORTRANRTL_LIBRARIES} ${LAPACK_LIBRARIES} )
target_link_libraries( pisces piscesf  ${FORTRANRTL_LIBRARIES} ${LAPACK_LIBRARIES} )


# ==============================================================================
#   MISC SETTINGS
# ==============================================================================
include(utils)

# 
# Optional Parallel build
#
option( PISCES_OPENMP "Build parallel PISCES using OpenMP" ON)
if ( PISCES_OPENMP )
  find_package( OpenMP )
  if ( OPENMP_FOUND )
    set_property( 
      TARGET  pisces 
      APPEND
      PROPERTY COMPILE_FLAGS ${OpenMP_CXX_FLAGS}
      )
    set_property( 
      TARGET pisces
      APPEND
      PROPERTY LINK_FLAGS ${OpenMP_CXX_FLAGS}
      )
  else ( OPENMP_FOUND )
    message(STATUS "Warning: Parallelization disabled. Compiler doesn't support OpenMP")
  endif ( OPENMP_FOUND )
endif( PISCES_OPENMP )

# Set default build type to optimized
set_build_type( Release )

# copy inputs for testing
##configure_file( test/w4_pol3_opt.inp ${PROJECT_BINARY_DIR} COPYONLY )
##configure_file( test/w12_pol3_opt.inp ${PROJECT_BINARY_DIR} COPYONLY )
##configure_file( test/w45_pol3_sp.inp ${PROJECT_BINARY_DIR} COPYONLY )


# Documentation block
option( PISCES_DOCS "Run doxygen to generate documentation" OFF)
if ( PISCES_DOCS )
  add_subdirectory( doc )
endif ( PISCES_DOCS )


# Documentation block
option( PISCES_PERFTOOLS "Copy performance tools" OFF)
if ( PISCES_PERFTOOLS )
  configure_file( test/perf/run.sh ${PROJECT_BINARY_DIR} COPYONLY )
endif ( PISCES_PERFTOOLS )

# ARPACK
# to help CMake find ARPACK, add
# -DARPACK_PATH=/path/to/arpack
option (HAVE_ARPACK "Is the ARPACK library installed" NO)
if (HAVE_ARPACK)
  find_package(ARPACK)
  if(ARPACK_FOUND)
    set_property(DIRECTORY PROPERTY COMPILE_DEFINITIONS HAVE_ARPACK)
    target_link_libraries(pisces ${ARPACK_LIBRARY})
  endif(ARPACK_FOUND)
endif(HAVE_ARPACK)

# FFTW
# to help CMake find FFTW, add
# -DFFTW_PATH=/path/to/fftw3
option (HAVE_FFTW "Is the FFTW library installed" NO)
if (HAVE_FFTW)
  find_package(FFTW)
  if(FFTW_FOUND)
    set_property(DIRECTORY PROPERTY COMPILE_DEFINITIONS HAVE_FFTW)
    include_directories(${FFTW_INCLUDES})
    target_link_libraries(pisces ${FFTW_LIBRARY} ${FFTW_OMP_LIBRARY})
  endif(FFTW_FOUND)
endif(HAVE_FFTW)

# add niceties for MS Visual Studio
if( WIN32)
  add_definitions( -D_CRT_SECURE_NO_WARNINGS )
  source_group( csrc REGULAR_EXPRESSION "\\.c$" )
  source_group( cxxsrc REGULAR_EXPRESSION "\\.cpp$" )
  source_group( hdr REGULAR_EXPRESSION "\\.h$" )
endif( WIN32)
