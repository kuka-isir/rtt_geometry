find_package(OROCOS-RTT)

#this option was set in global_rules.cmake
if(OROCOS-RTT_CORBA_FOUND)

  include( ${OROCOS-RTT_USE_FILE} )
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/config)
  set(ENABLE_CORBA TRUE)
  find_package(Corba REQUIRED)

  # for jacobian and jntarray
  find_package(cmake_modules REQUIRED)
  find_package(Eigen REQUIRED)
  include_directories(${Eigen_INCLUDE_DIRS})
  # Add includes / lib paths if necessary
#  IF( NOT ${ACE_DIR} STREQUAL /usr/include )
#    SET(CMAKE_C_FLAGS_RTT "${CMAKE_C_FLAGS_RTT} -I${ACE_DIR}" CACHE INTERNAL "")
#    SET(CMAKE_LINKFLAGS_RTT "${CMAKE_LINKFLAGS_RTT} -L${ACE_DIR}/../lib" CACHE INTERNAL "")
#  ENDIF( NOT ${ACE_DIR} STREQUAL /usr/include )
#  IF( NOT ${TAO_DIR} STREQUAL /usr/include )
#    SET(CMAKE_C_FLAGS_RTT "${CMAKE_C_FLAGS_RTT} -I${TAO_DIR}" CACHE INTERNAL "")
#  ENDIF( NOT ${TAO_DIR} STREQUAL /usr/include )
#  IF( NOT ${ORBSVCS_DIR} STREQUAL /usr/include )
#    SET(CMAKE_C_FLAGS_RTT "${CMAKE_C_FLAGS_RTT} -I${ORBSVCS_DIR}" CACHE INTERNAL "")
#  ENDIF( NOT ${ORBSVCS_DIR} STREQUAL /usr/include )

  FILE( GLOB IDLS [^.]*.idl )
  FILE( GLOB CPPS [^.]*.cpp )
  FILE( GLOB HPPS [^.]*.hpp [^.]*.h [^.]*.inl)

  ORO_ADD_CORBA_SERVERS(CPPS HPPS ${IDLS} )
  SET( ENV{GLOBAL_GENERATED_SRCS} "${CPPS}" )

  #This has been added in order to generate files before make -j<N>
  # make -j<N> causes corruption of generated files because of overwriting
  ADD_CUSTOM_TARGET( idl-kdl-corba DEPENDS ${CPPS} ${HPPS}
            COMMENT "Generating sources from .idl files")
            
  INCLUDE_DIRECTORIES( ${CMAKE_CURRENT_BINARY_DIR} )

  orocos_use_package(orocos_kdl)
  orocos_typekit(kdl-transport-corba SHARED ${CPPS})
  # Is used for building  the library
  TARGET_LINK_LIBRARIES(kdl-transport-corba ${OROCOS-RTT_CORBA_LIBRARIES})

  install(FILES ${HPPS} ${IDLS} DESTINATION include/orocos/kdl_typekit/transports/corba)

endif()

