if(NOT CONFIG_ZENOH_PICO)
    return()
endif()

zephyr_include_directories(../include)
zephyr_library()

zephyr_compile_definitions(ZENOH_ZEPHYR)
zephyr_compile_definitions(ZENOH_DEBUG=0)

file(READ ${CMAKE_CURRENT_SOURCE_DIR}/../version.txt version)
set(ZENOH_PICO ${PROJECT_VERSION})
set(ZENOH_PICO_MAJOR ${PROJECT_VERSION_MAJOR})
set(ZENOH_PICO_MINOR ${PROJECT_VERSION_MINOR})
set(ZENOH_PICO_PATCH ${PROJECT_VERSION_PATCH})
set(ZENOH_PICO_TWEAK ${PROJECT_VERSION_TWEAK})

configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/../include/zenoh-pico.h.in
  ${CMAKE_CURRENT_SOURCE_DIR}/../include/zenoh-pico.h
  @ONLY
)

configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/../library.json.in
  ${CMAKE_CURRENT_SOURCE_DIR}/../library.json
  @ONLY
)

# Zenoh pico feature configuration options
set(FRAG_MAX_SIZE 4096 CACHE STRING "Use this to override the maximum size for fragmented messages")
set(BATCH_UNICAST_SIZE 2048 CACHE STRING "Use this to override the maximum unicast batch size")
set(BATCH_MULTICAST_SIZE 2048 CACHE STRING "Use this to override the maximum multicast batch size")
set(Z_CONFIG_SOCKET_TIMEOUT 100 CACHE STRING "Default socket timeout in milliseconds")
set(Z_TRANSPORT_LEASE 10000 CACHE STRING "Link lease duration in milliseconds to announce to other zenoh nodes")

# Helper function: Set CMake variable to 1 if Kconfig symbol is "y", else 0
function(config_to_cmake varname kconfigsym)
    if(DEFINED CACHE{${kconfigsym}})
        set(val "$CACHE{${kconfigsym}}")
    elseif(DEFINED ${kconfigsym})
        set(val "${${kconfigsym}}")
    else()
        set(val "")
    endif()
    if(val STREQUAL "y")
        set(${varname} 1 PARENT_SCOPE)
    else()
        set(${varname} 0 PARENT_SCOPE)
    endif()
endfunction()

config_to_cmake(Z_FEATURE_UNSTABLE_API         CONFIG_ZENOH_PICO_UNSTABLE_API)
config_to_cmake(Z_FEATURE_PUBLICATION          CONFIG_ZENOH_PICO_PUBLICATION)
config_to_cmake(Z_FEATURE_SUBSCRIPTION         CONFIG_ZENOH_PICO_SUBSCRIPTION)
config_to_cmake(Z_FEATURE_QUERY                CONFIG_ZENOH_PICO_QUERY)
config_to_cmake(Z_FEATURE_QUERYABLE            CONFIG_ZENOH_PICO_QUERYABLE)
config_to_cmake(Z_FEATURE_LIVELINESS           CONFIG_ZENOH_PICO_LIVELINESS)
config_to_cmake(Z_FEATURE_INTEREST             CONFIG_ZENOH_PICO_INTEREST)
config_to_cmake(Z_FEATURE_FRAGMENTATION        CONFIG_ZENOH_PICO_FRAGMENTATION)
config_to_cmake(Z_FEATURE_ENCODING_VALUES      CONFIG_ZENOH_PICO_ENCODING_VALUES)
config_to_cmake(Z_FEATURE_MULTI_THREAD         CONFIG_ZENOH_PICO_MULTI_THREAD)

config_to_cmake(Z_FEATURE_LINK_TCP                CONFIG_ZENOH_PICO_LINK_TCP)
config_to_cmake(Z_FEATURE_LINK_BLUETOOTH          CONFIG_ZENOH_PICO_LINK_BLUETOOTH)
config_to_cmake(Z_FEATURE_LINK_WS                 CONFIG_ZENOH_PICO_LINK_WS)
config_to_cmake(Z_FEATURE_LINK_SERIAL             CONFIG_ZENOH_PICO_LINK_SERIAL)
config_to_cmake(Z_FEATURE_LINK_SERIAL_USB         CONFIG_ZENOH_PICO_LINK_SERIAL_USB)
config_to_cmake(Z_FEATURE_SCOUTING_UDP            CONFIG_ZENOH_PICO_SCOUTING_UDP)
config_to_cmake(Z_FEATURE_LINK_UDP_UNICAST        CONFIG_ZENOH_PICO_LINK_UDP_UNICAST)
config_to_cmake(Z_FEATURE_LINK_UDP_MULTICAST      CONFIG_ZENOH_PICO_LINK_UDP_MULTICAST)
config_to_cmake(Z_FEATURE_UNICAST_TRANSPORT       CONFIG_ZENOH_PICO_UNICAST_TRANSPORT)
config_to_cmake(Z_FEATURE_MULTICAST_TRANSPORT     CONFIG_ZENOH_PICO_MUTLICAST_TRANSPORT)
config_to_cmake(Z_FEATURE_RAWETH_TRANSPORT        CONFIG_ZENOH_PICO_RAWETH_TRANSPORT)
config_to_cmake(Z_FEATURE_TCP_NODELAY             CONFIG_ZENOH_PICO_TCP_NODELAY)
config_to_cmake(Z_FEATURE_LOCAL_SUBSCRIBER        CONFIG_ZENOH_PICO_LOCAL_SUBSCRIBER)
config_to_cmake(Z_FEATURE_SESSION_CHECK           CONFIG_ZENOH_PICO_SESSION_CHECK)
config_to_cmake(Z_FEATURE_BATCHING                CONFIG_ZENOH_PICO_BATCHING)
config_to_cmake(Z_FEATURE_BATCH_TX_MUTEX          CONFIG_ZENOH_PICO_BATCH_TX_MUTEX)
config_to_cmake(Z_FEATURE_BATCH_PEER_MUTEX        CONFIG_ZENOH_PICO_BATCH_PEER_MUTEX)
config_to_cmake(Z_FEATURE_MATCHING                CONFIG_ZENOH_PICO_MATCHING)
config_to_cmake(Z_FEATURE_RX_CACHE                CONFIG_ZENOH_PICO_RX_CACHE)
config_to_cmake(Z_FEATURE_UNICAST_PEER            CONFIG_ZENOH_PICO_UNICAST_PEER_MODE)
config_to_cmake(Z_FEATURE_AUTO_RECONNECT          CONFIG_ZENOH_PICO_AUTO_RECONNECT)
config_to_cmake(Z_FEATURE_MULTICAST_DECLARATIONS  CONFIG_ZENOH_PICO_MULTICAST_DECLARATIONS)

configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/../include/zenoh-pico/config.h.in
  ${CMAKE_CURRENT_SOURCE_DIR}/../include/zenoh-pico/config.h
  @ONLY
)

file(GLOB_RECURSE Sources
  "../src/api/*.c"
  "../src/collections/*.c"
  "../src/link/*.c"
  "../src/net/*.c"
  "../src/protocol/*.c"
  "../src/session/*.c"
  "../src/transport/*.c"
  "../src/utils/*.c"
  "../src/system/common/*.c"
)

file(GLOB Sources_Zephyr "../src/system/zephyr/*.c")
list(APPEND Sources ${Sources_Zephyr})

zephyr_library_sources(${Sources})
