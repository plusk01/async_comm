cmake_minimum_required(VERSION 3.0.2)
project(async_comm)

if (NOT DEFINED CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif (NOT DEFINED CMAKE_BUILD_TYPE)

find_package(Boost REQUIRED COMPONENTS system)
find_package(Threads)

add_library(${PROJECT_NAME} SHARED
  src/comm.cpp
  src/udp.cpp
  src/serial.cpp
  src/tcp_client.cpp
)
target_include_directories(${PROJECT_NAME} PRIVATE ${Boost_INCLUDE_DIRS})
target_include_directories(${PROJECT_NAME} PUBLIC
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)
target_compile_options(${PROJECT_NAME} PRIVATE -std=c++11)
target_link_libraries(${PROJECT_NAME} ${Boost_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})

# examples
option(ASYNC_COMM_BUILD_EXAMPLES "Build examples" OFF)

if(ASYNC_COMM_BUILD_EXAMPLES)
  add_executable(udp_hello_world examples/udp_hello_world.cpp)
  target_link_libraries(udp_hello_world ${PROJECT_NAME})

  add_executable(serial_loopback examples/serial_loopback.cpp)
  target_link_libraries(serial_loopback ${PROJECT_NAME})

  add_executable(serial_protocol examples/serial_protocol.cpp)
  target_link_libraries(serial_protocol ${PROJECT_NAME})

  add_executable(tcp_client_hello_world examples/tcp_client_hello_world.cpp)
  target_link_libraries(tcp_client_hello_world ${PROJECT_NAME})
endif(ASYNC_COMM_BUILD_EXAMPLES)

install(TARGETS ${PROJECT_NAME}
  EXPORT ${PROJECT_NAME}-targets
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
)
install(DIRECTORY include/${PROJECT_NAME}
  DESTINATION include
  FILES_MATCHING PATTERN "*.h"
)

# install CMake package configuration
install(EXPORT ${PROJECT_NAME}-targets DESTINATION lib/${PROJECT_NAME})
install(FILES ${PROJECT_NAME}-config.cmake DESTINATION lib/${PROJECT_NAME})

# install package.xml for ROS release
install(FILES package.xml DESTINATION share/${PROJECT_NAME})
