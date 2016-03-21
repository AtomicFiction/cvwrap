set(SOURCE_FILES
    "pluginMain.cpp"
    "cvWrapCmd.cpp"
    "cvWrapCmd.h"
    "cvWrapDeformer.cpp"
    "cvWrapDeformer.h"
    "bindingio.cpp"
    "bindingio.h"
    "common.cpp"
    "common.h"
    "cvwrap.cl"
)

include(CheckCSourceRuns)
if (CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_CLANG )
    set(CMAKE_REQUIRED_FLAGS "-mavx")
    check_c_source_runs("
        #include <immintrin.h>
        int main()
        {
            __m256i a = _mm256_setzero_si256();
            return 0;
        }"        
        HAVE_AVX)
endif()

if (HAVE_AVX)
    message(STATUS "AVX CPU feature found")
    if (WIN32)
	message(STATUS "Enabling AVX")
        set(COMPILE_FLAGS "/arch:AVX")
    else()
	message(STATUS "Enabling AVX")
        set(COMPILE_FLAGS "-mavx")
    endif()
endif()

find_package(Maya REQUIRED)
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${MAYA_CXX_FLAGS} ${COMPILE_FLAGS}")

include_directories(${MAYA_INCLUDE_DIR})
link_directories(${MAYA_LIBRARY_DIR})
add_library(${PROJECT_NAME} SHARED ${SOURCE_FILES})
target_link_libraries(${PROJECT_NAME} ${MAYA_LIBRARIES})

MAYA_PLUGIN(${PROJECT_NAME})
install(TARGETS ${PROJECT_NAME} ${MAYA_TARGET_TYPE} DESTINATION plug-ins/${MAYA_VERSION})
install(FILES "cvwrap.cl" DESTINATION plug-ins/${MAYA_VERSION})

