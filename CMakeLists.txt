cmake_minimum_required(VERSION 3.12)
add_definitions(-DUNICODE -D_UNICODE)
set (CMAKE_CXX_STANDARD 17)

project(al_khaser VERSION 1.0.0 LANGUAGES CXX ASM_MASM)

FILE(GLOB_RECURSE CPP_SOURCES RELATIVE "/al-khaser" "*.cpp")
FILE(GLOB_RECURSE H_SOURCES RELATIVE "/al-khaser" "*.h")
# !!! no 32-bit support for now
FILE(GLOB_RECURSE ASM_SOURCES RELATIVE "/al-khaser" "*_x64.asm")

add_library(al_khaser ${CPP_SOURCES} ${ASM_SOURCES})

if (BUILD_SHARED_LIBS AND MSVC)
	target_compile_definitions(al_khaser PRIVATE MYLIB_EXPORTS)
endif()


# Add include directories
target_include_directories(al_khaser PUBLIC 
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>   # for headers when building
	$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>  # for client in install mode
)
# Install the library and its headers
install(TARGETS al_khaser
		EXPORT al_khaser_targets
		ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
		LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
		RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}")

# keep the directory structure
foreach (H_FILE ${H_SOURCES})
	file(RELATIVE_PATH REL_PATH ${CMAKE_CURRENT_SOURCE_DIR} ${H_FILE})
	get_filename_component(DIR ${REL_PATH} DIRECTORY)
	install(FILES ${H_FILE} DESTINATION include/${DIR})
endforeach()

# Generate and install *-targets.cmake 
install(EXPORT al_khaser_targets
		FILE al_khaser-targets.cmake
		NAMESPACE al_khaser::
		DESTINATION share/al_khaser)

# Generate the config file in the current binary dir (this ensures it's not placed directly in source)
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/al_khaser-config.cmake"
"include(CMakeFindDependencyMacro)\n"
"include(\"\${CMAKE_CURRENT_LIST_DIR}/al_khaser-targets.cmake\")\n"
)

# Install the generated config file
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/al_khaser-config.cmake"
		DESTINATION share/al_khaser)