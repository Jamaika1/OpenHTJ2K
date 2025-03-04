cmake_minimum_required(VERSION 3.13)

project(open_htj2k LANGUAGES CXX DESCRIPTION "Open source implementation of 15444-1 and 15444-15")
option(BUILD_SHARED_LIBS "Shared Libraries" ON)
enable_language(CXX)

#### Compiler settings
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

message(STATUS "${CMAKE_HOST_SYSTEM_PROCESSOR}")

if (CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
	if (CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "^[xX]86_64$")
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /arch:AVX2 /EHsc /D \"_CRT_SECURE_NO_WARNINGS\"")
	else()
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHsc /D \"_CRT_SECURE_NO_WARNINGS\"")
	endif()
	set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS} /Od /DDEBUG /ZI")
	set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} /Ox")
	set(CMAKE_CXX_FLAGS_RelWithDebInfo "${CMAKE_CXX_FLAGS} /O2 /ZI")
endif()

if (CMAKE_CXX_COMPILER_ID MATCHES "Clang|GNU|Intel")
	if (CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "^[xX]86_64$")
  		set(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} "-Wall -Wformat -march=native -mtune=native")
	else()
		set(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} "-Wall -Wformat")
	endif()
endif()

if (CMAKE_CXX_COMPILER_ID MATCHES "GNU")
	set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS} -O0 -g -Wformat -fsanitize=address,undefined -fsanitize-recover=address -fno-omit-frame-pointer")
	set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -Ofast -flto -fopt-info-vec")
	set(CMAKE_CXX_FLAGS_RelWithDebInfo "${CMAKE_CXX_FLAGS} -O2 -g -fno-omit-frame-pointer -pipe")
endif()

if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
	set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS} -O0 -g -Wformat -fsanitize=address,undefined -fsanitize-recover=address -fno-omit-frame-pointer")
	set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -Ofast -flto -Rpass=loop-vectorize -Rpass-analysis=loop-vectorize")
	set(CMAKE_CXX_FLAGS_RelWithDebInfo "${CMAKE_CXX_FLAGS} -O2 -g -fno-omit-frame-pointer -pipe")
endif()

if (CMAKE_CXX_COMPILER_ID MATCHES "Intel")
	message(STATUS "Intel compiler")
	set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS} -O0 -g -p")
	set(CMAKE_CXX_FLAGS_RelWithDebInfo "${CMAKE_CXX_FLAGS} -O2 -g -parallel")
	set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -Ofast -parallel -qopenmp -ipo -liomp5")
endif()

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/../bin)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/../bin)

#### Source files settings "libopen_htj2k" shared library target
add_library(open_htj2k ${SOURCES})
target_include_directories(open_htj2k PUBLIC source/core/common source/core/codestream source/core/coding source/core/transform source/core/interface)
set_target_properties(open_htj2k PROPERTIES OUTPUT_NAME $<IF:$<CONFIG:Debug>,open_htj2k_d,open_htj2k_R>)

#find_package(OpenMP)
#if(OpenMP_FOUND)
# 	message(STATUS "OpenMP is found.")
#	target_link_libraries(open_htj2k OpenMP::OpenMP_CXX)
#endif()
add_subdirectory(source/core/codestream)
add_subdirectory(source/core/coding)
add_subdirectory(source/core/transform)
add_subdirectory(source/core/interface)
#find_package(OpenCV)
#if (OpenCV_FOUND)
#	message(STATUS "OpenCV is found.")
#	target_compile_definitions(open_htj2k PUBLIC VIEWER)
#	target_include_directories(open_htj2k PUBLIC ${OpenCV_INCLUDE_DIRS})
#	target_link_libraries(open_htj2k ${OpenCV_LIBS})
#endif()

#### decoder target
add_executable(open_htj2k_dec)
target_include_directories(open_htj2k_dec PUBLIC source/core/interface)
add_subdirectory(source/apps/decoder)
set_target_properties(open_htj2k_dec PROPERTIES OUTPUT_NAME $<IF:$<CONFIG:Debug>,open_htj2k_dec_dbg,open_htj2k_dec>)
target_link_libraries(open_htj2k_dec open_htj2k)

#### encoder target
add_executable(open_htj2k_enc)
target_include_directories(open_htj2k_enc PUBLIC source/core/interface)
add_subdirectory(source/apps/encoder)
set_target_properties(open_htj2k_enc PROPERTIES OUTPUT_NAME $<IF:$<CONFIG:Debug>,open_htj2k_enc_dbg,open_htj2k_enc>)
target_link_libraries(open_htj2k_enc open_htj2k)

#### imgcmp target
add_executable(imgcmp)
add_subdirectory(source/apps/imgcmp)
target_include_directories(imgcmp PUBLIC source/apps/imgcmp/)