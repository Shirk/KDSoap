set(QT_USE_QTNETWORK TRUE)
include(${QT_USE_FILE})
set(CMAKE_INCLUDE_CURRENT_DIR ON)

include_directories(.. ../KDSoapClient)

set(SOURCES
  KDSoapDelayedResponseHandle.cpp
  KDSoapServer.cpp
  KDSoapServerObjectInterface.cpp
  KDSoapServerSocket.cpp
  KDSoapServerThread.cpp
  KDSoapServerThread.cpp
  KDSoapServerThread.cpp
  KDSoapServerAuthInterface.cpp
  KDSoapServerRawXMLInterface.cpp
  KDSoapServerCustomVerbRequestInterface.cpp
  KDSoapSocketList.cpp
  KDSoapThreadPool.cpp
)

set_source_files_properties(KDSoapServerObjectInterface.cpp PROPERTIES SKIP_AUTOMOC TRUE)
add_library(kdsoap-server ${KDSoap_LIBRARY_MODE} ${SOURCES})
if(${PROJECT_NAME}_STATIC)
  target_compile_definitions(kdsoap-server PUBLIC KDSOAPSERVER_STATICLIB)
else()
  target_compile_definitions(kdsoap-server PRIVATE KDSOAP_BUILD_KDSOAPSERVER_LIB)
endif()
target_link_libraries(kdsoap-server kdsoap ${QT_LIBRARIES})
target_include_directories(kdsoap-server INTERFACE "$<INSTALL_INTERFACE:${INSTALL_INCLUDE_DIR}>")
set_target_properties(kdsoap-server PROPERTIES
  SOVERSION ${${PROJECT_NAME}_SOVERSION}
  VERSION ${${PROJECT_NAME}_VERSION}
)

#version libraries on Windows
if(WIN32)
  set(postfix ${${PROJECT_NAME}_SOVERSION})
  set(CMAKE_RELEASE_POSTFIX ${postfix})
  set_target_properties(kdsoap-server PROPERTIES RELEASE_POSTFIX ${CMAKE_RELEASE_POSTFIX})
  #append 'd' to debug libraries
  string(CONCAT postfix ${postfix} "d")
  set(CMAKE_DEBUG_POSTFIX ${postfix})
  set_target_properties(kdsoap-server PROPERTIES DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX})
endif()

if(KDSoap_IS_ROOT_PROJECT)
  include(ECMGenerateHeaders)
  ecm_generate_headers(server_HEADERS
    ORIGINAL
      CAMELCASE
    HEADER_NAMES
      KDSoapDelayedResponseHandle
      KDSoapServerGlobal
      KDSoapThreadPool
      KDSoapServerObjectInterface
      KDSoapServerAuthInterface
      KDSoapServerRawXMLInterface
      KDSoapServerCustomVerbRequestInterface
    COMMON_HEADER
      KDSoapServer
  )

  set(COMBINED_H ${CMAKE_CURRENT_BINARY_DIR}/KDSoapServer)
  file(APPEND ${COMBINED_H} "#include \"KDSoapServer.h\"\n")

  install(FILES
    ${server_HEADERS}
    KDSoapServer.h
    KDSoapServerAuthInterface.h
    KDSoapServerRawXMLInterface.h
    KDSoapServerCustomVerbRequestInterface.h
    KDSoapDelayedResponseHandle.h
    KDSoapServerObjectInterface.h
    KDSoapServerGlobal.h
    KDSoapThreadPool.h
    DESTINATION ${INSTALL_INCLUDE_DIR}/KDSoapServer
  )

  install(TARGETS kdsoap-server EXPORT KDSoapTargets
    RUNTIME DESTINATION ${INSTALL_RUNTIME_DIR}
    LIBRARY DESTINATION ${INSTALL_LIBRARY_DIR}
    ARCHIVE DESTINATION ${INSTALL_ARCHIVE_DIR}
  )

  if(MSVC AND NOT ${PROJECT_NAME}_STATIC)
    install(FILES "$<TARGET_PDB_FILE_DIR:kdsoap-server>/$<TARGET_PDB_FILE_NAME:kdsoap-server>" DESTINATION ${INSTALL_LIBRARY_DIR} CONFIGURATIONS Debug RelWithDebInfo)
  endif()

endif()
