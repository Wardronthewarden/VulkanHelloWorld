cmake_minimum_required(VERSION 3.21)

project(VkRenderer)

add_executable(VkRenderer main.cpp)

find_package(Vulkan REQUIRED)

set_property(TARGET VkRenderer PROPERTY CXX_STANDARD 17)
set_property(TARGET VkRenderer PROPERTY RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_CURRENT_BINARY_DIR})
set_property(TARGET VkRenderer PROPERTY RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_CURRENT_BINARY_DIR})

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

if(NOT EXISTS "${PROJECT_SOURCE_DIR}/Libraries/glm/CMakeLists.txt")
        message(FATAL_ERROR "GLM submodule was not downloaded. GIT_SUBMODULE was turned off or failed.")
else()
        add_subdirectory(Libraries/glm)
endif()


include_directories(${Vulkan_INCLUDE_DIRS} Libraries/GLFW/include Libraries/glm external)

target_link_directories(VkRenderer PUBLIC ${Vulkan_INCLUDE_DIRS} Libraries/GLFW/include Libraries/glm)

message(STATUS ${Vulkan_INCLUDE_DIRS})

target_link_libraries(VkRenderer PUBLIC glfw Vulkan::Vulkan)


#Simple solution since this project won't really be scaled much
configure_file(assets/test_tex.png assets/test_tex.png COPYONLY)
configure_file(assets/viking_room.obj assets/viking_room.obj COPYONLY)
configure_file(assets/viking_room.png assets/viking_room.png COPYONLY)
configure_file(shaders/frag.spv shaders/frag.spv COPYONLY)
configure_file(shaders/vert.spv shaders/vert.spv COPYONLY)