cmake_minimum_required(VERSION 3.21)

project(VkRenderer VERSION 1.0)

configure_file(VkRendererConfig.h.in VkRendererConfig.h)

add_executable(VkRenderer main.cpp)


target_compile_definitions(VkRenderer PUBLIC 
        $<$<CONFIG:Debug>:BUILD_DEBUG>
)

if(WIN32)
        target_link_options(VkRenderer PRIVATE 
                $<$<CONFIG:Release>:/ENTRY:mainCRTStartup>
                $<$<CONFIG:Release>:/SUBSYSTEM:WINDOWS>
        )
endif()


target_include_directories(${PROJECT_NAME} PUBLIC "${PROJECT_BINARY_DIR}")

find_package(Vulkan REQUIRED)

set_property(TARGET VkRenderer PROPERTY CXX_STANDARD 17)

#######################################################################
#       MAKE SURE SUBMODULES ARE DOWNLOADED     #######################
#######################################################################

find_package(Git QUIET)
if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
#Update submodules
        option(GIT_SUBMODULE "Check submodules during build" ON)
        if(GIT_SUBMODULE)
                message(STATUS "Submodule update")
                execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive
                                WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                                RESULT_VARIABLE GIT_SUBMOD_RESULT
                                )
                if(NOT GIT_SUBMOD_RESULT EQUAL "0")
                        message(FATAL_ERROR "git submodule update --init failed with ${GIT_SUBMOD_RESULT}")
                endif()
        endif()
endif()

#GLFW
set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)

if(NOT EXISTS "${PROJECT_SOURCE_DIR}/Libraries/glfw/CMakeLists.txt")
        message(FATAL_ERROR "GLFW submodule was not downloaded. GIT_SUBMODULE was turned off or failed.")
else()
        add_subdirectory(Libraries/glfw)
endif()

#glm

include(FetchContent)

FetchContent_Declare(
	glm
	GIT_REPOSITORY	https://github.com/g-truc/glm.git
	GIT_TAG 	bf71a834948186f4097caa076cd2663c69a10e1e #refs/tags/1.0.1
)

FetchContent_MakeAvailable(glm)

include_directories(${Vulkan_INCLUDE_DIRS} Libraries/GLFW/include external)

target_link_directories(VkRenderer PUBLIC ${Vulkan_INCLUDE_DIRS} Libraries/GLFW/include)

message(STATUS ${Vulkan_INCLUDE_DIRS})

target_link_libraries(VkRenderer PUBLIC glm glfw Vulkan::Vulkan)

#Installation
install(TARGETS VkRenderer)
install(DIRECTORY assets DESTINATION ".")
install(FILES shaders/frag.spv shaders/vert.spv DESTINATION shaders)

#Packaging
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set(CPACK_PACKAGE_EXECUTABLES VkRenderer;Renderer)
set(CPACK_PACKAGE_NAME ${PROJECT_NAME})
set(CPACK_PACKAGE_VERSION_MAJOR "${VkRenderer_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${VkRenderer_VERSION_MINOR}")
include(CPack)