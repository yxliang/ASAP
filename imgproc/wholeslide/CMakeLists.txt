PROJECT(WHOLESLIDEFILTERS)

SET(WHOLESLIDEFILTERS_SRCS 
  ConnectedComponentsWholeSlideFilter.h
  ConnectedComponentsWholeSlideFilter.cpp
  DistanceTransformWholeSlideFilter.h
  DistanceTransformWholeSlideFilter.cpp
  LabelStatisticsWholeSlideFilter.h
  LabelStatisticsWholeSlideFilter.cpp
  ThresholdWholeSlideFilter.h
  ThresholdWholeSlideFilter.cpp
  ArithmeticWholeSlideFilter.h
  ArithmeticWholeSlideFilter.cpp
  NucleiDetectionWholeSlideFilter.h
  NucleiDetectionWholeSlideFilter.cpp
)

ADD_LIBRARY(wholeslidefilters SHARED ${WHOLESLIDEFILTERS_SRCS})
TARGET_LINK_LIBRARIES(wholeslidefilters PRIVATE ${OpenCV_LIBS} annotation NucleiSegmentation basicfilters multiresolutionimageinterface core)
TARGET_INCLUDE_DIRECTORIES(
  wholeslidefilters PRIVATE
  ${MULTIRESOLUTIONIMAGEINTERFACE_SOURCE_DIR}
  ../../
  ${WHOLESLIDEFILTERS_SOURCE_DIR}
  ${Boost_INCLUDE_DIRS}
  ${DIAGPathology_BINARY_DIR}
  ${CMAKE_CURRENT_BINARY_DIR}
  ${CORE_BINARY_DIR}
  ${MULTIRESOLUTIONIMAGEINTERFACE_BINARY_DIR}
  ${BASICFILTERS_BINARY_DIR}
  ${NUCLEISEGMENTATION_BINARY_DIR}
  ${ANNOTATION_BINARY_DIR}
)
generate_export_header(wholeslidefilters)

SET_TARGET_PROPERTIES(wholeslidefilters PROPERTIES DEBUG_POSTFIX _d)
IF(WIN32)  
  SET_TARGET_PROPERTIES(wholeslidefilters PROPERTIES FOLDER pathology/diag/imgproc)    
ENDIF(WIN32)

INSTALL(FILES ConnectedComponentsWholeSlideFilter.h DistanceTransformWholeSlideFilter.h LabelStatisticsWholeSlideFilter.h ThresholdWholeSlideFilter.h ArithmeticWholeSlideFilter.h DESTINATION include/imgproc/wholeslidefilters)
INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/wholeslidefilters_export.h DESTINATION include/imgproc/wholeslidefilters)

INSTALL(TARGETS wholeslidefilters
  EXPORT asap-targets
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
)

OPTION(WRAP_WHOLESLIDEFILTERS_PYTHON "Wrap whole slide image processing in Python" OFF)

IF(WRAP_WHOLESLIDEFILTERS_PYTHON)
  add_definitions(-DSWIG_PYTHON_INTERPRETER_NO_DEBUG)
  find_package(PythonLibs REQUIRED)
  find_package(SWIG REQUIRED)
  include (${SWIG_USE_FILE})
  IF(SWIG_FOUND)
    if(NOT WIN32)
      set(CMAKE_SWIG_FLAGS "-I${Boost_INCLUDE_DIR}")
    endif(NOT WIN32)     
    MARK_AS_ADVANCED(SWIG_DIR SWIG_VERSION)
  ENDIF(SWIG_FOUND)

	# Wrapping part for Python
	IF (WRAP_WHOLESLIDEFILTERS_PYTHON AND SWIG_FOUND)   
	  # Set module extension, platform specific
	  IF(WIN32)
		SET(module_extension ".pyd")
	  ELSE(WIN32)
		SET(module_extension ".so")  
	  ENDIF(WIN32)
	  
	  set_source_files_properties(wholeslidefilters.i PROPERTIES CPLUSPLUS ON)
	  set_source_files_properties(wholeslidefilters.i PROPERTIES SWIG_FLAGS "-builtin")
	  include_directories(. ${PYTHON_INCLUDE_DIR} ${DIAGPathology_BINARY_DIR} ${CMAKE_CURRENT_BINARY_DIR})
	  SWIG_ADD_LIBRARY(wholeslidefilters LANGUAGE python SOURCES wholeslidefilters.i)
	  SWIG_LINK_LIBRARIES(wholeslidefilters wholeslidefilters core) 
	  SWIG_LINK_LIBRARIES(wholeslidefilters ${PYTHON_LIBRARIES} ) #${PYTHON_DEBUG_LIBRARIES})
	  
	  SET_TARGET_PROPERTIES(${SWIG_MODULE_wholeslidefilters_REAL_NAME} PROPERTIES FOLDER pathology/diag/imgproc)
	  SET_TARGET_PROPERTIES(${SWIG_MODULE_wholeslidefilters_REAL_NAME} PROPERTIES DEBUG_POSTFIX _d)
	  
	  # Cross-platform copying and installing instructions
	  INSTALL(FILES ${PROJECT_BINARY_DIR}/wholeslidefilters.py DESTINATION bin)

      INSTALL(TARGETS ${SWIG_MODULE_wholeslidefilters_REAL_NAME}
	  	EXPORT asap-targets
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION bin
        ARCHIVE DESTINATION lib
      )
	ENDIF(WRAP_WHOLESLIDEFILTERS_PYTHON AND SWIG_FOUND)  
ENDIF(WRAP_WHOLESLIDEFILTERS_PYTHON)