cmake_minimum_required(VERSION 3.16)
project(KisakBlack)

set(BIN_NAME "KisakBlack")

set(CMAKE_CXX_STANDARD 20)

set(SRC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/src")
set(TL_DIR  "${CMAKE_CURRENT_SOURCE_DIR}/tl")
set(DW_DIR  "${CMAKE_CURRENT_SOURCE_DIR}/DemonWare")

include(cmake_files.cmake)
# Checkpoint components stay portable and compile in both PC and NX builds.
# SP guards keep the multiplayer executable's behavior unchanged.
target_sources(KisakBlack PRIVATE
    "${SRC_DIR}/clientscript/cscr_save.cpp"
    "${SRC_DIR}/clientscript/cscr_save_write.cpp"
    "${SRC_DIR}/clientscript/cscr_checkpoint_identity.cpp"
    "${SRC_DIR}/game_mp/checkpoint_archive.cpp"
    "${SRC_DIR}/game_mp/g_checkpoint_sp.cpp"
    "${SRC_DIR}/game_mp/g_checkpoint_records_sp.cpp"
    "${SRC_DIR}/game_mp/g_checkpoint_world_sp.cpp"
    "${SRC_DIR}/game_mp/g_checkpoint_stage_sp.cpp"
    "${SRC_DIR}/game_mp/g_checkpoint_client_sp.cpp"
)

# The frontend's DEC-20 computer terminal (SP) and its Zork interpreter.
target_sources(KisakBlack PRIVATE
    "${SRC_DIR}/dec20/dec20_commands.cpp"
    "${SRC_DIR}/dec20/dec20_terminal.cpp"
    "${SRC_DIR}/dec20/dec20_zmachine.cpp"
    "${SRC_DIR}/dec20/dec20_zork.cpp"
)

target_sources(KisakBlack PRIVATE
    "${SRC_DIR}/live/custom_class_sync.cpp"
    "${SRC_DIR}/openblops_directory/directory_protocol.cpp"
    "${SRC_DIR}/openblops_directory/directory_client.cpp"
    "${SRC_DIR}/openblops_directory/directory_engine.cpp"
    "${SRC_DIR}/openblops_directory/directory_frontend.cpp"
    "${SRC_DIR}/openblops_directory/directory_zombies.cpp"
    "${SRC_DIR}/openblops_directory/directory_update.cpp"
)
# Default bo1_directory endpoint compiled into the game (numeric IPv4:UDP-port).
# Empty in the published source; set it per build tree, e.g.
# -DOPENBLOPS_DIRECTORY_DEFAULT=203.0.113.1:3075, so no address is committed.
set(OPENBLOPS_DIRECTORY_DEFAULT "" CACHE STRING "Default bo1_directory endpoint (numeric IPv4:UDP-port)")
if(OPENBLOPS_DIRECTORY_DEFAULT)
    set_property(SOURCE "${SRC_DIR}/openblops_directory/directory_engine.cpp" APPEND PROPERTY
        COMPILE_DEFINITIONS "BO1_DIRECTORY_DEFAULT=\"${OPENBLOPS_DIRECTORY_DEFAULT}\"")
endif()

include(cmake/direct-connect.cmake)
include(cmake/splitscreen.cmake)

if(NINTENDO_SWITCH)
    include(cmake/switch.cmake)
elseif(WIN32)
    include(cmake/windows.cmake)
else()
    message(FATAL_ERROR "No platform configuration for ${CMAKE_SYSTEM_NAME}")
endif()
