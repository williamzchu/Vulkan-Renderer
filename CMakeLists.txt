cmake_minimum_required(VERSION 3.20)
project(VulkanRenderer)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Find all packages from vcpkg.json
find_package(Vulkan REQUIRED)
find_package(glfw3 CONFIG REQUIRED)
find_package(glm CONFIG REQUIRED)
find_package(tinyobjloader CONFIG REQUIRED)
find_package(stb REQUIRED)
find_path(TINYGLTF_INCLUDE_DIR tiny_gltf.h REQUIRED)
find_package(nlohmann_json CONFIG REQUIRED)
find_package(ktx CONFIG REQUIRED)

# Find shader compiler
find_program(GLSLANG_VALIDATOR glslangValidator 
    HINTS $ENV{VULKAN_SDK}/bin 
    REQUIRED
)

# Your main executable
add_executable(VulkanApp
    src/main.cpp
)

target_compile_definitions(VulkanApp PRIVATE
    VULKAN_HPP_NO_STRUCT_CONSTRUCTORS
    VULKAN_HPP_DISPATCH_LOADER_DYNAMIC=1  # Optional, if you need dynamic dispatch
)

# Link all libraries
target_link_libraries(VulkanApp PRIVATE
    Vulkan::Vulkan
    glfw
    glm::glm
    tinyobjloader::tinyobjloader
    nlohmann_json::nlohmann_json
    KTX::ktx
)

# Include STB and TinyGLTF headers
target_include_directories(VulkanApp PRIVATE
    ${stb_INCLUDE_DIRS}
    ${TINYGLTF_INCLUDE_DIR}
)

# Set output directory
set_target_properties(VulkanApp PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
)

# Optional: Add shader compilation function
function(add_shaders TARGET)
    set(SHADER_DIR ${CMAKE_CURRENT_SOURCE_DIR}/shaders)
    set(SHADER_OUTPUT_DIR ${CMAKE_BINARY_DIR}/bin/shaders)
    
    file(MAKE_DIRECTORY ${SHADER_OUTPUT_DIR})
    
    file(GLOB VERT_SHADERS ${SHADER_DIR}/*.vert)
    file(GLOB FRAG_SHADERS ${SHADER_DIR}/*.frag)
    file(GLOB COMP_SHADERS ${SHADER_DIR}/*.comp)
    
    set(ALL_SHADERS ${VERT_SHADERS} ${FRAG_SHADERS} ${COMP_SHADERS})
    
    foreach(SHADER ${ALL_SHADERS})
        get_filename_component(SHADER_NAME ${SHADER} NAME)
        set(SHADER_OUTPUT ${SHADER_OUTPUT_DIR}/${SHADER_NAME}.spv)
        
        add_custom_command(
            OUTPUT ${SHADER_OUTPUT}
            COMMAND ${GLSLANG_VALIDATOR} -V ${SHADER} -o ${SHADER_OUTPUT}
            DEPENDS ${SHADER}
            COMMENT "Compiling shader: ${SHADER_NAME}"
            VERBATIM
        )
        
        list(APPEND SHADER_OUTPUTS ${SHADER_OUTPUT})
    endforeach()
    
    add_custom_target(${TARGET}_Shaders ALL DEPENDS ${SHADER_OUTPUTS})
    add_dependencies(${TARGET} ${TARGET}_Shaders)
endfunction()

# Compile shaders (if you have any in shaders/ folder)
# Uncomment this line when you add shaders
# add_shaders(VulkanApp)

# Optional: Copy assets to build directory
file(GLOB TEXTURES ${CMAKE_CURRENT_SOURCE_DIR}/assets/textures/*)
file(GLOB MODELS ${CMAKE_CURRENT_SOURCE_DIR}/assets/models/*)

if(TEXTURES)
    file(COPY ${TEXTURES} DESTINATION ${CMAKE_BINARY_DIR}/bin/textures)
endif()

if(MODELS)
    file(COPY ${MODELS} DESTINATION ${CMAKE_BINARY_DIR}/bin/models)
endif()

# Windows-specific: Set working directory for Visual Studio
if(WIN32 AND MSVC)
    set_target_properties(VulkanApp PROPERTIES
        VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
    )
endif()