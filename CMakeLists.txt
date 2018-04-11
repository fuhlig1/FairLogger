################################################################################
#    Copyright (C) 2018 GSI Helmholtzzentrum fuer Schwerionenforschung GmbH    #
#                                                                              #
#              This software is distributed under the terms of the             #
#              GNU Lesser General Public Licence (LGPL) version 3,             #
#                  copied verbatim in the file "LICENSE"                       #
################################################################################

cmake_minimum_required(VERSION 3.9.4 FATAL_ERROR)


# Project ######################################################################
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)
include(FairLoggerLib)

set_fairlogger_cmake_policies()
get_git_version(OUTVAR_PREFIX FairLogger)

project(FairLogger VERSION ${FairLogger_VERSION} LANGUAGES CXX)
message("${BWhite}${PROJECT_NAME}${CR} ${FairLogger_GIT_VERSION} from ${FairLogger_DATE}")

set_fairlogger_defaults()

include(CTest)
################################################################################


# Targets ######################################################################
# Configure Version.h
configure_file(logger/Version.h.in
  ${CMAKE_BINARY_DIR}/logger/Version.h
  @ONLY
)

add_library(FairLogger SHARED
  logger/Logger.cxx
  logger/Logger.h
)

target_include_directories(FairLogger
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/logger>
    $<INSTALL_INTERFACE:include>
)

if(BUILD_TESTING)
  add_executable(loggerTest test/loggerTest.cxx)
  target_link_libraries(loggerTest FairLogger pthread)
endif()
################################################################################


# Installation #################################################################
if(BUILD_TESTING)
  set(test_targets ${targets} loggerTest)
endif()
install(TARGETS
  FairLogger
  ${test_targets}

  EXPORT ${FairLogger_EXPORT_SET}
  LIBRARY DESTINATION ${FairLogger_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${FairLogger_INSTALL_BINDIR}
)

install(FILES
  logger/Logger.h
  ${CMAKE_BINARY_DIR}/logger/Version.h

  DESTINATION ${FairLogger_INSTALL_INCDIR}
)

install_fairlogger_cmake_package()
################################################################################


# Testing ######################################################################
if(BUILD_TESTING)
  add_test(NAME loggerTest
    COMMAND $<TARGET_FILE:loggerTest>
  )
endif()
################################################################################


# Summary ######################################################################
message("  ")
message("  ${Cyan}COMPONENT  BUILT?  INFO${CR}")
message("  ${BWhite}library${CR}     ${BGreen}YES${CR}    (default, always built)")
if(BUILD_TESTING)
  set(testing_summary "${BGreen}YES${CR}    (default, disable with ${BMagenta}-DBUILD_TESTING=OFF${CR})")
else()
  set(testing_summary "${BRed} NO${CR}    (enable with ${BMagenta}-DBUILD_TESTING=ON${CR})")
endif()
message("  ${BWhite}tests${CR}       ${testing_summary}")
message("  ")
################################################################################