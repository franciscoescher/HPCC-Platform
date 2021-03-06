################################################################################
#    HPCC SYSTEMS software Copyright (C) 2012 HPCC Systems®.
#
#    Licensed under the Apache License, Version 2.0 (the "License");
#    you may not use this file except in compliance with the License.
#    You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
#    Unless required by applicable law or agreed to in writing, software
#    distributed under the License is distributed on an "AS IS" BASIS,
#    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#    See the License for the specific language governing permissions and
#    limitations under the License.
################################################################################

project ( eclwatch ) 

set ( ECLWATCH_BUILD_STRATEGY "" CACHE STRING "Override ECL Watch Build Strategy ('SKIP', 'IF_MISSING')" )
if ( "${ECLWATCH_BUILD_STRATEGY}" STREQUAL "SKIP" )
    message ( "-- ECL Watch:  Skipping..." )
else ()
    message ( "-- ECL Watch:  Packaging Optimized/Built Sources..." )
    execute_process(COMMAND node --version
                    OUTPUT_VARIABLE _VERSION
                    RESULT_VARIABLE _NODE_VERSION_RESULT)
    if (NOT _NODE_VERSION_RESULT)
        string (REPLACE "v" "" NODE_VERSION_STRING "${_VERSION}")
        string (REPLACE "." ";" _VERSION_LIST "${NODE_VERSION_STRING}")
        list (GET _VERSION_LIST 0 NODE_VERSION_MAJOR)
        if (${NODE_VERSION_MAJOR} LESS 8)
            message(FATAL_ERROR "Node version ${NODE_VERSION_MAJOR} is too old, please install NodeJS as per https://github.com/hpcc-systems/HPCC-Platform/wiki/Building-HPCC#prerequisites" )
        endif ()
    else ()
        message(FATAL_ERROR "Unable to locate node/npm, please install NodeJS as per https://github.com/hpcc-systems/HPCC-Platform/wiki/Building-HPCC#prerequisites" )
    endif ()

    set ( ECLWATCH_BUILD_TMP "${CMAKE_CURRENT_BINARY_DIR}/tmp" )
    set ( ECLWATCH_BUILD_DEST "${CMAKE_CURRENT_BINARY_DIR}/build" )
    set ( ECLWATCH_BUILD_OUT "${CMAKE_CURRENT_BINARY_DIR}/eclwatch_build.txt" )

    install ( CODE "
        if ( ( NOT \"${ECLWATCH_BUILD_STRATEGY}\" STREQUAL \"IF_MISSING\" ) OR ( NOT EXISTS \"${ECLWATCH_BUILD_DEST}\" ) )
            message ( \"-- ECL Watch:  Rebuilding Site\" )
            message ( \"---- Build Target: ${ECLWATCH_BUILD_DEST}\" )
            message ( \"---- Output log:   ${ECLWATCH_BUILD_OUT}\" )
            message ( \"---- Creating out of source build environment\" )
            file ( REMOVE_RECURSE \"${ECLWATCH_BUILD_TMP}\" )
            execute_process ( COMMAND \"${CMAKE_COMMAND}\" -E copy_directory \"${CMAKE_CURRENT_SOURCE_DIR}/.\" \"${ECLWATCH_BUILD_TMP}\")
            message ( \"---- Installing third party libraries\" )
            execute_process ( WORKING_DIRECTORY \"${ECLWATCH_BUILD_TMP}\" COMMAND \"npm\" \"install\" OUTPUT_FILE \"${ECLWATCH_BUILD_OUT}\" )
            message ( \"---- Clean previous build\" )
            execute_process ( WORKING_DIRECTORY \"${ECLWATCH_BUILD_TMP}\" COMMAND \"npm\" \"run\" \"clean\" OUTPUT_FILE \"${ECLWATCH_BUILD_OUT}\")
            message ( \"---- Copy resources\" ) 
            execute_process ( WORKING_DIRECTORY \"${ECLWATCH_BUILD_TMP}\" COMMAND \"npm\" \"run\" \"copy-res\" OUTPUT_FILE \"${ECLWATCH_BUILD_OUT}\" )
            message ( \"---- Build website\" )
            execute_process ( WORKING_DIRECTORY \"${ECLWATCH_BUILD_TMP}\" COMMAND \"npm\" \"run\" \"bundle\" OUTPUT_FILE \"${ECLWATCH_BUILD_OUT}\" )
            execute_process ( COMMAND \"${CMAKE_CURRENT_SOURCE_DIR}/../build.sh\" \"${ECLWATCH_BUILD_DEST}\" OUTPUT_FILE \"${ECLWATCH_BUILD_OUT}\" )
            execute_process ( COMMAND \"${CMAKE_COMMAND}\" -E copy_directory \"${ECLWATCH_BUILD_TMP}/build/.\" \"${ECLWATCH_BUILD_DEST}\")
            file ( REMOVE_RECURSE \"${ECLWATCH_BUILD_TMP}\" )
        else ()
            message( \"-- ECL Watch:  Reusing (old) Site\" )
        endif ()
    " COMPONENT Runtime)
    Install ( DIRECTORY "${ECLWATCH_BUILD_DEST}/" DESTINATION componentfiles/files COMPONENT Runtime USE_SOURCE_PERMISSIONS )
endif ()

