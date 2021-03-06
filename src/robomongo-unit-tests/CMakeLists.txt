#########################################
###   Unit Testing with Google Test   ###
#########################################

### --- Important Note
if (SYSTEM_LINUX)        
    message("\n *Note: Currently unit testing is disabled for Linux " 
                "due to MongoDB linking problems")
    return()
endif()

### --- Enable and setup gtest 
enable_testing()
include_directories(${gtest_SOURCE_DIR}/include ${gtest_SOURCE_DIR})
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

### --- Setup test source files
# New test & source file pairs MUST have the following format: 
#  Test file:   /path/Foo_test.cpp
#  Source file: /path/Foo.cpp or /path/Foo.h
set(ROBO_SRC_DIR ${CMAKE_HOME_DIRECTORY}/src/robomongo)
set(SOURCES_TEST
    ${ROBO_SRC_DIR}/utils/RoboCrypt_test.cpp
    ${ROBO_SRC_DIR}/utils/StringOperations_test.cpp
    ${ROBO_SRC_DIR}/core/HexUtils_test.cpp
)

### --- Setup robo_unit_tests exec. & link ROBO_OBJ_FILES
add_executable(robo_unit_tests ${SOURCES_TEST})
add_dependencies(robo_unit_tests robomongo)
target_include_directories(robo_unit_tests PRIVATE ${CMAKE_HOME_DIRECTORY}/src)

get_target_property(ROBO_SOURCES robomongo SOURCES)
list(FILTER ROBO_SOURCES INCLUDE REGEX "cpp")
list(FILTER ROBO_SOURCES EXCLUDE REGEX "main.cpp")

if(SYSTEM_WINDOWS)
    set(OBJ_DIR ${CMAKE_BINARY_DIR}/src/robomongo/robomongo.dir/${CMAKE_BUILD_TYPE}/)
    set(ROBO_OBJ_FILES "${ROBO_OBJ_FILES}${OBJ_DIR}mocs_compilation.obj;")
    set(ROBO_OBJ_FILES "${ROBO_OBJ_FILES}${OBJ_DIR}qrc_gui.obj;")
    set(ROBO_OBJ_FILES "${ROBO_OBJ_FILES}${OBJ_DIR}qrc_robo.obj;")
    foreach(SRC_FILE ${ROBO_SOURCES})
        get_filename_component(FILENAME ${SRC_FILE} NAME)
        string(REPLACE ".cpp" ".obj" FILENAME ${FILENAME})
        set(ROBO_OBJ_FILES "${ROBO_OBJ_FILES}${OBJ_DIR}${FILENAME};")        
    endforeach()
elseif(SYSTEM_MACOSX) 
    set(OBJ_DIR ${CMAKE_BINARY_DIR}/src/robomongo/CMakeFiles/robomongo.dir/)
    set(ROBO_OBJ_FILES "${ROBO_OBJ_FILES}${OBJ_DIR}/robomongo_autogen/mocs_compilation.cpp.o;")
    set(ROBO_OBJ_FILES "${ROBO_OBJ_FILES}${OBJ_DIR}/robomongo_autogen/YHP5W5E6RA/qrc_gui.cpp.o;")
    set(ROBO_OBJ_FILES "${ROBO_OBJ_FILES}${OBJ_DIR}/robomongo_autogen/3YJK5W5UP7/qrc_robo.cpp.o;")
    foreach(SRC_FILE ${ROBO_SOURCES})
        string(REPLACE ".cpp" ".cpp.o" OBJ_FILE ${SRC_FILE})
        set(ROBO_OBJ_FILES "${ROBO_OBJ_FILES}${OBJ_DIR}${OBJ_FILE};")        
    endforeach()

    find_library(SECURITY NAMES Security)
    find_library(CORE_FOUNDATION NAMES CoreFoundation)
    set(SSL_LIBRARIES ${SECURITY} ${CORE_FOUNDATION})
    target_link_libraries(robo_unit_tests ${SSL_LIBRARIES} -lresolv)
# elseif(SYSTEM_LINUX) 
#     set(OBJ_DIR ${CMAKE_BINARY_DIR}/src/robomongo/CMakeFiles/robomongo.dir/)
#     message("--- OBJ_DIR: " ${OBJ_DIR})
#     set(ROBO_OBJ_FILES "${ROBO_OBJ_FILES}${OBJ_DIR}/robomongo_automoc.cpp.o;")
#     set(ROBO_OBJ_FILES "${ROBO_OBJ_FILES}${OBJ_DIR}/robomongo_automoc.dir/gui/resources/qrc_gui.cpp.o;")
#     set(ROBO_OBJ_FILES "${ROBO_OBJ_FILES}${OBJ_DIR}/robomongo_automoc.dir/resources/qrc_robo.cpp.o;")    
#     foreach(SRC_FILE ${ROBO_SOURCES})
#         string(REPLACE ".cpp" ".cpp.o" OBJ_FILE ${SRC_FILE})
#         set(ROBO_OBJ_FILES "${ROBO_OBJ_FILES}${OBJ_DIR}${OBJ_FILE};")        
#     endforeach()
endif()

# Disable WebEngineWidgets for Linux
SET(WebEngineWidgets "Qt5::WebEngineWidgets") 
if(SYSTEM_LINUX)
   SET(WebEngineWidgets)  
endif()

target_link_libraries(robo_unit_tests 
    gtest 
    gtest_main
    Qt5::Widgets
    Qt5::Network
    Qt5::Xml
    ${WebEngineWidgets}
    qjson
    qscintilla
    mongodb
    ssh
    Threads::Threads
    ${ROBO_OBJ_FILES}
)

### --- Install DLLs for Windows
if(CMAKE_BUILD_TYPE STREQUAL "Debug") 
    set(D "d") 
endif()

if(SYSTEM_WINDOWS)
    get_target_property(target_path Qt5::Core LOCATION)
    get_filename_component(qt_bin_dir ${target_path} DIRECTORY)
    set(QT_BIN_FILES "${QT_BIN_FILES}" ${qt_bin_dir}/Qt5Core${D}${CMAKE_SHARED_LIBRARY_SUFFIX};)
    set(QT_BIN_FILES "${QT_BIN_FILES}" ${qt_bin_dir}/Qt5Gui${D}${CMAKE_SHARED_LIBRARY_SUFFIX};)
    set(QT_BIN_FILES "${QT_BIN_FILES}" ${qt_bin_dir}/Qt5Network${D}${CMAKE_SHARED_LIBRARY_SUFFIX};)
    set(QT_BIN_FILES "${QT_BIN_FILES}" ${qt_bin_dir}/Qt5PrintSupport${D}${CMAKE_SHARED_LIBRARY_SUFFIX};)
    set(QT_BIN_FILES "${QT_BIN_FILES}" ${qt_bin_dir}/Qt5Widgets${D}${CMAKE_SHARED_LIBRARY_SUFFIX};)
    set(QT_BIN_FILES "${QT_BIN_FILES}" ${qt_bin_dir}/Qt5Xml${D}${CMAKE_SHARED_LIBRARY_SUFFIX};)
    set(QT_BIN_FILES "${QT_BIN_FILES}" ${qt_bin_dir}/Qt5Positioning${D}${CMAKE_SHARED_LIBRARY_SUFFIX};)
    set(QT_BIN_FILES "${QT_BIN_FILES}" ${qt_bin_dir}/Qt5Qml${D}${CMAKE_SHARED_LIBRARY_SUFFIX};)
    set(QT_BIN_FILES "${QT_BIN_FILES}" ${qt_bin_dir}/Qt5Quick${D}${CMAKE_SHARED_LIBRARY_SUFFIX};)
    set(QT_BIN_FILES "${QT_BIN_FILES}" ${qt_bin_dir}/Qt5QuickWidgets${D}${CMAKE_SHARED_LIBRARY_SUFFIX};)
    set(QT_BIN_FILES "${QT_BIN_FILES}" ${qt_bin_dir}/Qt5WebChannel${D}${CMAKE_SHARED_LIBRARY_SUFFIX};)
    set(QT_BIN_FILES "${QT_BIN_FILES}" ${qt_bin_dir}/Qt5WebEngineCore${D}${CMAKE_SHARED_LIBRARY_SUFFIX};)
    set(QT_BIN_FILES "${QT_BIN_FILES}" ${qt_bin_dir}/Qt5WebEngineWidgets${D}${CMAKE_SHARED_LIBRARY_SUFFIX};)
    set(QT_BIN_FILES "${QT_BIN_FILES}" ${qt_bin_dir}/Qt5PrintSupport${D}${CMAKE_SHARED_LIBRARY_SUFFIX};)

    get_target_property(TESTS_BIN_DIR robo_unit_tests BINARY_DIR)
    file(COPY ${QT_BIN_FILES} DESTINATION ${TESTS_BIN_DIR}/${CMAKE_BUILD_TYPE})
    file(COPY 
        ${OpenSSL_DIR}/libssl-1_1-x64.dll
        ${OpenSSL_DIR}/libcrypto-1_1-x64.dll
        DESTINATION ${TESTS_BIN_DIR}/${CMAKE_BUILD_TYPE})
endif()