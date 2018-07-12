PROJECT(WSIArithmetic)

SET(WSIArithmetic_src
  WSIArithmetic.cpp
)
ADD_EXECUTABLE(WSIArithmetic ${WSIArithmetic_src})

# SET_TARGET_PROPERTIES(WSIArithmetic PROPERTIES DEBUG_POSTFIX _d)
# TARGET_LINK_LIBRARIES(WSIArithmetic PRIVATE wholeslidefilters core multiresolutionimageinterface Boost::disable_autolinking Boost::program_options)
TARGET_LINK_LIBRARIES(WSIArithmetic wholeslidefilters core multiresolutionimageinterface Boost::disable_autolinking Boost::program_options)
TARGET_INCLUDE_DIRECTORIES(WSIArithmetic PRIVATE
  .
  ../../
  ${Boost_INCLUDE_DIRS}
  ${WSIArithmetic_SOURCE_DIR}/../../io/multiresolutionimageinterface
  ${WHOLESLIDEFILTERS_SOURCE_DIR}
  ${DIAGPathology_BINARY_DIR}
  ${CORE_BINARY_DIR}
  ${MULTIRESOLUTIONIMAGEINTERFACE_BINARY_DIR}
  ${WHOLESLIDEFILTERS_BINARY_DIR}
)
# TARGET_COMPILE_DEFINITIONS(WSIArithmetic PRIVATE -DBOOST_ALL_DYN_LINK)
SET_TARGET_PROPERTIES(WSIArithmetic PROPERTIES
                      RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR}/bin
					  RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}/bin
					#  PREFIX "./"
					  DEBUG_POSTFIX "d"
					  )

INSTALL(
  TARGETS WSIArithmetic 
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
)

IF(WIN32)
  SET_TARGET_PROPERTIES(WSIArithmetic PROPERTIES FOLDER pathology/diag/executables)   
ENDIF(WIN32)
