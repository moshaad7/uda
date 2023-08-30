# Copyright 2014-Present Couchbase, Inc.
#
# Use of this software is governed by the Business Source License included in
# the file licenses/Couchbase-BSL.txt.  As of the Change Date specified in that
# file, in accordance with the Business Source License, use of this software will
# be governed by the Apache License, Version 2.0, included in the file
# licenses/APL.txt.

CMAKE_MINIMUM_REQUIRED (VERSION 3.13)
PROJECT (uda NONE)

# This project is intended to be built as part of a Couchbase Server build,
# but for nominal support building stand-alone, the following variables
# need to be specified:
#
# CMAKE_MODULE_PATH - should contain a directory containing
#      FindCouchbaseGo.cmake, eg. tlm/cmake/Modules
#
# CMAKE_INSTALL_PREFIX - should point to a directory where the bin/ output
#      directory will be placed
#
# GODEPSDIR - should point to a Go workspace directory containing all
#      transitive Go dependencies
#
# In addition, projects that only require the end-user cbq utility may set
# CBQ_ONLY to enable only that target.

INCLUDE (FindCouchbaseGo)

message("SHAAD: HEAVEN")

SET (LDFLAGS)
IF (APPLE)
  # On macOS, we must use -rpath to locate libraries
  # TODO: Remove debug info stripping (-s) added to allow 1.7.6 to work on macOS
  SET (LDFLAGS "-s -extldflags '-Wl,-rpath,@executable_path/../lib'")
ENDIF ()

IF (BUILD_ENTERPRISE)
    SET (_gotags "enterprise")
    SET (NEWEVAL_DIR ../../../../../goproj/src/github.com/couchbase/eventing-ee/evaluator)
    SET(_eval_no_default_path NO_DEFAULT_PATH)
ELSE (BUILD_ENTERPRISE)
    SET (_gotags "community")
ENDIF(BUILD_ENTERPRISE)

SET (GOVERSION SUPPORTED_NEWER)

GET_FILENAME_COMPONENT (CURL_LIBRARY_DIR "${CURL_LIBRARIES}" DIRECTORY)

# STRING(REGEX REPLACE "-[^-]*$" "-N1QL" QUERY_VERSION ${PRODUCT_VERSION})
# SET (LDFLAGS "-X github.com/couchbase/query/util.VERSION=${QUERY_VERSION}")


# GET_FILENAME_COMPONENT (CURL_LIBRARY_DIR "${CURL_LIBRARIES}" DIRECTORY)
SET(CGO_INCLUDE_DIRS "${sigar_SOURCE_DIR}/include;${NEWEVAL_INCLUDE_DIR};${CURL_INCLUDE_DIR}")
SET(CGO_LIBRARY_DIRS "${sigar_BINARY_DIR}/src;${evaluator_BINARY_DIR};${CURL_LIBRARY_DIR}")


IF (BUILD_ENTERPRISE)
find_path(NEWEVAL_INCLUDE_DIR worker.h
    HINTS ${NEWEVAL_DIR}/worker/include)
ENDIF (BUILD_ENTERPRISE)

SET (UDA_DEFAULT_GOTAG "server")

GoModBuild (TARGET uda PACKAGE github.com/moshaad7/uda/cmd/uda
  GOTAGS "${UDA_DEFAULT_GOTAG} ${_gotags}"
  CGO_INCLUDE_DIRS "${CGO_INCLUDE_DIRS}"
  CGO_LIBRARY_DIRS "${CGO_LIBRARY_DIRS}"
  INSTALL_PATH bin OUTPUT uda
  GOVERSION ${GOVERSION})

ADD_DEPENDENCIES(uda sigar n1ql-yacc)
IF (BUILD_ENTERPRISE)
ADD_DEPENDENCIES (uda jseval)
ENDIF ()