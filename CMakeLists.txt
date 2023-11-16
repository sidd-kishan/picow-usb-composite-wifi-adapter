cmake_minimum_required(VERSION 3.17)

# We prefer to have all linked submodules at toplevel
#set(PICO_TINYUSB_PATH ${CMAKE_CURRENT_SOURCE_DIR}/tinyusb)
set(PICO_BOARD pico_w)
include(pico-sdk/pico_sdk_init.cmake)
project(pico-wifi-adapter)
pico_sdk_init()

set(LWIP_DIR ${CMAKE_CURRENT_SOURCE_DIR}/pico-sdk/lib/lwip)
set (LWIP_INCLUDE_DIRS
    "${LWIP_DIR}/src/include"
    "${CMAKE_CURRENT_SOURCE_DIR}"
)
include(${LWIP_DIR}/src/Filelists.cmake)
include(${PICO_TINYUSB_PATH}/hw/bsp/family_support.cmake)


# gets PROJECT name for the example (e.g. <BOARD>-<DIR_NAME>)
family_get_project_name(PROJECT ${CMAKE_CURRENT_LIST_DIR})

# Extra stuff from TinyUSB, that is not part of tinyusb_device library
set(TINYUSB_LIBNETWORKING_SOURCES
	${PICO_TINYUSB_PATH}/lib/networking
)


set(LWIP "${LWIP_DIR}")
if (NOT EXISTS ${LWIP}/src)
  family_example_missing_dependency(${PROJECT} "lib/lwip")
  return()
endif()

project(${PROJECT} C CXX ASM)

# Checks this example is valid for the family and initializes the project
#family_initialize_project(${PROJECT} ${CMAKE_CURRENT_LIST_DIR})

add_executable(${PROJECT_NAME} main.c msc_disk.c ${PICO_TINYUSB_PATH}/src/class/cdc/cdc_device.c usb_descriptors.c)

pico_enable_stdio_usb(${PROJECT_NAME} 0)
pico_enable_stdio_uart(${PROJECT_NAME} 0)
target_include_directories(${PROJECT_NAME} PRIVATE ${LWIP_INCLUDE_DIRS} ${CMAKE_CURRENT_LIST_DIR}/src ${LWIP}/src/include ${LWIP}/src/include/ipv4 ${LWIP}/src/include/lwip/apps ${TOP}/lib/networking ${PICO_TINYUSB_PATH}/src ${PICO_TINYUSB_PATH}/hw  ${TINYUSB_LIBNETWORKING_SOURCES})


# lib/networking sources
target_sources(${PROJECT} PUBLIC
  ${TINYUSB_LIBNETWORKING_SOURCES}/dhserver.c
  ${TINYUSB_LIBNETWORKING_SOURCES}/dnserver.c
  ${TINYUSB_LIBNETWORKING_SOURCES}/rndis_reports.c
  )

# lwip sources
target_sources(${PROJECT} PUBLIC
  ${LWIP}/src/core/altcp.c
  ${LWIP}/src/core/altcp_alloc.c
  ${LWIP}/src/core/altcp_tcp.c
  ${LWIP}/src/core/def.c
  ${LWIP}/src/core/dns.c
  ${LWIP}/src/core/inet_chksum.c
  ${LWIP}/src/core/init.c
  ${LWIP}/src/core/ip.c
  ${LWIP}/src/core/mem.c
  ${LWIP}/src/core/memp.c
  ${LWIP}/src/core/netif.c
  ${LWIP}/src/core/pbuf.c
  ${LWIP}/src/core/raw.c
  ${LWIP}/src/core/stats.c
  ${LWIP}/src/core/sys.c
  ${LWIP}/src/core/tcp.c
  ${LWIP}/src/core/tcp_in.c
  ${LWIP}/src/core/tcp_out.c
  ${LWIP}/src/core/timeouts.c
  ${LWIP}/src/core/udp.c
  ${LWIP}/src/core/ipv4/autoip.c
  ${LWIP}/src/core/ipv4/dhcp.c
  ${LWIP}/src/core/ipv4/etharp.c
  ${LWIP}/src/core/ipv4/icmp.c
  ${LWIP}/src/core/ipv4/igmp.c
  ${LWIP}/src/core/ipv4/ip4.c
  ${LWIP}/src/core/ipv4/ip4_addr.c
  ${LWIP}/src/core/ipv4/ip4_frag.c
  ${LWIP}/src/netif/ethernet.c
  ${LWIP}/src/netif/slipif.c
  ${LWIP}/src/apps/http/httpd.c
  ${LWIP}/src/apps/http/fs.c
  )

# due to warnings from other net source, we need to prevent error from some of the warnings options
if (CMAKE_C_COMPILER_ID STREQUAL "GNU")
  target_compile_options(${PROJECT} PUBLIC
    -Wno-error=null-dereference
    -Wno-error=conversion
    -Wno-error=sign-conversion
    -Wno-error=sign-compare
    )
elseif (CMAKE_C_COMPILER_ID STREQUAL "IAR")

endif ()


target_link_libraries(${PROJECT_NAME} pico_stdlib pico_unique_id tinyusb_board tinyusb_device pico_cyw43_arch_none pico_multicore)
pico_add_extra_outputs(${PROJECT_NAME})
target_compile_definitions(${PROJECT_NAME} PRIVATE PICO_ENTER_USB_BOOT_ON_EXIT=1 CYW43_SLEEP_CHECK_MS=1)
