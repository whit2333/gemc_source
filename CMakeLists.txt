cmake_minimum_required(VERSION 3.8 FATAL_ERROR)
project(GEMC VERSION 0.0.1)

include(GNUInstallDirs)

#----------------------------------------------------------------------------
# Use our modules and utility functions
set(CMAKE_MODULE_PATH
  ${CMAKE_INSTALL_PREFIX}
  ${PROJECT_SOURCE_DIR}/cmake/Modules
  ${CMAKE_MODULE_PATH}
  )

SET(GEMC_MAJOR_VERSION 0)
SET(GEMC_MINOR_VERSION 0)
SET(GEMC_PATCH_VERSION 0)
set(GEMC_VERSION ${GEMC_MAJOR_VERSION}.${GEMC_MINOR_VERSION}.${GEMC_PATCH_VERSION})

SET(SHARE_DIR_NAME bubble_sim)
SET(MACRO_DIR    share/${SHARE_DIR_NAME})
SET(GEOMETRY_DIR share/${SHARE_DIR_NAME}/geometry_files)

SET(GEMC_MACRO_DIR ${CMAKE_INSTALL_PREFIX}/${MACRO_DIR} )
SET(GEMC_GEOMETRY_DIR ${CMAKE_INSTALL_PREFIX}/${GEOMETRY_DIR} )



#----------------------------------------------------------------------------
# Find Geant4 package, activating all available UI and Vis drivers by default
# You can set WITH_GEANT4_UIVIS to OFF via the command line or ccmake/cmake-gui
# to build a batch mode only executable
#
option(WITH_GEANT4_UIVIS "Build example with Geant4 UI and Vis drivers" ON)
if(WITH_GEANT4_UIVIS)
  find_package(Geant4 REQUIRED ui_all vis_all)
else()
  find_package(Geant4 REQUIRED)
endif()

# Find includes in corresponding build directories
set(CMAKE_INCLUDE_CURRENT_DIR ON)
# Instruct CMake to run moc automatically when needed
set(CMAKE_AUTOMOC ON)
# Create code from a list of Qt designer ui files
set(CMAKE_AUTOUIC ON)

# Find the QtWidgets library
find_package(Qt5Widgets  CONFIG REQUIRED)
find_package(Qt5Xml  CONFIG REQUIRED)
find_package(Qt5Sql  CONFIG REQUIRED)



#----------------------------------------------------------------------------
# ROOT
list(APPEND CMAKE_PREFIX_PATH $ENV{ROOTSYS})
find_package(ROOT REQUIRED COMPONENTS MathCore RIO Hist MathMore Graf Graf3d
   Gpad GX11 GeomPainter X3d Gviz3d RGL
   Eve EG Foam GenVector Geom GLEW Gui HistPainter MathCore Matrix MLP Net
   Physics Spectrum Thread Tree TreeViewer)
#find_package(ROOT REQUIRED COMPONENTS MathCore RIO Hist MathMore Graf HistPainter Rint)
include(${ROOT_USE_FILE})
include_directories(${CMAKE_SOURCE_DIR} ${ROOT_INCLUDE_DIRS})
add_definitions(${ROOT_CXX_FLAGS})

find_package(EVIO REQUIRED)

get_target_property(EVIO_INCLUDE_DIRS EVIO::evioxx INTERFACE_INCLUDE_DIRECTORIES)

# CADMesh
find_package(cadmesh REQUIRED)
#get_target_property(cadmesh_INCLUDE_DIRS cadmesh INTERFACE_INCLUDE_DIRECTORIES)
if(cadmesh_FOUND)
endif(cadmesh_FOUND)

find_package(MLibrary REQUIRED)
get_target_property(MLibrary_INCLUDE_DIRS MLibrary::MLibrary INTERFACE_INCLUDE_DIRECTORIES)

#----------------------------------------------------------------------------
# Setup Geant4 include directories and compile definitions
# Setup include directory for this project
#
include(${Geant4_USE_FILE})
include_directories(
  ${PROJECT_SOURCE_DIR}/include
  ${PROJECT_BINARY_DIR}
  )


#----------------------------------------------------------------------------
# Locate sources and headers for this project
# NB: headers are included so they will show up in IDEs
#
# Materials
set(materials_sources 
  materials/material_factory.cc
  materials/cpp_materials.cc
  materials/mysql_materials.cc
  materials/text_materials.cc)


set(mirrors_sources
  mirrors/mirrors_factory.cc
  mirrors/mysql_mirrors.cc
  mirrors/text_mirrors.cc)


set(parameters_sources
  parameters/parameter_factory.cc
  parameters/mysql_parameters.cc
  parameters/text_parameters.cc)

set(util_sources
  utilities/string_utilities.cc
  utilities/utils.cc
  utilities/lStdHep.cc
  utilities/lXDR.cc
  utilities/options.cc)

set(det_sources
  detector/detector.cc
  detector/detector_factory.cc
  detector/mysql_det_factory.cc
  detector/gdml_det_factory.cc
  detector/cad_det_factory.cc
  detector/clara_det_factory.cc
  detector/text_det_factory.cc)

set(sensi_sources
  sensitivity/sensitiveDetector.cc
  sensitivity/identifier.cc
  sensitivity/Hit.cc
  sensitivity/backgroundHits.cc
  sensitivity/HitProcess.cc
  sensitivity/sensitiveID.cc)


set(phys_sources
  physics/PhysicsList.cc
  physics/PhysicsListMessenger.cc)

set(field_sources
  fields/field.cc
  fields/fieldFactory.cc
  fields/asciiField.cc
  fields/mappedField.cc
  fields/multipoleField.cc
  fields/symmetries/dipole.cc
  fields/symmetries/cylindrical.cc
  fields/symmetries/phi-segmented.cc
  fields/symmetries/cartesian_3d.cc)
# Hit Processes
#env.Append(CPPPATH = ['hitprocess', 'hitprocess/clas12', 'hitprocess/clas12/svt', 'hitprocess/clas12/micromegas'])
#env.Append(CPPPATH = ['hitprocess/Aprime', 'hitprocess/GlueX', 'hitprocess/solid'])

set(hitp_sources
  hitprocess/HitProcess_MapRegister.cc
  hitprocess/flux_hitprocess.cc
  hitprocess/counter_hitprocess.cc
  hitprocess/clas12/micromegas/Lorentz.cc
  hitprocess/clas12/micromegas/FMT_hitprocess.cc
  hitprocess/clas12/micromegas/fmt_strip.cc
  hitprocess/clas12/micromegas/BMT_hitprocess.cc
  hitprocess/clas12/micromegas/bmt_strip.cc
  hitprocess/clas12/micromegas/ftm_hitprocess.cc
  hitprocess/clas12/micromegas/ftm_strip.cc
  hitprocess/clas12/rtpc_hitprocess.cc
  hitprocess/clas12/svt/bst_hitprocess.cc
  hitprocess/clas12/svt/bst_strip.cc
  hitprocess/clas12/ctof_hitprocess.cc
  hitprocess/clas12/cnd_hitprocess.cc
  hitprocess/clas12/dc_hitprocess.cc
  hitprocess/clas12/ec_hitprocess.cc
  hitprocess/clas12/ecs_hitprocess.cc
  hitprocess/clas12/ftof_hitprocess.cc
  hitprocess/clas12/ft_cal_hitprocess.cc
  hitprocess/clas12/ft_hodo_hitprocess.cc
  hitprocess/clas12/htcc_hitprocess.cc
  hitprocess/clas12/ltcc_hitprocess.cc
  hitprocess/clas12/pcal_hitprocess.cc
  hitprocess/clas12/rich_hitprocess.cc
  hitprocess/bdx/cormo_hitprocess.cc
  hitprocess/bdx/veto_hitprocess.cc
  hitprocess/bdx/crs_hitprocess.cc
  hitprocess/eic/eic_compton_hitprocess.cc
  hitprocess/eic/eic_dirc_hitprocess.cc
  hitprocess/eic/eic_ec_hitprocess.cc
  hitprocess/eic/eic_preshower_hitprocess.cc
  hitprocess/eic/eic_rich_hitprocess.cc
  hitprocess/injector/bubble_hitprocess.cc
  hitprocess/HPS/ECAL_hitprocess.cc
  hitprocess/HPS/SVT_hitprocess.cc
  hitprocess/HPS/muon_hodo_hitprocess.cc)

# Output
set(output_sources
  output/outputFactory.cc
  output/evio_output.cc
  output/txt_output.cc
  output/txt_simple_output.cc
  output/gbank.cc)

set(gui_sources
  gui/src/gemc_MainGui.cc
  gui/src/detector_editor.cc
  gui/src/runControl/run_control.cc
  gui/src/runControl/primaryTab.cc
  gui/src/runControl/momControls.cc
  gui/src/runControl/vtxControls.cc
  gui/src/camera_control.cc
  gui/src/detector_tree.cc
  gui/src/infos.cc
  gui/src/g4dialog.cc
  gui/src/gsignal.cc
  gui/src/physicsListGui.cc
  gui/src//gtrigger.cc
  utilities/graph.cc)

set(gemc_sources
  gemc.cc
  src/dmesg_init.cc
  src/run_conditions.cc
  src/gemc_options.cc
  src/MDetectorConstruction.cc
  src/MEventAction.cc
  src/MPrimaryGeneratorAction.cc
  src/ActionInitialization.cc
  src/MSteppingAction.cc)


#env.Prepend(LIBS =  ['gmaterials', 'gmirrors', 'gparameters', 'gutilities', 'gdetector', 'gsensitivity', 'gphysics', 'gfields', 'ghitprocess', 'goutput', 'ggui'])
#env.Program(source = gemc_sources, target = "gemc")

add_library(libGEMC SHARED  
  ${phys_sources}
  ${mirrors_sources}
  ${materials_sources}
  ${parameters_sources}
  ${util_sources}
  ${det_sources}
  ${sensi_sources}
  ${field_sources}
  ${hitp_sources}
  ${output_sources}
  ${gui_sources}
  )

target_include_directories(libGEMC
  PUBLIC #$<INSTALL_INTERFACE:include>
  $<INSTALL_INTERFACE:include>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/options>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/utilities>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/hitprocess>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/gui/src>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/gui>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/mirrors>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/materials>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/parameters>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/detector>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/detectors>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/fields>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/generator>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/output>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/physics>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/sensitivity>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>

    PUBLIC ${MLibrary_INCLUDE_DIRS}
    PUBLIC ${EVIO_INCLUDE_DIRS} /usr/local/include/evio
    PUBLIC ${CADMESH_INCLUDE_DIRS}
  )
message("EVIO_INCLUDE_DIRS  ${EVIO_INCLUDE_DIRS}")

#jtarget_compile_options(GEMC
#j  PUBLIC
#j    ${${PROJECT_NAME_UC}_CXX_FLAGS_LIST}
#j  PRIVATE
#j    ${${PROJECT_NAME_UC}_DIAG_FLAGS_LIST}
#j  )

#target_compile_features(GEMC PUBLIC cxx_std_14)


target_link_libraries(libGEMC
  PUBLIC Qt5::Widgets
  PUBLIC Qt5::Xml
  PUBLIC Qt5::Sql
  PUBLIC ${Geant4_LIBRARIES} ${cadmesh_LIBRARIES} 
  PUBLIC ${MLibrary_LIBRARIES}
  PUBLIC -L${CMAKE_INSTALL_PREFIX}/lib -lMLibrary
  PUBLIC EVIO::EVIO
  PUBLIC ${cadmesh_LIBRARIES}
  ${CMAKE_DL_LIBS}
  )

#set_target_properties(${LIBNAME} PROPERTIES
#  SOVERSION ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
#  VERSION ${PROJECT_VERSION}
#  EXPORT_NAME ${LIBNAME}
#  )

add_executable(GEMC  ${gemc_sources} 
  )

target_include_directories(GEMC
  PUBLIC #$<INSTALL_INTERFACE:include>
  $<INSTALL_INTERFACE:include>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/options>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/utilities>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/hitprocess>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/gui/src>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/gui>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/mirrors>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/materials>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/parameters>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/detector>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/detectors>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/fields>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/generator>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/output>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/physics>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/sensitivity>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>

    PUBLIC ${MLibrary_INCLUDE_DIRS}
    PUBLIC ${EVIO_INCLUDE_DIRS}
    PUBLIC ${CADMESH_INCLUDE_DIRS}
  )

#jtarget_compile_options(GEMC
#j  PUBLIC
#j    ${${PROJECT_NAME_UC}_CXX_FLAGS_LIST}
#j  PRIVATE
#j    ${${PROJECT_NAME_UC}_DIAG_FLAGS_LIST}
#j  )

#target_compile_features(GEMC PUBLIC cxx_std_14)

message("cadmesh_LIBRARIES: ${cadmesh_LIBRARIES}")
target_link_libraries(GEMC
  PUBLIC libGEMC
  PUBLIC Qt5::Widgets
  PUBLIC Qt5::Xml
  PUBLIC Qt5::Sql
  PUBLIC ${Geant4_LIBRARIES} 
  PUBLIC ${cadmesh_LIBRARIES} 
  PUBLIC ${MLibrary_LIBRARIES}
  PUBLIC -L${CMAKE_INSTALL_PREFIX}/lib -lMLibrary
  PUBLIC -L${CMAKE_INSTALL_PREFIX}/lib -levio -levioxx
  PUBLIC -L${CMAKE_INSTALL_PREFIX}/lib -lccdb
  PUBLIC EVIO::EVIO
  PUBLIC ${cadmesh_LIBRARIES}
  ${CMAKE_DL_LIBS}
  PUBLIC ROOT::Core ROOT::MathCore ROOT::Graf ROOT::Matrix ROOT::Physics ROOT::Rint ROOT::Tree
  )


install(TARGETS libGEMC GEMC
  EXPORT GEMCTargets
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  )




#install(FILES ${allheaders} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
##install(DIRECTORY nlohmann DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
#
#install(DIRECTORY fields DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY frequencySyncSignal DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY g4display DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY g4volume DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY gdata DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY gdynamic DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY gfactory DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY gfields DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY ghit DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY gmedia DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY gruns DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY gstring DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY gtouchable DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY gvolume DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY options DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY splash DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY qtButtonsWidget DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY textProgressBar DESTINATION include FILES_MATCHING PATTERN "*.h")
#install(DIRECTORY translationTable DESTINATION include FILES_MATCHING PATTERN "*.h")
#      #install(DIRECTORY include/hcana  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
#install(DIRECTORY include/spdlog DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/hcana)
#file(GLOB sources ${PROJECT_SOURCE_DIR}/src/*.cc)
#file(GLOB headers ${PROJECT_SOURCE_DIR}/include/*.hh ${PROJECT_BINARY_DIR}/GEMC.h)

##----------------------------------------------------------------------------
## install library config
#include(CMakePackageConfigHelpers)
#
## Install and export targets
#install(EXPORT GEMCTargets
#    FILE GEMCTargets.cmake
#    NAMESPACE GEMC::
#    DESTINATION lib/GEMC
#    )
#
#set(TARGETS_INSTALL_PATH lib/GEMC/GEMC.cmake)
#CONFIGURE_PACKAGE_CONFIG_FILE(
#  cmake/GEMCConfig.cmake.in  
#  ${CMAKE_CURRENT_BINARY_DIR}/GEMCConfig.cmake
#  INSTALL_DESTINATION lib
#  PATH_VARS TARGETS_INSTALL_PATH
#  )
#
#write_basic_package_version_file("GEMCConfigVersion.cmake"
#    VERSION ${VERSION}
#    COMPATIBILITY SameMajorVersion
#    )
#
#install(FILES
#  ${CMAKE_CURRENT_BINARY_DIR}/GEMCConfig.cmake
#  ${CMAKE_CURRENT_BINARY_DIR}/GEMCConfigVersion.cmake
#  DESTINATION lib/GEMC
#  )


# Add the executable, and link it to the Geant4 libraries
#
#add_executable(exampleB1 exampleB1.cc ${sources} ${headers})
#target_include_directories(exampleB1 PUBLIC ${CADMESH_INCLUDE_DIRS} )
#target_link_libraries(exampleB1 ${Geant4_LIBRARIES} ${cadmesh_LIBRARIES} )

#add_executable(bubble_chamber bubble_chamber.cc ${sources} ${headers})
#target_include_directories(bubble_chamber PUBLIC ${CADMESH_INCLUDE_DIRS} )
#target_link_libraries(bubble_chamber ${Geant4_LIBRARIES} ${cadmesh_LIBRARIES})
#
##----------------------------------------------------------------------------
## For internal Geant4 use - but has no effect if you build this
## example standalone
##
#add_custom_target(GEMC DEPENDS bubble_chamber)
#
##----------------------------------------------------------------------------
## Copy all scripts to the build directory, i.e. the directory in which we
## build B1. This is so that we can run the executable directly because it
## relies on these scripts being in the current working directory.
##
#set(EXAMPLEB1_SCRIPTS
#  exampleB1.in
#  exampleB1.out
#  examples/init_vis.mac
#  examples/run1.mac
#  examples/run2.mac
#  examples/vis.mac
#  examples/vis2.mac
#  examples/trajectory_model.mac
#  )
#foreach(_script ${EXAMPLEB1_SCRIPTS})
#  configure_file(
#    ${PROJECT_SOURCE_DIR}/${_script}
#    ${PROJECT_BINARY_DIR}/${_script}
#    COPYONLY
#    )
#endforeach()
#install(FILES ${EXAMPLEB1_SCRIPTS} 
#  DESTINATION share/${SHARE_DIR_NAME}/examples )
#
#install(DIRECTORY  ${PROJECT_SOURCE_DIR}/geometry_files
#  DESTINATION share/${SHARE_DIR_NAME} )
#
#install(FILES
#  ${PROJECT_SOURCE_DIR}/bin/run_bubble_sim 
#  DESTINATION bin
#  PERMISSIONS WORLD_READ WORLD_EXECUTE
#  )
#
#
##----------------------------------------------------------------------------
## Install the executable to 'bin' directory under CMAKE_INSTALL_PREFIX
##
##install(TARGETS exampleB1 DESTINATION bin)
#install(TARGETS bubble_chamber DESTINATION bin)
#
## ----------------------------------------------------------------------------
## Configured files 
## These files end with ".in" which is removed when installed
#set(EXAMPLEB1_CONF_FILES
#  BubbleChamberSim.h
#  init_default.mac
#  #vis.mac
#  )
#foreach(_script ${EXAMPLEB1_CONF_FILES})
#  configure_file("${_script}.in" "${PROJECT_BINARY_DIR}/${_script}" @ONLY)
#  install(FILES ${PROJECT_BINARY_DIR}/${_script} DESTINATION ${CMAKE_INSTALL_PREFIX}/share/${SHARE_DIR_NAME} )
#endforeach()
#
#
### Add all targets to the build-tree export set
#export(TARGETS bubble_chamber FILE "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Targets.cmake")
##
### Export the package for use from the build-tree
### (this registers the build-tree with a global CMake-registry)
##export(PACKAGE ${PROJECT_NAME})
##
##set(CONF_LIBRARIES ${PROJECT_NAME})
##
##set(CONF_EXECUTABLES insane-config)
##
##set(CONF_INCLUDE_DIRS 
##   ${CMAKE_INSTALL_PREFIX}/include
##   ${CMAKE_INSTALL_PREFIX}/include/${PROJECT_NAME}
##   )
##
##set(CONF_LIBRARY_DIRS ${CMAKE_INSTALL_PREFIX}/lib)
##
###configure_file(${PROJECT_NAME}Config.cmake.in "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${PROJECT_NAME}Config.cmake" @ONLY)
##
##configure_file(${PROJECT_NAME}ConfigVersion.cmake.in "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake" @ONLY)
##
### Install the FooBarConfig.cmake and FooBarConfigVersion.cmake
##install(FILES
##  "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${PROJECT_NAME}Config.cmake"
##  "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
##  DESTINATION "${INSTALL_CMAKE_DIR}" COMPONENT dev)
##
### Install the export set for use with the install-tree
##install(EXPORT ${PROJECT_NAME}Targets DESTINATION
##  "${INSTALL_CMAKE_DIR}" COMPONENT dev)

