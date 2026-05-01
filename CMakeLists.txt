cmake_minimum_required(VERSION 3.25)
project(GenericProject LANGUAGES CXX)
enable_testing()

add_library(ProjectStrictWarnings INTERFACE) # Used for linking Warning flags
add_library(ProjectASan INTERFACE) # Used for linking ASan

set(DevToolsDir "Dev Tools")

# Clang Formatter
find_program(CLANG_FORMATTER NAMES clang-format)

if(CLANG_FORMATTER)
    file(GLOB_RECURSE ALL_SOURCE_FILES *.cpp *.h *.cxx *.hxx *.hpp *.cc *.hh *.tpp *.inl)

    add_custom_target(format
        COMMAND ${CLANG_FORMATTER} -i -style=file ${ALL_SOURCE_FILES}
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
        COMMENT "Formatting source files..."
    )
    set_target_properties(format PROPERTIES FOLDER ${DevToolsDir})
else()
    message(WARNING "clang-format not found")
endif()

option(ENABLE_STRICT "Enable Strict Warnings" OFF) # Option to enable compiler specific "Warning as Error"
option(ASAN "Enable ASan" OFF) # Option to enable ASAN

# Doxygen
find_package(Doxygen)
if (DOXYGEN_FOUND)
    message(STATUS "Doxygen found!")
    doxygen_add_docs(
        docs
        ${CMAKE_CURRENT_SOURCE_DIR}
        COMMENT "Generating documentation with doxygen..."
        CONFIG_FILE ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile
        ALL
    )
    set_target_properties(docs PROPERTIES FOLDER ${DevToolsDir})
else()
    message(WARN "Doxygen not found! Skipping documentation generation.")
endif()


# Turn on solution folders
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# Enables Strict mode for all the projects globally.
if(ENABLE_STRICT)
    message(STATUS "Turning on Strict Mode")
    if(MSVC)
        target_compile_options(ProjectStrictWarnings INTERFACE $<$<COMPILE_LANGUAGE:CXX>:/WX /W4 /permissive- /fp:strict>)
    else()
        target_compile_options(ProjectStrictWarnings INTERFACE $<$<COMPILE_LANGUAGE:CXX>:-fno-omit-frame-pointer -Wall -Wextra -Werror -pedantic-errors>)
    endif()
endif()


# Enables Address Sanitizer for all the projects globally.
if(ASAN)
    message(STATUS "Turning on Address Sanitizer")
    if(MSVC)
        add_compile_options(/fsanitize=address)
    else()
        add_compile_options(-fsanitize=address -fno-omit-frame-pointer)
        add_link_options(-fsanitize=address)
    endif()
endif()

include(dependencies.cmake)

add_subdirectory(test-suite)
add_subdirectory(benchmark-suite)

# Link ProjectStrictWarnings to enable strict mode.
target_link_libraries(TestSuite PRIVATE ProjectStrictWarnings)
target_link_libraries(BenchmarkSuite PRIVATE ProjectStrictWarnings)


set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT TestSuite)