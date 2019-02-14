set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

add_executable(loopback loopback.c)
target_include_directories(loopback PRIVATE ../src)
target_link_libraries(loopback LIBUSBgadget::libusb-gadget Threads::Threads)
