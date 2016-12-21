cmake_minimum_required(VERSION 3.4.0)

option(USE_OPENMP "Set to ON to build use openmp"  ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2")

# get NDK ROOT dir from ENV, these should be already exported to system environment
set(ARTOOLKIT_ROOT $ENV{ARTOOLKIT_ROOT})
set(NDK_ROOT $ENV{NDK_ROOT})

include_directories(src/main/jni)
include_directories(${ARTOOLKIT_ROOT}/include)
include_directories(${ARTOOLKIT_ROOT}/include/android)

# add curl
include_directories(${ARTOOLKIT_ROOT}/android/jni/curl/include)
add_library(curl STATIC IMPORTED)
set_target_properties(curl PROPERTIES
    IMPORTED_LOCATION F:/Vision/ARToolKit5-bin-5.3.2-Android/android/jni/curl/libs/${ANDROID_ABI}/libcurl.a)

# add static libraries
set(STATIC_LIBS ar2 kpm util eden argsub_es armulti ar aricp jpeg arvideo)

foreach(LIB ${STATIC_LIBS})
    message("add lib: ${LIB}")
    add_library(${LIB} STATIC IMPORTED)
    set_target_properties(${LIB} PROPERTIES
            IMPORTED_LOCATION ${ARTOOLKIT_ROOT}/android/obj/local/${ANDROID_ABI}/lib${LIB}.a)
endforeach(LIB)

# target lib
set(src_files
    src/main/jni/nftSimple.cpp
    src/main/jni/ARMarkerNFT.c
    src/main/jni/trackingSub.c
    )

add_library( 
             nftSimpleNative
             SHARED
             ${src_files}
             )

# cpufeatures library
add_library(
            cpufeature
            SHARED
            ${NDK_ROOT}/sources/android/cpufeatures/cpu-features.c
            )


find_library( # Sets the name of the path variable.
              local-libs

              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log
              android
              )

target_link_libraries( # Specifies the target library.
                       nftSimpleNative

                       # Links the target library to the log library
                       # included in the NDK.
                       ${local-libs}
                       z
                       dl
                       cpufeature
                       EGL
                       GLESv1_CM
                       ${STATIC_LIBS}
                       curl
                       )
