project(dirtsand)
cmake_minimum_required(VERSION 2.6)

if(CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX)
    set(WARN_FLAGS "-Wall -Wextra -Wno-unused-parameter")
    set(CMAKE_C_FLAGS "${WARN_FLAGS} ${CMAKE_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "-std=gnu++0x ${WARN_FLAGS} ${CMAKE_CXX_FLAGS}")
endif()
set(CMAKE_C_FLAGS_DEBUG "-DDEBUG ${CMAKE_C_FLAGS_DEBUG}")
set(CMAKE_CXX_FLAGS_DEBUG "-DDEBUG ${CMAKE_CXX_FLAGS_DEBUG}")

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")
find_package(Threads REQUIRED)
find_package(Postgres REQUIRED)
find_package(OpenSSL REQUIRED)
find_package(Readline REQUIRED)
find_package(ZLIB REQUIRED)

include_directories("${CMAKE_SOURCE_DIR}"
                    "${CMAKE_SOURCE_DIR}/PlasMOUL"
                    "${POSTGRES_INCLUDE_DIR}"
                    "${OPENSSL_INCLUDE_DIR}"
                    "${READLINE_INCLUDE_DIR}"
                    "${ZLIB_INCLUDE_DIR}")

try_compile(HAVE_ATOMIC ${dirtsand_BINARY_DIR}
            ${dirtsand_SOURCE_DIR}/cmake/CheckAtomic.cpp
            OUTPUT_VARIABLE OUTPUT)
if(HAVE_ATOMIC)
    add_definitions(-DHAVE_ATOMIC)
endif()

set(dirtsand_MODULES
    "PlasMOUL"
    "SDL"
    "Types"
    "NetIO"
    "GateKeeper"
    "FileServ"
    "AuthServ"
    "GameServ"
)

foreach(Module_Name ${dirtsand_MODULES})
    file(GLOB_RECURSE ${Module_Name}_SOURCES ${Module_Name}/*.cpp)
    file(GLOB_RECURSE ${Module_Name}_HEADERS ${Module_Name}/*.h)
    source_group("${Module_Name}" FILES ${${Module_Name}_SOURCES} ${${Module_Name}_HEADERS})
    list(APPEND all_SOURCES ${${Module_Name}_SOURCES} ${${Module_Name}_HEADERS})
endforeach(Module_Name)

file(GLOB dirtsand_HEADERS *.h)
file(GLOB dirtsand_SOURCES *.cpp)

set(dirtsand_TEXT
    static_ages.ini
    dirtsand.sample.ini
    README
)

source_group("dirtsand" FILES ${dirtsand_SOURCES} ${dirtsand_HEADERS})
source_group("Text Resources" FILES ${dirtsand_TEXT})

add_executable(dirtsand ${all_SOURCES} ${dirtsand_SOURCES} ${dirtsand_HEADERS} ${dirtsand_TEXT})
target_link_libraries(dirtsand ${CMAKE_THREAD_LIBS_INIT} ${POSTGRES_LIBRARY}
                               ${OPENSSL_CRYPTO_LIBRARIES} ${READLINE_LIBRARY}
                               ${ZLIB_LIBRARY})
set_target_properties(dirtsand PROPERTIES
                      RUNTIME_OUTPUT_DIRECTORY "${dirtsand_SOURCE_DIR}/bin")

install(TARGETS dirtsand
        RUNTIME DESTINATION "bin")
install(PROGRAMS ${CMAKE_SOURCE_DIR}/bin/dsData.sh
        DESTINATION "bin")
install(FILES static_ages.ini
        DESTINATION ".")
