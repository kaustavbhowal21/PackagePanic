cmake_minimum_required(VERSION 3.14)

# Name your project (matching your repo name)
project(SearchAndDestroy VERSION 1.0.0 LANGUAGES CXX)

# --- Modern C++ Standards ---
# Require C++17 (Standard for libosmium and modern STL features)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF) # Strict adherence to the standard

# --- Compiler Flags (The "Pro-Move") ---
# Turn on strict warnings to catch bugs early, and set optimization to O3 for speed
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wpedantic -O3")

# --- Include Directories ---
# Tells the compiler where to find your .hpp header files
include_directories(${CMAKE_SOURCE_DIR}/include)

# --- System Dependencies ---
# libosmium requires these underlying system libraries to parse PBF files
find_package(Threads REQUIRED)
find_package(ZLIB REQUIRED)
find_package(EXPAT REQUIRED)

# --- Main Executable ---
# Tells CMake to compile your source files into a binary named 'alns_engine'
add_executable(alns_engine 
    src/main.cpp
    # You will add your other .cpp files here later (e.g., src/Vehicle.cpp)
)

# --- Link Libraries ---
# Link the required system libraries to your binary
target_link_libraries(alns_engine PRIVATE 
    Threads::Threads 
    ZLIB::ZLIB 
    EXPAT::EXPAT
)

# --- Testing (Placeholder for Phase 4) ---
# enable_testing()
# add_subdirectory(tests)