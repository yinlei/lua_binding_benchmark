cmake_minimum_required(VERSION 2.8.11)
project(lua_binding_benchmark)

include(cmake/FindLua.cmake)

link_directories(${LUA_LIBRARY_DIRS})
include_directories(${LUA_INCLUDE_DIRS})
include_directories("./")

if(NOT RESULT_OUTPUT_DIR)
	set(RESULT_OUTPUT_DIR result)
endif(NOT RESULT_OUTPUT_DIR)

if(NOT MSVC)
add_definitions("-std=c++1y")
endif(NOT MSVC)

set(BENCHMARK_BINARIES "")
set(LIBRARY_DIRS "")

add_executable(output_csv_title test/output_csv_title.cpp)
target_link_libraries(output_csv_title ${LUA_LIBRARIES})
set(BENCHMARK_BINARIES ${BENCHMARK_BINARIES} output_csv_title)

add_executable(benchmark_plainapi test/plainapi.cpp)
target_link_libraries(benchmark_plainapi ${LUA_LIBRARIES})
set(BENCHMARK_BINARIES ${BENCHMARK_BINARIES} benchmark_plainapi)

add_executable(benchmark_plainapi_without_typesafe test/plainapi.cpp)
target_compile_definitions(benchmark_plainapi_without_typesafe PRIVATE -DBENCHMARK_WITHOUT_TYPESAFE)
target_link_libraries(benchmark_plainapi_without_typesafe ${LUA_LIBRARIES})
set(BENCHMARK_BINARIES ${BENCHMARK_BINARIES} benchmark_plainapi_without_typesafe)


add_executable(benchmark_kaguya test/kaguya.cpp)
target_include_directories(benchmark_kaguya PRIVATE binding_libs/kaguya/include)
target_link_libraries(benchmark_kaguya ${LUA_LIBRARIES})
set(BENCHMARK_BINARIES ${BENCHMARK_BINARIES} benchmark_kaguya)
set(LIBRARY_DIRS ${LIBRARY_DIRS} binding_libs/kaguya/)

add_executable(benchmark_Selene test/selene.cpp)
target_include_directories(benchmark_Selene PRIVATE binding_libs/Selene/include)
target_link_libraries(benchmark_Selene ${LUA_LIBRARIES})
set(BENCHMARK_BINARIES ${BENCHMARK_BINARIES} benchmark_Selene)
set(LIBRARY_DIRS ${LIBRARY_DIRS} binding_libs/Selene/)

add_executable(benchmark_sol test/sol.cpp)
target_include_directories(benchmark_sol PRIVATE binding_libs/sol)
target_link_libraries(benchmark_sol ${LUA_LIBRARIES})
target_compile_definitions(benchmark_sol PRIVATE -DBENCHMARK_WITHOUT_TYPESAFE)
set(BENCHMARK_BINARIES ${BENCHMARK_BINARIES} benchmark_sol)
set(LIBRARY_DIRS ${LIBRARY_DIRS} binding_libs/sol/)

add_executable(benchmark_sol2 test/sol2.cpp)
target_include_directories(benchmark_sol2 PRIVATE binding_libs/sol2)
target_link_libraries(benchmark_sol2 ${LUA_LIBRARIES})
target_compile_definitions(benchmark_sol2 PRIVATE -DBENCHMARK_WITHOUT_TYPESAFE)
set(BENCHMARK_BINARIES ${BENCHMARK_BINARIES} benchmark_sol2)
set(LIBRARY_DIRS ${LIBRARY_DIRS} binding_libs/sol2/)

add_executable(benchmark_luwra test/luwra.cpp)
target_include_directories(benchmark_luwra PRIVATE binding_libs/luwra/lib)
target_link_libraries(benchmark_luwra ${LUA_LIBRARIES})
set(BENCHMARK_BINARIES ${BENCHMARK_BINARIES} benchmark_luwra)

add_executable(benchmark_lua-intf test/lua-intf.cpp)
target_include_directories(benchmark_lua-intf PRIVATE binding_libs/lua-intf)
target_link_libraries(benchmark_lua-intf ${LUA_LIBRARIES})
set(BENCHMARK_BINARIES ${BENCHMARK_BINARIES} benchmark_lua-intf)
set(LIBRARY_DIRS ${LIBRARY_DIRS} binding_libs/lua-intf/)

add_subdirectory(binding_libs/luabind-deboostified/src/)
target_include_directories(luabind PRIVATE binding_libs/luabind-deboostified)

add_executable(benchmark_luabind-deboostified test/luabind-deboostified.cpp)
target_include_directories(benchmark_luabind-deboostified PRIVATE binding_libs/luabind-deboostified)
target_link_libraries(benchmark_luabind-deboostified luabind)
set(BENCHMARK_BINARIES ${BENCHMARK_BINARIES} benchmark_luabind-deboostified)
set(LIBRARY_DIRS ${LIBRARY_DIRS} binding_libs/luabind-deboostified/)


#cannot build with lua5.3.4
option(DISABLE_LUACPPINTERFACE OFF)
# use compat flags to make it build with lua 5.3.4
if(NOT DISABLE_LUACPPINTERFACE)
#luacppinterface's cmakelist is bad
#add_subdirectory(binding_libs/luacppinterface)
add_executable(benchmark_luacppinterface test/luacppinterface.cpp)
target_include_directories(benchmark_luacppinterface PRIVATE binding_libs/luacppinterface/LuaCppInterface)
target_compile_definitions(benchmark_luacppinterface PRIVATE -DLUA_COMPAT_APIINTCASTS -DBENCHMARK_WITHOUT_TYPESAFE)
target_link_libraries(benchmark_luacppinterface ${LUA_LIBRARIES})
set(BENCHMARK_BINARIES ${BENCHMARK_BINARIES} benchmark_luacppinterface)
set(LIBRARY_DIRS ${LIBRARY_DIRS} binding_libs/luacppinterface/)

endif(NOT DISABLE_LUACPPINTERFACE)

file(MAKE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/${RESULT_OUTPUT_DIR})

add_custom_target(execute_benchmark DEPENDS ${BENCHMARK_BINARIES})
foreach(BENCHMARK_BINARY ${BENCHMARK_BINARIES})
add_custom_command(TARGET execute_benchmark POST_BUILD COMMAND ${BENCHMARK_BINARY})
endforeach(BENCHMARK_BINARY)
add_custom_command(TARGET execute_benchmark POST_BUILD
COMMAND ${CMAKE_COMMAND} -E copy result.csv ${CMAKE_CURRENT_SOURCE_DIR}/${RESULT_OUTPUT_DIR}/result.csv)



if(${CMAKE_SYSTEM_NAME} STREQUAL Linux)

add_custom_command(TARGET execute_benchmark POST_BUILD
          COMMAND bash system_info_dump.linux.sh > ${RESULT_OUTPUT_DIR}/system_info.txt
	  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
else()

add_custom_command(TARGET execute_benchmark POST_BUILD
          COMMAND ${CMAKE_COMMAND} -E echo "" > ${RESULT_OUTPUT_DIR}/system_info.txt
	  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
endif()

add_custom_command(TARGET execute_benchmark POST_BUILD
          COMMAND lua library_info_dump.lua ${LIBRARY_DIRS} > ${RESULT_OUTPUT_DIR}/library_info.txt WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
