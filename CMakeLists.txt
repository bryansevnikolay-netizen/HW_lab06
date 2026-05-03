cmake_minimum_required(VERSION 3.10)
project(lab03)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(formatter_lib)
add_subdirectory(formatter_ex_lib)
add_subdirectory(solver_lib)
add_subdirectory(solver_application)
add_subdirectory(hello_world_application)

install(TARGETS solver
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

set(CPACK_PACKAGE_NAME "solver")
set(CPACK_PACKAGE_VERSION "1.0.0")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Quadratic equation solver")
set(CPACK_PACKAGE_VENDOR "bryansevnikolay-netizen")
set(CPACK_PACKAGE_CONTACT "bryansev.nikolay@gmail.com")

set(CPACK_GENERATOR "DEB;RPM;DragNDrop;WIX;ZIP;TGZ")

set(CPACK_DEBIAN_PACKAGE_DEPENDS "libc6 (>= 2.28)")

set(CPACK_RPM_PACKAGE_REQUIRES "glibc >= 2.28")

set(CPACK_WIX_UPGRADE_GUID "809da3ca-e1c5-4304-9375-31ef0b5d16d3")

set(CPACK_DMG_VOLUME_NAME "Solver Installer")
set(CPACK_DMG_FORMAT "UDBZ")

set(CPACK_SOURCE_GENERATOR "TGZ;ZIP")
set(CPACK_SOURCE_IGNORE_FILES 
    "/build/"
    "/.git/"
    ".*~$"
    "/.github/"
)

include(CPack)
