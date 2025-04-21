cmake_minimum_required(VERSION 3.20)
project(realtime_chat LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(chat_server src/main.cpp) 