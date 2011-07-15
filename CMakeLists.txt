
cmake_policy(SET CMP0003 NEW)
cmake_policy(SET CMP0005 NEW)

cmake_minimum_required(VERSION 2.8)

set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin CACHE INTERNAL "" FORCE )
set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib CACHE INTERNAL "" FORCE )

# needed for python lib
if(CMAKE_COMPILER_IS_GNUCC)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
endif()

# -----------------------------------------------------------------------------
# configure python

if(NOT DEFINED PYTHON_ROOT_DIR)
	execute_process(
		COMMAND python3 -c "from distutils.sysconfig import PREFIX; print(PREFIX)"
		OUTPUT_VARIABLE _prefix
		OUTPUT_STRIP_TRAILING_WHITESPACE
	)
	set(PYTHON_ROOT_DIR "${_prefix}" CACHE PATH INTERNAL)
	unset(_site_packages)
endif()

if(NOT DEFINED PYTHON_SITE_PACKAGES)
	execute_process(
		COMMAND python3 -c "from distutils.sysconfig import get_python_lib; print(get_python_lib())"
		OUTPUT_VARIABLE _site_packages
		OUTPUT_STRIP_TRAILING_WHITESPACE
	)
	set(PYTHON_SITE_PACKAGES "${_site_packages}" CACHE PATH INTERNAL)
	unset(_site_packages)
endif()

if(NOT DEFINED PYTHON_INCLUDE_DIR)
	execute_process(
		COMMAND python3 -c "from distutils.sysconfig import get_python_inc; print(get_python_inc())"
		OUTPUT_VARIABLE _inc_dir
		OUTPUT_STRIP_TRAILING_WHITESPACE
	)
	set(PYTHON_INCLUDE_DIR "${_inc_dir}" CACHE PATH INTERNAL)
	unset(_site_packages)
endif()

if(NOT DEFINED PYTHON_LIBRARY)
	# library dir
	execute_process(
		COMMAND python3 -c "from distutils.sysconfig import PREFIX; import os; print(os.path.join(PREFIX, 'lib'))"
		OUTPUT_VARIABLE _lib_dir
		OUTPUT_STRIP_TRAILING_WHITESPACE
	)
	execute_process(
		COMMAND python3 -c "from distutils.sysconfig import get_python_version, PREFIX; import sys; print('libpython%s%s' % (get_python_version(), sys.abiflags))"
		OUTPUT_VARIABLE _lib_name
		OUTPUT_STRIP_TRAILING_WHITESPACE
	)

	find_library(PYTHON_LIBRARY
		NAMES
			"${_lib_name}"
		PATHS
			"${_lib_dir}"
	)

	unset(_lib_dir)
	unset(_lib_name)	
endif()

# set standard cmake/python names, uncached
set(PYTHON_LIBRARIES "${PYTHON_LIBRARY}")
set(PYTHON_INCLUDE_DIRS "${PYTHON_INCLUDE_DIR}")


# -----------------------------------------------------------------------------
# stubs

set(SRC
	src/stubs/stubs.c

	src/stubs/BLI_memarena.h
	src/stubs/MEM_guardedalloc.h
	src/stubs/MEM_sys_types.h
)

include_directories(
	src/stubs
)

add_library(stubs_lib ${SRC})

# -----------------------------------------------------------------------------
# blenlib

add_definitions(-DMATH_STANDALONE)

set(SRC
	src/blenlib/intern/math_base.c
	src/blenlib/intern/math_base_inline.c
	src/blenlib/intern/math_color.c
	src/blenlib/intern/math_geom.c
	src/blenlib/intern/math_geom_inline.c
	src/blenlib/intern/math_matrix.c
	src/blenlib/intern/math_rotation.c
	src/blenlib/intern/math_vector.c
	src/blenlib/intern/math_vector_inline.c

	src/blenlib/BLI_math.h
	src/blenlib/BLI_math_base.h
	src/blenlib/BLI_math_color.h
	src/blenlib/BLI_math_geom.h
	src/blenlib/BLI_math_inline.h
	src/blenlib/BLI_math_matrix.h
	src/blenlib/BLI_math_rotation.h
	src/blenlib/BLI_math_vector.h
	src/blenlib/BLI_utildefines.h
)

include_directories(
	src/blenlib
	${PYTHON_INCLUDE_DIRS}
)


add_library(blenlib_lib ${SRC})


# -----------------------------------------------------------------------------
# mathutils

set(SRC
	src/mathutils/mathutils.c
	src/mathutils/mathutils_Color.c
	src/mathutils/mathutils_Euler.c
	src/mathutils/mathutils_Matrix.c
	src/mathutils/mathutils_Quaternion.c
	src/mathutils/mathutils_Vector.c
	src/mathutils/mathutils_geometry.c

	src/mathutils/mathutils.h
	src/mathutils/mathutils_Color.h
	src/mathutils/mathutils_Euler.h
	src/mathutils/mathutils_Matrix.h
	src/mathutils/mathutils_Quaternion.h
	src/mathutils/mathutils_Vector.h
	src/mathutils/mathutils_geometry.h
)

include_directories(
	src/mathutils
)

# -----------------------------------------------------------------------------
# python module

add_library(mathutils_module SHARED ${SRC})

set_target_properties(
	mathutils_module
	PROPERTIES
		PREFIX ""
		OUTPUT_NAME mathutils
		LINKER_LANGUAGE C
		LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
		RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin  # only needed on windows
)

if(WIN32)
	# python modules use this
	set_target_properties(
		mathutils_module
		PROPERTIES
		SUFFIX ".pyd"
	)	
endif()

# link
target_link_libraries(mathutils_module
	stubs_lib
	blenlib_lib
)

# -----------------------------------------------------------------------------
# installation

install(
	TARGETS mathutils_module
	LIBRARY
	DESTINATION
		"${PYTHON_SITE_PACKAGES}"
)