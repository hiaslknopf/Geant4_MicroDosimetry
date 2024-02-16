#----------------------------------------------------------------------------
# Setup the project
cmake_minimum_required(VERSION 3.16...3.27)
project(exp_microdosimetry)

#----------------------------------------------------------------------------
# Find Geant4 package, activating all available Vis drivers by default
# You can set WITH_GEANT4_VIS to OFF via the command line or ccmake/cmake-gui
# to build a batch mode only executable
#
option(WITH_GEANT4_UIVIS "Build example with Geant4 UI and Vis drivers" ON)
if(WITH_GEANT4_UIVIS)
  find_package(Geant4 REQUIRED ui_all vis_all)
else()
  find_package(Geant4 REQUIRED)
endif()
#----------------------------------------------------------------------------
# Setup Geant4 include directories and compile definitions
#
include(${Geant4_USE_FILE})
# Setup of ROOT ANALYSIS : optional.
option(WITH_ANALYSIS_USE "Build example with analysis objects" ON)
if(WITH_ANALYSIS_USE)
add_definitions(-DANALYSIS_USE)
else()
endif()

#----------------------------------------------------------------------------
# Locate sources and headers for this project
#
include_directories(${PROJECT_SOURCE_DIR}/include
                     ${Geant4_INCLUDE_DIR})
file(GLOB sources ${PROJECT_SOURCE_DIR}/src/*.cc)
file(GLOB headers ${PROJECT_SOURCE_DIR}/include/*.hh)

#----------------------------------------------------------------------------
# Add the executable, and link it to the Geant4 libraries
#
add_executable(exp_microdosimetry exp_microdosimetry exp_microdosimetry.cc ${sources} ${headers})
target_link_libraries(exp_microdosimetry ${Geant4_LIBRARIES})

#----------------------------------------------------------------------------
# Find all .mac files in macros subdirectory
file(GLOB_RECURSE exp_microdosimetry_MACROS
     "${PROJECT_SOURCE_DIR}/macros/*.mac")

# Find .C and .py files specifically in the analysis subdirectory
file(GLOB_RECURSE exp_microdosimetry_C_FILES
     "${PROJECT_SOURCE_DIR}/analysis/*.C")
file(GLOB_RECURSE exp_microdosimetry_PY_FILES
     "${PROJECT_SOURCE_DIR}/analysis/*.py")

# Combine all found files into a list
set(exp_microdosimetry_SCRIPTS ${exp_microdosimetry_MACROS} ${exp_microdosimetry_C_FILES} ${exp_microdosimetry_PY_FILES})

# Copy each found file to the build directory
foreach(_script ${exp_microdosimetry_SCRIPTS})
  get_filename_component(_script_name "${_script}" NAME)
  get_filename_component(_script_path "${_script}" PATH)
  file(RELATIVE_PATH _relative_path "${PROJECT_SOURCE_DIR}" "${_script_path}")
  
  # Ensure the target directory exists
  file(MAKE_DIRECTORY "${PROJECT_BINARY_DIR}/${_relative_path}")
  
  file(COPY ${_script} DESTINATION "${PROJECT_BINARY_DIR}/${_relative_path}")
endforeach()

# Create an empty /analysis/output directory in the build directory if it doesn't already exist
file(MAKE_DIRECTORY "${PROJECT_BINARY_DIR}/analysis/output")

#----------------------------------------------------------------------------
# Install the executable to 'bin' directory under CMAKE_INSTALL_PREFIX
#
install(TARGETS exp_microdosimetry DESTINATION bin)

