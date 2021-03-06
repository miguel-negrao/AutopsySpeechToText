cmake_minimum_required (VERSION 3.0.0)
project (autopsy_speech_modules 
    VERSION 0.1.0
    LANGUAGES CXX C)

set (CMAKE_CXX_STANDARD 17)

#On Linux allow using libdeepspeech.so by placing it in same directory as executable

# libdeepspeech.so is not installed system wide on a linux system, and cmake will
# remove the ld link to libdeeepspeech.so used in the build phase.
# libdeepspeech will be copied next to deepspeech_csv on install and in order for
# deepspeech_csv to use the .so file this is needed.
SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_RPATH}:\$ORIGIN")

include(InstallRequiredSystemLibraries)


#PTHREADS
#set(THREADS_PREFER_PTHREAD_FLAG ON)
#find_package(Threads REQUIRED)

#LIBDEEPSPEECH

if(WIN32)
set(USE_CUDA "ON" 
    CACHE FILEPATH "Whether to copy the cuda libraries")

if(USE_CUDA)
    set(CUDA_LIB_DIR "C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v10.1/bin" CACHE FILEPATH "Location of cuda 10.1 dlls on Windows")
endif(USE_CUDA)
endif(WIN32)

set(LIBDEEPSPEECH_PATH "" 
    CACHE FILEPATH "Path to folder containing libdeepspeech.so and deepspeech.h files. It can be be obtained from native_client builds of deepspeech on github.")

add_library(libdeepspeech SHARED IMPORTED)
set_property(TARGET libdeepspeech PROPERTY IMPORTED_LOCATION ${LIBDEEPSPEECH_PATH}/libdeepspeech.so)    
if(WIN32)
    set_property(TARGET libdeepspeech PROPERTY IMPORTED_IMPLIB ${LIBDEEPSPEECH_PATH}/libdeepspeech.so.if.lib)
endif(WIN32)
# needed for deepspeech.h
target_include_directories(libdeepspeech INTERFACE
    ${LIBDEEPSPEECH_PATH}
    )

#EXECUTABLES

#DEEPSPEECH_CSV

set(deepspeech_csv_sources 
    src/deepspeech_csv/main.cc
)
add_executable(deepspeech_csv ${deepspeech_csv_sources})
#target_link_libraries(deepspeech_csv Threads::Threads)
target_link_libraries(deepspeech_csv libdeepspeech)
#files should already be 16kHz mono 16bit PCM Wav
if(UNIX)
    target_compile_definitions(deepspeech_csv PUBLIC NO_SOX)
endif(UNIX) 

#For debugging the CSV segmentation:
#In that case Sox is required, so comment above
#target_compile_definitions(deepspeech_csv PUBLIC CSV_DEBUG)
#target_link_libraries(deepspeech_csv sox)
 

# INSTALL AND PACKAGE
if(WIN32)
#ffmpeg binaries
    set(FFMPEG_PATH "${CMAKE_SOURCE_DIR}/../ffmpeg-win64-static/bin"
        CACHE PATH "Path to the ffmpeg binaries")
    install(FILES ${FFMPEG_PATH}/ffmpeg.exe ${FFMPEG_PATH}/ffprobe.exe DESTINATION speech_modules/bin)
endif()

#autopsy modules python files
install(DIRECTORY autopsy_modules/speech_modules
    DESTINATION .
)

#deepspeech_csv
install(TARGETS deepspeech_csv DESTINATION speech_modules/bin/deepspeech)

#libdeepspeech.so
install(FILES "${LIBDEEPSPEECH_PATH}/libdeepspeech.so" DESTINATION speech_modules/bin/deepspeech)
#cuda libraries
if(WIN32)
    install(FILES "${CUDA_LIB_DIR}/cudart64_101.dll" DESTINATION speech_modules/bin/deepspeech)
    install(FILES "${CUDA_LIB_DIR}/cublasLt64_10.dll" DESTINATION speech_modules/bin/deepspeech)
    install(FILES "${CUDA_LIB_DIR}/cublas64_10.dll" DESTINATION speech_modules/bin/deepspeech)
    install(FILES "${CUDA_LIB_DIR}/cufft64_10.dll" DESTINATION speech_modules/bin/deepspeech)
    install(FILES "${CUDA_LIB_DIR}/curand64_10.dll" DESTINATION speech_modules/bin/deepspeech)
    install(FILES "${CUDA_LIB_DIR}/cusolver64_10.dll" DESTINATION speech_modules/bin/deepspeech)
    install(FILES "${CUDA_LIB_DIR}/cusparse64_10.dll" DESTINATION speech_modules/bin/deepspeech)
    install(FILES "${CUDA_LIB_DIR}/cudnn64_7.dll" DESTINATION speech_modules/bin/deepspeech)
endif(WIN32)

#inaSpeechSegmenter binary
set(INA_SPEECH_SEGMENTER_BUNDLE "${CMAKE_SOURCE_DIR}/out/dist/ina_speech_segmenter"
        CACHE PATH "Path to the inaSpeechSegmenter bundle generated with pyinstaller")
install(DIRECTORY "${INA_SPEECH_SEGMENTER_BUNDLE}/" DESTINATION speech_modules/bin/ina_speech_segmenter)
#cuda libraries
if(WIN32)
    install(FILES "${CUDA_LIB_DIR}/cudart64_101.dll" DESTINATION speech_modules/bin/ina_speech_segmenter)
    install(FILES "${CUDA_LIB_DIR}/cublas64_10.dll" DESTINATION speech_modules/bin/ina_speech_segmenter)
    install(FILES "${CUDA_LIB_DIR}/cublasLt64_10.dll" DESTINATION speech_modules/bin/ina_speech_segmenter)
    install(FILES "${CUDA_LIB_DIR}/cusparse64_10.dll" DESTINATION speech_modules/bin/ina_speech_segmenter)
    install(FILES "${CUDA_LIB_DIR}/cudnn64_7.dll" DESTINATION speech_modules/bin/ina_speech_segmenter)
endif(WIN32)
if (UNIX)
install(FILES
    "${CMAKE_SOURCE_DIR}/out/dist/ina_speech_segmenter/ina_speech_segmenter"
    DESTINATION speech_modules/bin/ina_speech_segmenter
    PERMISSIONS OWNER_EXECUTE OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)
endif()

#CUDA libraries needed by deepspeech_csv will be installed by ina_speech_segmenter.

#deepspeech English model
set(DEEPSPEECH_ENGLISH_MODELS "${CMAKE_SOURCE_DIR}/../deepspeech-0.9.3-models/english" 
    CACHE FILEPATH "Path to the to the engish model folder obtained from Mozilla's github. Currently should be deepspeech-0.9.3-models. Files should be renamed to deepspeech.pbmm and deepspeech.scorer")

install(FILES
    "${DEEPSPEECH_ENGLISH_MODELS}/deepspeech.scorer"
	"${DEEPSPEECH_ENGLISH_MODELS}/deepspeech.pbmm"
	DESTINATION speech_modules/models/english)

#deepspeech Chinese model
set(DEEPSPEECH_CHINESE_MODELS "${CMAKE_SOURCE_DIR}/../deepspeech-0.9.3-models/chinese" 
    CACHE FILEPATH "Path to the to the engish model folder obtained from Mozilla's github. Currently should be deepspeech-0.9.3-models. Files should be renamed to deepspeech.pbmm and deepspeech.scorer")

install(FILES
    "${DEEPSPEECH_CHINESE_MODELS}/deepspeech.scorer"
	"${DEEPSPEECH_CHINESE_MODELS}/deepspeech.pbmm"
	DESTINATION speech_modules/models/chinese)

#utility script
install(FILES
    "${CMAKE_SOURCE_DIR}/python/deepspeech_iss.py"
    DESTINATION speech_modules/bin
    PERMISSIONS OWNER_EXECUTE OWNER_WRITE OWNER_READ GROUP_READ WORLD_READ)

#Install microsoft runtime
if(WIN32)
    INSTALL(FILES ${CMAKE_INSTALL_SYSTEM_RUNTIME_LIBS} DESTINATION speech_modules/bin/deepspeech COMPONENT Libraries)
endif()

set(CPACK_OUTPUT_FILE_PREFIX "")
set(CPACK_GENERATOR "ZIP" CACHE STRING "Generators to support. semi-colon delimited list")
set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY OFF)
include(CPack)
