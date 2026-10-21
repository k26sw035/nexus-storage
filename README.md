# NexusStorage (C++ Cloud Client)

A lightweight, high-performance Modern C++ (C++17) client library designed for seamless integration with AWS S3-compatible cloud storage services.

## Features
- **Modern C++ Standard:** Built using C++17 design patterns, smart pointers, and strict type safety[cite: 9, 10].
- **AWS SigV4 Authentication:** Built-in HMAC-SHA256 request signing mechanism for cloud API security.
- **Clean Architecture:** Modular design separating HTTP networking, authentication, and user-facing storage APIs[cite: 8].
- **Zero Heavy Dependencies:** Optimized build configuration with minimal overhead[cite: 8].

## Build Instructions

### Prerequisites
- C++17 compliant compiler (GCC 8+, Clang 7+, or MSVC 2019+)
- CMake 3.15 or higher

### Building from Source
```bash
mkdir build && cd build
cmake ..
cmake --build .