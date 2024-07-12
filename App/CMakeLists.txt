cmake_minimum_required(VERSION 3.24)
project(App)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/dist)

set(
        APP_SOURCES

        src/main.cpp

        src/app.cpp

        src/scripts/TestRenderer.cpp
        src/scripts/TrackFramerate.cpp
        src/scripts/ImGuiDemoWindow.cpp
        src/scripts/CameraController.cpp
        src/scripts/Rotator.cpp
        src/scripts/PBRColorRandomizer.cpp
        src/scripts/FullscreenTexture.cpp

        src/utility/utility.cpp
)

if (WIN32 AND NOT EMSCRIPTEN)
    list(APPEND APP_SOURCES platform/windows/resources.rc)
endif ()

add_executable(App ${APP_SOURCES})

if (EMSCRIPTEN)
    if (CMAKE_BUILD_TYPE STREQUAL "Debug")
        target_link_options(
                App PRIVATE
                -sNO_DISABLE_EXCEPTION_CATCHING
        )
    endif ()

    # this is only needed if you want it to relink if emscriptenUtility.js changes, but no other cpp files change
    set_target_properties(App PROPERTIES LINK_DEPENDS ${CMAKE_SOURCE_DIR}/ge-engine/GameEngine/src/GameEngine/Utility/emscriptenUtility.js)

    if (DEBUG_FASTER)
        target_compile_options(
                App PRIVATE
                -O2
        )
        target_link_options(
                App PRIVATE
                -O2
        )
        target_compile_options(
                GameEngine PRIVATE
                -O2
        )
        target_link_options(
                GameEngine PRIVATE
                -O2
        )
    endif ()
endif ()

target_precompile_headers(App REUSE_FROM GameEngine)

target_link_libraries(App PRIVATE GameEngine)

target_include_directories(
        App PUBLIC
        ../ge-engine/GameEngine/src
)

function(copy_file_relative NEW_TARGET_NAME DIR PATH TARGET)
    set(SOURCE ${CMAKE_SOURCE_DIR}/${DIR}/${PATH})
    set(DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/dist/${PATH})

    add_custom_command(
            OUTPUT ${DESTINATION}
            COMMAND ${CMAKE_COMMAND} -E copy ${SOURCE} ${DESTINATION}
            DEPENDS ${SOURCE}
            COMMENT "Copying ${NEW_TARGET_NAME}..."
    )

    add_custom_target(${NEW_TARGET_NAME} ALL DEPENDS ${DESTINATION})
    add_dependencies(${TARGET} ${NEW_TARGET_NAME})
endfunction()

if (EMSCRIPTEN)
    copy_file_relative(index_html App/platform/emscripten index.html App)
    copy_file_relative(favicon_ico App/platform/emscripten favicon.ico App)
    copy_file_relative(parse_hdr_worker_js ge-engine/GameEngine/src/GameEngine/Utility parseHDRWorker.js GameEngine)
endif ()
