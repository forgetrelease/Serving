set(CMAKE_OSX_ARCHITECTURES arm64)
FILE(GLOB paddle_inference_engine_srcs ${CMAKE_CURRENT_LIST_DIR}/src/*.cpp)
add_library(paddle_inference_engine ${paddle_inference_engine_srcs})
target_include_directories(paddle_inference_engine PUBLIC
        ${CMAKE_BINARY_DIR}/Paddle/fluid_install_dir/)
add_dependencies(paddle_inference_engine pdserving extern_paddle configure)
if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
	target_link_libraries(paddle_inference_engine pdserving paddle_inference -lpthread -lm -ldl -lz)
else()
	target_link_libraries(paddle_inference_engine pdserving paddle_inference -lpthread -lcrypto -lm -lrt -lssl -ldl -lz)
endif()

install(TARGETS paddle_inference_engine 
        ARCHIVE DESTINATION ${PADDLE_SERVING_INSTALL_DIR}/lib
        )
