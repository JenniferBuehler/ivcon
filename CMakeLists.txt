cmake_minimum_required(VERSION 2.8.3)
project(ivcon)


# by default, compiling with catkin is
# enabled. It will only be disabled again
# if CATKIN_DEVEL_PREFIX is not set.
# To enforce compiling *without* catkin
# even if CATKIN_DEVEL_PREFIX is defined,
# compile with the flag -COMPILE_WITH_CATKIN=false

if (NOT DEFINED COMPILE_WITH_CATKIN)
    message(STATUS "Compiling ivcon: COMPILE_WITH_CATKIN is not defined, setting to TRUE by default and **compiling with catkin**.")
    set (COMPILE_WITH_CATKIN true)
    set (COMPILE_WITH_CATKIN_DEFAULT true) # flag that compiling with catkin is due to default setting
endif (NOT DEFINED COMPILE_WITH_CATKIN)

if (COMPILE_WITH_CATKIN AND NOT DEFINED COMPILE_WITH_CATKIN_DEFAULT)
    message(STATUS "Compiling ivcon: COMPILE_WITH_CATKIN is true, so **compiling with catkin**.")
endif (COMPILE_WITH_CATKIN AND NOT DEFINED COMPILE_WITH_CATKIN_DEFAULT)

if (NOT COMPILE_WITH_CATKIN)
    message(STATUS "Compiling ivcon: COMPILE_WITH_CATKIN is false, so **compiling with cmake.**")
    set (ENFORCE_CMAKE true)
endif (NOT COMPILE_WITH_CATKIN)

if (COMPILE_WITH_CATKIN AND NOT CATKIN_DEVEL_PREFIX)
    set(MSG_STATUS STATUS)
    if (NOT COMPILE_WITH_CATKIN_DEFAULT)
        set(MSG_STATUS WARNING)
    endif (NOT COMPILE_WITH_CATKIN_DEFAULT)
    message(${MSG_STATUS} "Compiling ivcon: COMPILE_WITH_CATKIN is true, but CATKIN_DEVEL_PREFIX is not defined, so **enforcing compilation with cmake**.")
    set (COMPILE_WITH_CATKIN false)
endif (COMPILE_WITH_CATKIN AND NOT CATKIN_DEVEL_PREFIX)

if (COMPILE_WITH_CATKIN)
    find_package(catkin REQUIRED COMPONENTS roscpp) 
endif (COMPILE_WITH_CATKIN) 

SET(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/CMakeMacros/")
find_package(Qt4 REQUIRED)
find_package(Coin REQUIRED)
find_package(SoQt4 REQUIRED)

if (COMPILE_WITH_CMAKE) 
    ###################################
    ## catkin specific configuration ##
    ###################################
    ## The catkin_package macro generates cmake config files for your package
    ## Declare things to be passed to dependent projects
    ## INCLUDE_DIRS: uncomment this if you package contains header files
    ## LIBRARIES: libraries you create in this project that dependent projects also need
    ## CATKIN_DEPENDS: catkin_packages dependent projects also need
    ## DEPENDS: system dependencies of this project that dependent projects also need
    catkin_package(
      INCLUDE_DIRS include include/ivcon
      LIBRARIES ivcon ivcon_static
      CATKIN_DEPENDS roscpp
      DEPENDS libsoqt4-dev libcoin80-dev libqt4-dev
      # DEPENDS Qt4 Coin SoQt4
    )
endif (COMPILE_WITH_CMAKE) 

###########
## Build ##
###########

ADD_DEFINITIONS(${QT_DEFINITIONS})




## Specify additional locations of header files
## Your package locations should be listed before other locations
# include_directories(include)
include_directories(
  ${catkin_INCLUDE_DIRS}
  include/ivcon
  include
  ${SoQt_INCLUDE_DIR}
  ${Coin_INCLUDE_DIR}
  ${QT_INCLUDE_DIR}
  ${QT_INCLUDE_DIR}/Qt
)

set(IVCON_SRC
    src/asc.cpp  src/dxf.cpp   src/ivconv.cpp  src/mesh.cpp  src/smf.cpp  src/tec.cpp   src/txt.cpp    src/vla.cpp
    src/ase.cpp  src/gmod.cpp  src/iv.cpp     src/obj.cpp   src/stl.cpp  src/tmat.cpp  src/ucd.cpp    src/wrl.cpp
    src/byu.cpp  src/hrc.cpp   src/pov.cpp   src/tds.cpp  src/tri.cpp  src/utils.cpp  src/xgl.cpp
)

# build the shared library:
add_library(ivcon SHARED ${IVCON_SRC})

# build the static library:
add_library(ivcon_static STATIC ${IVCON_SRC})

if ((CMAKE_VERSION VERSION_GREATER 2.8.12) OR (CMAKE_VERSION VERSION_EQUAL 2.8.12))
    target_compile_options(ivcon_static PRIVATE -fPIC)
else ((CMAKE_VERSION VERSION_GREATER 2.8.12) OR (CMAKE_VERSION VERSION_EQUAL 2.8.12))
    # not recommended, but works with catkin:
    set_target_properties(ivcon_static PROPERTIES COMPILE_FLAGS -fPIC)
    # when compiling with catkin (and bloom releae),
    # has error "<command-line>:0:1: error: macro names must be identifiers":
    # target_compile_definitions(ivcon_static PRIVATE -fPIC) 
    # could at some point improve this.
endif ((CMAKE_VERSION VERSION_GREATER 2.8.12) OR (CMAKE_VERSION VERSION_EQUAL 2.8.12))

if (COMPILE_WITH_CMAKE) 
    add_executable(ivcon_node src/main.cpp)
endif (COMPILE_WITH_CMAKE) 

## Add cmake target dependencies of the executable/library
## as an example, message headers may need to be generated before nodes
# add_dependencies(ivcon_node ivcon_generate_messages_cpp)

set(LINK_LIBS
   ${catkin_LIBRARIES}
   ${SOQT_LIBRARY}
   ${Coin_LIBRARIES}
   ${QT_LIBRARIES}
)

## Specify libraries to link a library or executable target against
target_link_libraries(ivcon
   ${LINK_LIBS}
)

target_link_libraries(ivcon_static
   ${LINK_LIBS}
)

if (COMPILE_WITH_CMAKE) 
    target_link_libraries(ivcon_node
       ivcon 
       ${LINK_LIBS}
    )
endif (COMPILE_WITH_CMAKE) 


#############
## Install ##
#############

if (COMPILE_WITH_CMAKE) 
    # all install targets should use catkin DESTINATION variables
    # See http://ros.org/doc/api/catkin/html/adv_user_guide/variables.html
    set ( INCLUDE_DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})
    set ( LIB_DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION})
    set ( BIN_DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})
    set ( SHARE_DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})
else (COMPILE_WITH_CMAKE) 
    set ( INCLUDE_DESTINATION ${CMAKE_INSTALL_PREFIX}/include/${PROJECT_NAME}/)
    set ( LIB_DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/)
    set ( BIN_DESTINATION ${CMAKE_INSTALL_PREFIX}/bin/)
    set ( SHARE_DESTINATION ${CMAKE_INSTALL_PREFIX}/share/)
endif (COMPILE_WITH_CMAKE) 

install(TARGETS
    ivcon 
    EXPORT ivcon-lib-targets
    DESTINATION ${LIB_DESTINATION})

install(EXPORT ivcon-lib-targets
    DESTINATION ${LIB_DESTINATION})

install(TARGETS
    ivcon_static 
    EXPORT ivcon-static-lib-targets
    DESTINATION ${LIB_DESTINATION})

install(EXPORT ivcon-static-lib-targets
    DESTINATION ${LIB_DESTINATION})



# all install targets should use catkin DESTINATION variables
# See http://ros.org/doc/api/catkin/html/adv_user_guide/variables.html

## Mark executable scripts (Python etc.) for installation
## in contrast to setup.py, you can choose the destination
# install(PROGRAMS
#   scripts/my_python_script
#   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

set(INSTALL_LIBS_AND_EXES ivcon ivcon_static)
if (COMPILE_WITH_CMAKE) 
    set(INSTALL_LIBS_AND_EXES ${INSTALL_LIBS_AND_EXES} ivcon_node)
endif (COMPILE_WITH_CMAKE) 

## Mark executables and/or libraries for installation
install(TARGETS ${INSTALL_LIBS_AND_EXES}
   ARCHIVE DESTINATION ${LIB_DESTINATION}
   LIBRARY DESTINATION ${LIB_DESTINATION}
   RUNTIME DESTINATION ${BIN_DESTINATION}
)

## Mark cpp header files for installation
install(DIRECTORY include/${PROJECT_NAME}/
   DESTINATION ${INCLUDE_DESTINATION}
   FILES_MATCHING PATTERN "*.h"
   PATTERN ".git" EXCLUDE
)

## Mark other files for installation (e.g. launch and bag files, etc.)
# install(FILES
#   # myfile1
#   # myfile2
#   DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
# )

#############
## Testing ##
#############

## Add gtest based cpp test target and link libraries
# catkin_add_gtest(${PROJECT_NAME}-test test/test_ivcon.cpp)
# if(TARGET ${PROJECT_NAME}-test)
#   target_link_libraries(${PROJECT_NAME}-test ${PROJECT_NAME})
# endif()

## Add folders to be run by python nosetests
# catkin_add_nosetests(test)
