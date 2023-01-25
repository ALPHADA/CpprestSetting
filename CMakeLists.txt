cmake_minimum_required(VERSION 3.15)
project(basic)

set(CMAKE_C_STANDARD 17)
set(CMAKE_CXX_STANDARD 17)
if (APPLE)
    set(OPENSSL_ROOT_DIR ${OPENSSL_ROOT_DIR} /usr/local/opt/openssl/)
    set(OPENSSL_CRYPTO_LIBRARY ${OPENSSL_ROOT_DIR}/lib/libcrypto.dylib CACHE FILEPATH "" FORCE)
    set(OPENSSL_SSL_LIBRARY ${OPENSSL_ROOT_DIR}/lib/libssl.dylib CACHE FILEPATH "" FORCE)
    set(OPENSSL_CRYPTO_LIBRARY ${OPENSSL_ROOT_DIR}/lib/libcrypto.dylib CACHE FILEPATH "" FORCE)
endif()

find_package(Boost COMPONENTS regex system filesystem REQUIRED)
find_package(cpprestsdk REQUIRED)
find_package(OpenSSL REQUIRED)
# find_package(libmongocxx REQUIRED)
# find_package(libbsoncxx REQUIRED)

include_directories("/usr/local/lib")

add_definitions(-std=c++17)

add_executable(basic main.cpp)
target_link_libraries(basic PRIVATE 
                            ${Boost_FILESYSTEM_LIBRARY}
                            ${Boost_SYSTEM_LIBRARY}
                            ${Boost_REGEX_LIBRARY}
                            ${OpenSSL_LIBRARIES}
                            ${OPENSSL_CRYPTO_LIBRARY} 
                            cpprestsdk::cpprest)