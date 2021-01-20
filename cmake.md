find_package()用法

## 1.module模式
  在${CMAKE_MODULE_PATH}下找findXXX.cmake
  
## 2.config模式
  在${CMAKE_PREFIX_PATH}下找xxx-config.cmake
  
  ### config使用mongocxx举例：
  ```
  cmake_minimum_required(VERSION 3.17)
project(test_mongo)

set(CMAKE_CXX_STANDARD 17)
add_executable(test_mongo main.cpp)

list(APPEND CMAKE_PREFIX_PATH /usr/local/Cellar/mongo-cxx-driver/lib/cmake/mongocxx-3.6.2/)
list(APPEND CMAKE_PREFIX_PATH /usr/local/Cellar/mongo-cxx-driver/lib/cmake/bsoncxx-3.6.2)

find_package(mongocxx REQUIRED)

if (mongocxx_FOUND)
    target_link_libraries(test_mongo PRIVATE mongo::mongocxx_shared)
else (mongocxx_FOUND)
    message(FATAL_ERROR "mongocxx library not found")
endif (mongocxx_FOUND)




  ```
