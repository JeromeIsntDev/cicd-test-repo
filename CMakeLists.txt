cmake_minimum_required(VERSION 3.16)

project(MyProject VERSION 1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(MyProject 
  src/main.cpp
)

# Packaging with CPack
install(TARGETS MyProject
    RUNTIME DESTINATION bin
)

set(CPACK_PACKAGE_NAME "MyGame")
set(CPACK_PACKAGE_VERSION "1.0.0")
set(CPACK_GENERATOR "TGZ")   # or "ZIP", "DEB", "NSIS" etc.

include(CPack)