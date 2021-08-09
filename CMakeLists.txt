cmake_minimum_required (VERSION 3.11)
project(bb-beagle-config VERSION 0.1.0)

option(BEAGLE_CONFIG_SANITIZE "Use sanitize debug options" OFF)
option(ARMHF_DEB "Debian Package for armhf" OFF)

include(FetchContent)
set(FETCHCONTENT_UPDATES_DISCONNECTED TRUE)

FetchContent_Declare(ftxui
  GIT_REPOSITORY https://github.com/ArthurSonzogni/FTXUI
  GIT_TAG fe290e7fed7086abe4fc3b3b21bcc71a07b89d47
)

FetchContent_GetProperties(ftxui)
if(NOT ftxui_POPULATED)
  FetchContent_Populate(ftxui)
  add_subdirectory(${ftxui_SOURCE_DIR} ${ftxui_BINARY_DIR} EXCLUDE_FROM_ALL)
endif()

add_executable(bb-beagle-config 
  src/main.cpp 
  src/ui/focusable.hpp
  src/ui/panel/emmc/emmc_impl.cpp
  src/ui/panel/gpio/gpio_impl.cpp
  src/ui/panel/ics/ics_impl.cpp
  src/ui/panel/led/led_impl.cpp
  src/ui/panel/panel.hpp
  src/ui/panel/placeholder/placeholder_impl.cpp
  src/ui/panel/pru/pru_impl.cpp
  src/ui/panel/wifi/wifi_impl.cpp
  src/ui/ui.cpp
  src/ui/ui.hpp
  src/utils.hpp
  src/utils.cpp
)

target_link_libraries(bb-beagle-config
  PRIVATE ftxui::component
  PRIVATE stdc++fs
)

target_include_directories(bb-beagle-config
  PRIVATE src/
)

set_property(TARGET bb-beagle-config PROPERTY CXX_STANDARD 17)

target_compile_options(bb-beagle-config
  PRIVATE "-Wall"
  PRIVATE "-Wextra"
  PRIVATE "-pedantic"
  PRIVATE "-Werror"
  PRIVATE "-Wdeprecated"
  PRIVATE "-Wshadow"
  PRIVATE "-D_GLIBCXX_USE_CXX17_ABI=0"
)

# Useful for debugging and find 
if (BEAGLE_CONFIG_SANITIZE)
  target_compile_options(bb-beagle-config PRIVATE -fsanitize=address,leak,undefined)
  target_link_libraries(bb-beagle-config PRIVATE -fsanitize=address,leak,undefined)
endif()

INSTALL(TARGETS bb-beagle-config DESTINATION /usr/sbin)

if(ARMHF_DEB)

  set(ARCHITECTURE "armhf")
  set(CPACK_GENERATOR "DEB")
  set(CPACK_SOURCE_GENERATOR "TGZ")
  set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Shreyas Atre")
  set(CPACK_PACKAGE_CONTACT "Shreyas Atre shreyasatre16@gmail.com")
  set(CPACK_PACKAGE_VERSION "${CMAKE_PROJECT_VERSION}")
  set(CPACK_DEBIAN_PACKAGE_NAME "${CMAKE_PROJECT_NAME}")
  set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE ${ARCHITECTURE})
  set(CPACK_PACKAGE_FILE_NAME "${CMAKE_PROJECT_NAME}-${CMAKE_PROJECT_VERSION}-${ARCHITECTURE}")
  set(CPACK_DEBIAN_PACKAGE_DESCRIPTION "Configure your beaglebone device")
  set(CPACK_DEBIAN_PACKAGE_PRIORITY "standard")
  set(CPACK_DEBIAN_PACKAGE_HOMEPAGE "https://github.com/SAtacker/beagle-config")
  set(CPACK_PACKAGING_INSTALL_PREFIX "${CMAKE_INSTALL_PREFIX}")
  set(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_SOURCE_DIR}/README.md")
  set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/LICENSE")
  set(CPACK_RESOURCE_FILE_README "${CMAKE_SOURCE_DIR}/README.md")
  set(CPACK_RESOURCE_FILE_WELCOME "${CMAKE_SOURCE_DIR}/README.md")
  # set(CPACK_SOURCE_IGNORE_FILES "${CMAKE_SOURCE_DIR}/build/;${CMAKE_SOURCE_DIR}/.git/") 
  set(CPACK_DEBIAN_PACKAGE_CONTROL_EXTRA "${CMAKE_SOURCE_DIR}/script/postinst")

  include(CPack)
  
endif(ARMHF_DEB)