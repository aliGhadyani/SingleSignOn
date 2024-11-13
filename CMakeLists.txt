cmake_minimum_required(VERSION 3.30)
project(SingleSignOn LANGUAGES CSharp)

# Set the .NET SDK version
set(DOTNET_VERSION "8.0")

# Define the project directory and output
set(BLAZOR_PROJECT_DIR "${CMAKE_SOURCE_DIR}/src")
set(BUILD_OUTPUT_DIR "${CMAKE_BINARY_DIR}/build")

# Specify the target framework (e.g., net8.0)
set(TARGET_FRAMEWORK "net8.0")

# Configure environment variables for .NET SDK
set(ENV{DOTNET_ROOT} "$ENV{HOME}/.dotnet")
set(DOTNET_EXECUTABLE "$ENV{DOTNET_ROOT}/dotnet")

# Verify .NET SDK
execute_process(
    COMMAND ${DOTNET_EXECUTABLE} --version
    OUTPUT_VARIABLE DOTNET_SDK_VERSION
    OUTPUT_STRIP_TRAILING_WHITESPACE
)
message(STATUS "Using .NET SDK Version: ${DOTNET_SDK_VERSION}")

if(NOT DOTNET_SDK_VERSION VERSION_GREATER_EQUAL DOTNET_VERSION)
    message(FATAL_ERROR "Required .NET SDK version ${DOTNET_VERSION} not found.")
endif()

# Set the build configuration (Debug/Release)
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Debug")
endif()
set(BUILD_CONFIGURATION ${CMAKE_BUILD_TYPE})

# C# language settings (for .NET Core/Blazor project)
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# Add C# Blazor Server project
add_custom_target(BlazorServerApp ALL
    COMMAND ${DOTNET_EXECUTABLE} build ${BLAZOR_PROJECT_DIR}
            -c ${BUILD_CONFIGURATION}
            -o ${BUILD_OUTPUT_DIR}
    WORKING_DIRECTORY ${BLAZOR_PROJECT_DIR}
    COMMENT "Building Blazor Server App"
)

# Target for running the Blazor app
add_custom_target(RunBlazorApp
    COMMAND ${DOTNET_EXECUTABLE} run --project ${BLAZOR_PROJECT_DIR} -c ${BUILD_CONFIGURATION}
    WORKING_DIRECTORY ${BLAZOR_PROJECT_DIR}
    COMMENT "Running Blazor Server App"
)

# Clean target
add_custom_target(CleanBlazorApp
    COMMAND ${DOTNET_EXECUTABLE} clean ${BLAZOR_PROJECT_DIR}
    WORKING_DIRECTORY ${BLAZOR_PROJECT_DIR}
    COMMENT "Cleaning Blazor Server App"
)

# Install target
install(
    DIRECTORY ${BUILD_OUTPUT_DIR}/
    DESTINATION bin
)

# Additional settings
set_property(TARGET BlazorServerApp PROPERTY FOLDER "Blazor")
set_property(TARGET RunBlazorApp PROPERTY FOLDER "Blazor")
set_property(TARGET CleanBlazorApp PROPERTY FOLDER "Blazor")
