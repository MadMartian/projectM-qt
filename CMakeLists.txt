
project(projectM-qt) # the name of your project
cmake_minimum_required(VERSION 3.22.0)
cmake_policy(SET CMP0005 NEW)

SET(CPACK_BINARY_RPM OFF)
#SET(CPACK_BINARY_Z OFF)
SET(CPACK_SOURCE_TGZ ON)
SET(CPACK_BINARY_TGZ ON)
#SET(CPACK_SOURCE_Z OFF)
SET(CPACK_PACKAGE_VERSION_MAJOR "2")
SET(CPACK_PACKAGE_VERSION_MINOR "0")
SET(CPACK_PACKAGE_VERSION_PATCH "0")
include(CPack)

include_directories( ${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR} )
find_package(PkgConfig REQUIRED)

if (${CMAKE_PROJECT_NAME} MATCHES "PROJECTM_ROOT")
	set(LIBPROJECTM_FOUND true)
else(${CMAKE_PROJECT_NAME} MATCHES "PROJECTM_ROOT")
	pkg_search_module(LIBPROJECTM REQUIRED libprojectM)
endif(${CMAKE_PROJECT_NAME} MATCHES "PROJECTM_ROOT")

OPTION(BUILD_PROJECTM_QT_STATIC "Build a static library of projectM-qt rather than the usual shared library format" OFF)

find_package(Qt5 COMPONENTS Core Widgets OpenGL Xml REQUIRED) # find and setup Qt5 for this project

if (Qt5Core_FOUND AND Qt5Widgets_FOUND AND Qt5OpenGL_FOUND AND Qt5Xml_FOUND)
MESSAGE(STATUS "[projectM-qt] Qt5 detected.")
else (Qt5Core_FOUND AND Qt5Widgets_FOUND AND Qt5OpenGL_FOUND AND Qt5Xml_FOUND)
MESSAGE(FATAL_ERROR "Not all required Qt5 components detected. Please install Qt5 Core, Widgets, OpenGL and Xml before continuing (visit www.trolltech.org for more info)")
endif(Qt5Core_FOUND AND Qt5Widgets_FOUND AND Qt5OpenGL_FOUND AND Qt5Xml_FOUND)
set(QT_USE_OPENGL TRUE)

if (LIBPROJECTM_FOUND)
	MESSAGE(STATUS "[projectM-qt] projectM detected.")
else (LIBPROJECTM_FOUND)
	MESSAGE(FATAL_ERROR "projectM not detected! Please install the projectM module or build from top level projectM source directory.")
endif(LIBPROJECTM_FOUND)


CONFIGURE_FILE("${CMAKE_CURRENT_SOURCE_DIR}/libprojectM-qt.pc.in" "${CMAKE_CURRENT_SOURCE_DIR}/libprojectM-qt.pc" @ONLY)

ADD_DEFINITIONS(-DLINUX -DPROJECTM_PREFIX="${PROJECTM_PREFIX}")
ADD_DEFINITIONS(${QT_DEFINITIONS})
ADD_DEFINITIONS(-DQT_PLUGIN)
ADD_DEFINITIONS(-DQT_NO_DEBUG)
#ADD_DEFINITIONS(-DQT_STATIC)

# the variable "projectM-qt_SRCS" contains all .cpp files of this project
set(projectM-qt_SRCS
    qprojectm_mainwindow.cpp
    qprojectm_mainwindow.hpp
    configfile.hpp configfile.cpp
    qpresetfiledialog.hpp
    qplaylistfiledialog.cpp
    qplaylistfiledialog.hpp
    qplaylistmodel.cpp
    qplaylistmodel.hpp
    qxmlplaylisthandler.hpp
    qprojectmconfigdialog.cpp
    qprojectmconfigdialog.hpp
    qplaylisttableview.hpp
    qpresettextedit.cpp
    qpresettextedit.hpp
    qpreseteditordialog.cpp
    qpreseteditordialog.hpp
    qprojectm.hpp
    qprojectmwidget.hpp
)


set(projectM-qt_MOC_HDRS
    qprojectm_mainwindow.hpp
    qpresetfiledialog.hpp
    qplaylistfiledialog.hpp
    qplaylistmodel.hpp
    qprojectmconfigdialog.hpp
    qplaylisttableview.hpp
    qpresettextedit.hpp
    qpreseteditordialog.hpp
    qprojectm.hpp
    qprojectmwidget.hpp
)

set(projectM-qt_UIS
    qprojectm_mainwindow.ui
    qprojectmconfigdialog.ui
    qpreseteditordialog.ui
)

# Qt resource file
set( projectM-qt_RESOURCES
    application.qrc
)

# compile resources
qt5_add_resources( projectM-qt_RC_SRCS ${projectM-qt_RESOURCES} )

qt5_wrap_ui(projectM-qt_UIS_H ${projectM-qt_UIS})

# After this call, foo_MOC_SRCS = moc_Class1.cxx moc_Class2.cxx moc_Class3.cxx.
qt5_wrap_cpp(projectM-qt_MOC_SRCS ${projectM-qt_MOC_HDRS})

if(COMMAND cmake_policy)
      cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)    

if (BUILD_PROJECTM_QT_STATIC)
ADD_LIBRARY(projectM-qt STATIC ${projectM-qt_SRCS} ${projectM-qt_RC_SRCS} ${projectM-qt_MOC_SRCS} ${projectM-qt_UIS_H})
else(BUILD_PROJECTM_QT_STATIC)
ADD_LIBRARY(projectM-qt SHARED ${projectM-qt_SRCS} ${projectM-qt_RC_SRCS} ${projectM-qt_MOC_SRCS} ${projectM-qt_UIS_H})
endif(BUILD_PROJECTM_QT_STATIC)

SET_TARGET_PROPERTIES(projectM-qt PROPERTIES VERSION 1.10 SOVERSION 1)

if (${CMAKE_PROJECT_NAME} MATCHES "PROJECTM_ROOT")
   set(PROJECTM_INCLUDE ${PROJECTM_ROOT_SOURCE_DIR}/libprojectM)
   set(PROJECTM_LINK ${PROJECTM_ROOT_BINARY_DIR}/libprojectM)
elseif (${CMAKE_PROJECT_NAME} MATCHES ${PROJECT_NAME})
   set(PROJECTM_INCLUDE ${LIBPROJECTM_INCLUDEDIR}/libprojectM)
   set(PROJECTM_LINK ${PROJECTM_LDFLAGS})
endif (${CMAKE_PROJECT_NAME} MATCHES "PROJECTM_ROOT")

LINK_DIRECTORIES(${PROJECTM_LINK})

include_directories(${PROJECTM_INCLUDE} ${Qt5Core_INCLUDE_DIRS} ${Qt5Widgets_INCLUDE_DIRS} ${Qt5Xml_INCLUDE_DIRS} ${Qt5OpenGL_INCLUDE_DIRS} ${CMAKE_CURRENT_LIBRARY_DIR})

# link the "qprojectM" target against the Qt libraries. which libraries exactly, is defined by the "include(${QT_USE_FILE})" line above, which sets up this variable.
target_link_libraries(projectM-qt projectM Qt5::Core Qt5::Widgets Qt5::OpenGL Qt5::Xml)

install(FILES qprojectm_mainwindow.hpp DESTINATION include/libprojectM-qt)
install(TARGETS projectM-qt DESTINATION lib${LIB_SUFFIX})
install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/libprojectM-qt.pc" DESTINATION ${CMAKE_INSTALL_PREFIX}/lib${LIB_SUFFIX}/pkgconfig)
install(FILES "images/icons/prjm16-transparent.svg" DESTINATION	${CMAKE_INSTALL_PREFIX}/share/pixmaps)
