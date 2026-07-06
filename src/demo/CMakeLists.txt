add_executable(DemoApp)
target_compile_features(DemoApp PRIVATE cxx_std_20)

# Directories
set(SourceDir "${CMAKE_CURRENT_SOURCE_DIR}/src/")
set(IncludeDir "${CMAKE_CURRENT_SOURCE_DIR}/include/")

# Files
set(TestFiles "Main.cpp")
list(TRANSFORM TestFiles PREPEND ${SourceDir})

set(IncludeFiles "Utils.h")
list(TRANSFORM IncludeFiles PREPEND ${IncludeDir})


target_sources(
        DemoApp
        PRIVATE
        ${TestFiles}

        PRIVATE
        FILE_SET HEADERS
        BASE_DIRS ${IncludeDir}
        FILES ${IncludeFiles}
)

target_include_directories(
        DemoApp PRIVATE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
)

target_link_libraries(
        DemoApp PRIVATE
        StrictWarnings
)
