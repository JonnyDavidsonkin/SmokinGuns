cmake_minimum_required(VERSION 3.14)
project(SmokinGuns
  VERSION 1.2
  DESCRIPTION "Smokin' Guns - Western-themed Quake 3 modification (SDL3 port)"
  LANGUAGES C
)

# ============================================================
# Options
# ============================================================
option(BUILD_CLIENT              "Build game client"                    ON)
option(BUILD_SERVER              "Build dedicated server"               ON)
option(BUILD_GAME_SO             "Build game shared libs"               OFF)
option(BUILD_RENDERER_OPENGL2    "Build OpenGL2 renderer"               ON)
option(USE_OPENAL                "Use OpenAL audio"                     ON)
option(USE_OPENAL_DLOPEN         "Dynamically load OpenAL"              ON)
option(USE_CURL                  "Use libcurl"                         ON)
option(USE_CURL_DLOPEN           "Dynamically load libcurl"             ON)
option(USE_CODEC_VORBIS          "Enable Ogg Vorbis"                    ON)
option(USE_CODEC_OPUS            "Enable Opus"                         ON)
option(USE_MUMBLE                "Enable Mumble"                       ON)
option(USE_VOIP                  "Enable VoIP"                         ON)
option(USE_FREETYPE              "Enable FreeType"                     OFF)
option(USE_RENDERER_DLOPEN       "Build renderers as shared libs"       ON)
option(USE_INTERNAL_SPEEX        "Use internal Speex"                  ON)
option(USE_INTERNAL_OGG          "Use internal Ogg"                    ON)
option(USE_INTERNAL_VORBIS       "Use internal Vorbis"                 ON)
option(USE_INTERNAL_OPUS         "Use internal Opus"                   ON)
option(USE_INTERNAL_ZLIB         "Use internal zlib"                   ON)
option(USE_INTERNAL_JPEG         "Use internal JPEG"                   OFF)
option(USE_LOCAL_HEADERS         "Use bundled SDL headers"             OFF)

# ============================================================
# Compiler setup
# ============================================================
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

add_compile_options(
  -Wall -fno-strict-aliasing -Wimplicit -Wstrict-prototypes -pipe -DUSE_ICON
  -Wformat=2 -Wno-format-zero-length -Wformat-security -Wno-format-nonliteral
  -Wstrict-aliasing=2 -Wmissing-format-attribute -Wdisabled-optimization
)

# Version
set(VERSION_CONSTANT "1.2")
set(PRODUCT_RELEASE "dev")

# ============================================================
# SDL3 - Fetch from GitHub using FetchContent
# ============================================================
include(FetchContent)
FetchContent_Declare(
  SDL3
  GIT_REPOSITORY https://github.com/libsdl-org/SDL.git
  GIT_TAG main
  GIT_SHALLOW TRUE
  OVERRIDE_FIND_PACKAGE
)
# We need SDL3 but we'll build with our compat layer
set(SDL_SHARED_ENABLED_BY_DEFAULT OFF CACHE BOOL "" FORCE)
set(SDL_STATIC_ENABLED_BY_DEFAULT ON CACHE BOOL "" FORCE)
set(SDL_TEST_ENABLED_BY_DEFAULT OFF CACHE BOOL "" FORCE)
set(SDL_X11_XCURSOR OFF CACHE BOOL "" FORCE)
set(SDL_X11_XDBE OFF CACHE BOOL "" FORCE)
set(SDL_X11_XINPUT OFF CACHE BOOL "" FORCE)
set(SDL_X11_XRANDR OFF CACHE BOOL "" FORCE)
set(SDL_X11_XSCRNSAVER OFF CACHE BOOL "" FORCE)
set(SDL_X11_XSHAPE OFF CACHE BOOL "" FORCE)
set(SDL_X11_XSYNC OFF CACHE BOOL "" FORCE)
set(SDL_X11_XFIXES OFF CACHE BOOL "" FORCE)
set(SDL_X11_XTEST OFF CACHE BOOL "" FORCE)

FetchContent_MakeAvailable(SDL3)

# ============================================================
# Paths
# ============================================================
set(CDIR "${CMAKE_SOURCE_DIR}/code")

include_directories(
  ${CDIR}/qcommon ${CDIR}/client ${CDIR}/server ${CDIR}/sys ${CDIR}/sdl
  ${CDIR}/renderercommon ${CDIR}/renderergl1 ${CDIR}/renderergl2
  ${CDIR}/botlib ${CDIR}/ui ${CDIR}/null
  ${CDIR}/jpeg-8c ${CDIR}/zlib
  ${CDIR}/libspeex/include ${CDIR}/libogg-1.3.1/include
  ${CDIR}/libvorbis-1.3.4/include ${CDIR}/libvorbis-1.3.4/lib
  ${CDIR}/opus-1.1/include ${CDIR}/opus-1.1/celt ${CDIR}/opus-1.1/silk
  ${CDIR}/opus-1.1/silk/float ${CDIR}/opusfile-0.5/include
)

# ============================================================
# Defines
# ============================================================
add_definitions(
  -DSTANDALONE -DSMOKINGUNS
  -DPRODUCT_VERSION=\"${VERSION_CONSTANT}\"
  -DPRODUCT_RELEASE=\"${PRODUCT_RELEASE}\"
  -DUSE_SDL3_COMPAT
)

if(NOT USE_INTERNAL_ZLIB)
  add_definitions(-DNO_GZIP)
endif()

if(USE_INTERNAL_JPEG)
  add_definitions(-DUSE_INTERNAL_JPEG)
endif()

if(USE_OPENAL)
  add_definitions(-DUSE_OPENAL)
  if(USE_OPENAL_DLOPEN)
    add_definitions(-DUSE_OPENAL_DLOPEN)
  endif()
endif()

if(USE_CURL)
  add_definitions(-DUSE_CURL)
  if(USE_CURL_DLOPEN)
    add_definitions(-DUSE_CURL_DLOPEN)
  endif()
endif()

if(USE_CODEC_VORBIS)
  add_definitions(-DUSE_CODEC_VORBIS)
endif()

if(USE_CODEC_OPUS AND USE_INTERNAL_OPUS)
  add_definitions(-DUSE_CODEC_OPUS -DOPUS_BUILD -DHAVE_LRINTF -DFLOATING_POINT -DUSE_ALLOCA)
endif()

if(USE_MUMBLE)
  add_definitions(-DUSE_MUMBLE)
endif()

if(USE_VOIP)
  add_definitions(-DUSE_VOIP)
endif()

if(USE_RENDERER_DLOPEN)
  add_definitions(-DUSE_RENDERER_DLOPEN)
endif()

if(USE_FREETYPE)
  add_definitions(-DBUILD_FREETYPE)
endif()

# ============================================================
# Find packages
# ============================================================
find_package(OpenGL REQUIRED)
find_package(Threads REQUIRED)

# ============================================================
# Source files (using GLOB for auto-discovery)
# ============================================================

# Common engine sources
file(GLOB COMMON_SOURCES
  ${CDIR}/qcommon/*.c
)

# Client engine sources
file(GLOB CLIENT_SOURCES
  ${CDIR}/client/*.c
  ${CDIR}/server/*.c
  ${CDIR}/sdl/*.c
)

# Botlib
file(GLOB BOTLIB_SOURCES
  ${CDIR}/botlib/*.c
)

# System/platform
file(GLOB SYS_SOURCES
  ${CDIR}/sys/*.c
)

# Renderer OpenGL1
file(GLOB RENDERER1_SOURCES
  ${CDIR}/renderergl1/*.c
)

# Renderer OpenGL2
file(GLOB RENDERER2_SOURCES
  ${CDIR}/renderergl2/*.c
)

# SDL1.2 compat layer
set(SDL12_COMPAT_SOURCES
  ${CDIR}/sdl12_compat.c
)

# Game module sources
file(GLOB GAME_SOURCES ${CDIR}/game/*.c)
file(GLOB CGAME_SOURCES ${CDIR}/cgame/*.c)
file(GLOB UI_SOURCES ${CDIR}/ui/*.c)

# ============================================================
# Build logic
# ============================================================

set(LINUX_LIBS ${CMAKE_DL_LIBS} m)

# ---- Helper function ----
function(setup_renderer target)
  target_compile_options(${target} PRIVATE -fPIC -fvisibility=hidden)
  target_link_libraries(${target} PRIVATE OpenGL::GL SDL3::SDL3 ${CMAKE_THREAD_LIBS_INIT} m ${CMAKE_DL_LIBS})
  if(USE_FREETYPE)
    target_link_libraries(${target} PRIVATE Freetype::Freetype)
  endif()
endfunction()

# ---- Client binary ----
if(BUILD_CLIENT)
  set(ALL_CLIENT_SOURCES
    ${COMMON_SOURCES} ${CLIENT_SOURCES} ${BOTLIB_SOURCES} ${SYS_SOURCES}
    ${SDL12_COMPAT_SOURCES}
  )

  add_executable(smokinguns ${ALL_CLIENT_SOURCES})

  target_include_directories(smokinguns PRIVATE ${CDIR})
  target_compile_definitions(smokinguns PRIVATE)

  # Botlib files need -DBOTLIB
  set_source_files_properties(${BOTLIB_SOURCES} PROPERTIES COMPILE_DEFINITIONS "BOTLIB")

  target_link_libraries(smokinguns PRIVATE
    OpenGL::GL SDL3::SDL3 ${CMAKE_THREAD_LIBS_INIT} ${LINUX_LIBS}
  )

  # ---- OpenGL1 renderer ----
  add_library(renderer_opengl1 SHARED ${RENDERER1_SOURCES})
  setup_renderer(renderer_opengl1)

  # ---- OpenGL2 renderer ----
  if(BUILD_RENDERER_OPENGL2 AND RENDERER2_SOURCES)
    add_library(renderer_opengl2 SHARED ${RENDERER2_SOURCES})
    setup_renderer(renderer_opengl2)
  endif()

  # ---- Game modules ----
  if(BUILD_GAME_SO)
    if(GAME_SOURCES)
      add_library(qagame MODULE ${GAME_SOURCES})
      target_compile_options(qagame PRIVATE -fPIC -DQAGAME)
    endif()
    if(CGAME_SOURCES)
      add_library(cgame MODULE ${CGAME_SOURCES})
      target_compile_options(cgame PRIVATE -fPIC -DCGAME)
    endif()
    if(UI_SOURCES)
      add_library(ui MODULE ${UI_SOURCES})
      target_compile_options(ui PRIVATE -fPIC -DUI)
    endif()
  endif()
endif()

# ---- Dedicated server ----
if(BUILD_SERVER)
  # Dedicated server needs server sources, null system, and console
  set(SERVER_SOURCES
    ${COMMON_SOURCES} ${BOTLIB_SOURCES}
    ${CDIR}/server/sv_bot.c ${CDIR}/server/sv_ccmds.c ${CDIR}/server/sv_client.c
    ${CDIR}/server/sv_game.c ${CDIR}/server/sv_init.c ${CDIR}/server/sv_main.c
    ${CDIR}/server/sv_net_chan.c ${CDIR}/server/sv_snapshot.c ${CDIR}/server/sv_world.c
    ${CDIR}/null/null_client.c ${CDIR}/null/null_input.c ${CDIR}/null/null_snddma.c
    ${CDIR}/sys/con_tty.c ${CDIR}/sys/sys_main.c ${CDIR}/sys/sys_unix.c
    ${CDIR}/sys/sys_local.h
  )

  add_executable(smokinguns_dedicated ${SERVER_SOURCES})

  target_compile_options(smokinguns_dedicated PRIVATE -DDEDICATED)
  target_compile_definitions(smokinguns_dedicated PRIVATE DEDICATED)

  # Dedicated server doesn't need SDL3 or OpenGL
  target_link_libraries(smokinguns_dedicated PRIVATE
    ${CMAKE_THREAD_LIBS_INIT} ${LINUX_LIBS} z
  )
endif()

# ============================================================
# Status
# ============================================================
message(STATUS "=== Smokin' Guns CMake Build (SDL3) ===")
message(STATUS "Version: ${VERSION_CONSTANT}")
message(STATUS "Build: ${CMAKE_SYSTEM_NAME} ${CMAKE_SYSTEM_PROCESSOR}")
message(STATUS "Client: ${BUILD_CLIENT}")
message(STATUS "Server: ${BUILD_SERVER}")
message(STATUS "Game SO: ${BUILD_GAME_SO}")
message(STATUS "SDL3: Fetched from GitHub")
message(STATUS "")