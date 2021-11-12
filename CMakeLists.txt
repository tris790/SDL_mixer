cmake_minimum_required(VERSION 2.8.12)
project(SDL2_MIXER C)

# FIXME: missing CMakeLists.txt for MPG123
set(SUPPORT_MP3_MPG123 OFF CACHE BOOL "" FORCE)

option(SUPPORT_FLAC "Support loading FLAC music with libFLAC" OFF)
option(SUPPORT_OGG "Support loading OGG Vorbis music via Tremor" ON)
option(SUPPORT_MP3_MPG123 "Support loading MP3 music via MPG123" ON)
option(SUPPORT_MOD_MODPLUG "Support loading MOD music via modplug" OFF)
option(SUPPORT_MID_TIMIDITY "Support TiMidity" OFF)

option(SDL_PATH "SDL path" undefined)
include_directories(include src src/codecs ${SDL_PATH}/include)

add_library(SDL2_MIXER)

target_sources(SDL2_MIXER PRIVATE
        src/effect_position.c src/effects_internal.c src/effect_stereoreverse.c
        src/mixer.c src/music.c src/utils.c
        src/codecs/load_aiff.c src/codecs/load_voc.c
        src/codecs/music_cmd.c src/codecs/music_flac.c
        src/codecs/music_fluidsynth.c src/codecs/music_mad.c
        src/codecs/music_mikmod.c src/codecs/music_modplug.c
        src/codecs/music_mpg123.c src/codecs/music_nativemidi.c
        src/codecs/music_ogg.c src/codecs/music_opus.c
        src/codecs/music_timidity.c src/codecs/music_wav.c)

if (SUPPORT_FLAC)
    add_definitions(-DMUSIC_FLAC)
    add_subdirectory(external/flac-1.3.3)
    include_directories(external/flac-1.3.3/include)
    target_link_libraries(SDL2_MIXER PRIVATE FLAC)
endif()

if (SUPPORT_OGG)
    add_definitions(-DMUSIC_OGG -DOGG_USE_TREMOR -DOGG_HEADER=<ivorbisfile.h>)
    add_subdirectory(external/libogg-1.3.2)
    add_subdirectory(external/libvorbisidec-1.2.1)
    include_directories(external/libvorbisidec-1.2.1)
    target_link_libraries(SDL2_MIXER PRIVATE vorbisidec ogg)
endif()

if (SUPPORT_MP3_MPG123)
    add_definitions(-DMUSIC_MP3_MPG123)
    add_subdirectory(external/mpg123-1.25.13)
    target_link_libraries(SDL2_MIXER PRIVATE mpg123)
endif()

if (SUPPORT_MOD_MODPLUG)
    add_definitions(-DMUSIC_MOD_MODPLUG -DMODPLUG_HEADER=<modplug.h>)
    add_subdirectory(external/libmodplug-0.8.9.0)
    include_directories(external/libmodplug-0.8.9.0/src)
    target_link_libraries(SDL2_MIXER PRIVATE modplug)
endif()

if (SUPPORT_MID_TIMIDITY)
    add_definitions(-DMUSIC_MID_TIMIDITY)
    add_subdirectory(timidity)
    target_link_libraries(SDL2_MIXER PRIVATE timidity)
endif()

target_include_directories(SDL2_MIXER PUBLIC include)
target_link_libraries(SDL2_MIXER PRIVATE msvcrt)
target_link_libraries(SDL2_MIXER PRIVATE SDL2)
