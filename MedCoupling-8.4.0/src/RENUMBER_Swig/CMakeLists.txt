# Copyright (C) 2012-2016  CEA/DEN, EDF R&D
#
# This library is free software; you can redistribute it and/or
# modify it under the terms of the GNU Lesser General Public
# License as published by the Free Software Foundation; either
# version 2.1 of the License, or (at your option) any later version.
#
# This library is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public
# License along with this library; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307 USA
#
# See http://www.salome-platform.org/ or email : webmaster.salome@opencascade.com
#

INCLUDE(${SWIG_USE_FILE})

ADD_DEFINITIONS(${PYTHON_DEFINITIONS} ${NUMPY_DEFINITIONS} ${SCIPY_DEFINITIONS})

SET_SOURCE_FILES_PROPERTIES(MEDRenumber.i PROPERTIES CPLUSPLUS ON)
IF ("${PYTHON_VERSION_MAJOR}" STREQUAL "3")
  SET_SOURCE_FILES_PROPERTIES(MEDRenumber.i PROPERTIES SWIG_FLAGS "-py3")
ELSE()
  SET_SOURCE_FILES_PROPERTIES(MEDRenumber.i PROPERTIES SWIG_DEFINITIONS "-shadow")
ENDIF()
SET(SWIG_MODULE_MEDRenumber_EXTRA_FLAGS "${NUMPY_DEFINITIONS};${SCIPY_DEFINITIONS};-DWITHOUT_AUTOFIELD")

IF(Boost_FOUND)
  SET(SWIG_MODULE_MEDRenumber_EXTRA_FLAGS -DHAS_BOOST ${SWIG_MODULE_MEDRenumber_EXTRA_FLAGS})
ENDIF(Boost_FOUND)

IF(METIS_FOUND)
  SET(SWIG_MODULE_MEDRenumber_EXTRA_FLAGS -DHAS_METIS ${SWIG_MODULE_MEDRenumber_EXTRA_FLAGS})
ENDIF(METIS_FOUND)

SET (MEDRenumber_SWIG_DPYS_FILES
    MEDRenumberCommon.i)

INCLUDE_DIRECTORIES(
  ${PYTHON_INCLUDE_DIRS}
  ${PTHREAD_INCLUDE_DIR} # pthread dependancy due to python2.7 library
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${NUMPY_INCLUDE_DIR}
  ${CMAKE_CURRENT_BINARY_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}/../RENUMBER
  ${CMAKE_CURRENT_SOURCE_DIR}/../MEDCoupling_Swig
  ${CMAKE_CURRENT_SOURCE_DIR}/../MEDCoupling
  ${CMAKE_CURRENT_SOURCE_DIR}/../INTERP_KERNEL
  ${CMAKE_CURRENT_SOURCE_DIR}/../INTERP_KERNEL/Bases
  ${CMAKE_CURRENT_SOURCE_DIR}/../INTERP_KERNEL/Geometric2D
  ${CMAKE_CURRENT_SOURCE_DIR}/../INTERP_KERNEL/ExprEval
  ${CMAKE_CURRENT_SOURCE_DIR}/../INTERP_KERNEL/GaussPoints
  )

SWIG_ADD_MODULE(MEDRenumber python MEDRenumber.i)
SWIG_LINK_LIBRARIES(MEDRenumber ${PYTHON_LIBRARIES} ${PLATFORM_LIBS} renumbercpp)
SWIG_CHECK_GENERATION(MEDRenumber)

IF(WIN32)
  SET_TARGET_PROPERTIES(_MEDRenumber PROPERTIES DEBUG_OUTPUT_NAME _MEDRenumber_d)
ENDIF(WIN32)
INSTALL(TARGETS ${SWIG_MODULE_MEDRenumber_REAL_NAME} DESTINATION ${MEDCOUPLING_INSTALL_PYTHON})

SALOME_INSTALL_SCRIPTS(${CMAKE_CURRENT_BINARY_DIR}/MEDRenumber.py ${MEDCOUPLING_INSTALL_PYTHON})

INSTALL(FILES MEDRenumber.i MEDRenumberCommon.i DESTINATION ${MEDCOUPLING_INSTALL_HEADERS})
INSTALL(FILES MEDRenumberTest.py DESTINATION ${MEDCOUPLING_INSTALL_SCRIPT_PYTHON})

SALOME_ACCUMULATE_ENVIRONMENT(PYTHONPATH ${CMAKE_CURRENT_BINARY_DIR}/../MEDCoupling_Swig)
SALOME_GENERATE_TESTS_ENVIRONMENT(tests_env)
ADD_TEST(MEDRenumberTest ${PYTHON_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/MEDRenumberTest.py)
SET_TESTS_PROPERTIES(MEDRenumberTest PROPERTIES ENVIRONMENT "${tests_env}")

# Application tests

SET(TEST_INSTALL_DIRECTORY ${MEDCOUPLING_INSTALL_TESTS}/RENUMBER_Swig)
INSTALL(FILES MEDRenumberTest.py DESTINATION ${TEST_INSTALL_DIRECTORY})

INSTALL(FILES CTestTestfileInstall.cmake
        DESTINATION ${TEST_INSTALL_DIRECTORY}
        RENAME CTestTestfile.cmake)
