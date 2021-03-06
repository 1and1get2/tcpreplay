find_package(Qt4)
if(Qt4_FOUND AND BUILD_QT_GUI)
    set(PROGRAM "tcpreplay-gui")
    set(GUI_PATH ${CMAKE_SOURCE_DIR}/src/gui)

    set(QT_USE_QTMAIN 1)
    include(${QT_USE_FILE})

    include_directories(${CMAKE_SOURCE_DIR}/src)
    include_directories(${CMAKE_SOURCE_DIR}/src/tcpedit)

    file(GLOB sources
        ${GUI_PATH}/*.cpp
    )
    file(GLOB ui_files
        ${GUI_PATH}/*.ui
    )
    set(moc_headers
        ${GUI_PATH}/GuiTcpEdit.h
        ${GUI_PATH}/GuiTcpReplay.h
    )
    set(resources
        ${GUI_PATH}/${PROGRAM}.qrc
    )

    QT4_WRAP_CPP(moc_sources ${moc_headers})
    QT4_WRAP_UI(ui_headers ${ui_files})

    # No resource file for now...
    #QT4_ADD_RESOURCES(resources_sources ${resources})

    add_executable(${PROGRAM}
        ${sources}
        ${ui_headers}
        ${moc_sources}
        ${resources_sources}
    )

    link_directories(
        ${CMAKE_BINARY_DIR}
        ${CMAKE_BINARY_DIR}/src
        ${CMAKE_BINARY_DIR}/src/common
        ${CMAKE_BINARY_DIR}/src/tcpedit
    )

    # default libraries
    set(main_libs pcap common tcpedit)

    # Only include our strlcpy if the system doesn't provide it
    if(NOT SYSTEM_STRLCPY)
        set(main_libs ${main_libs} strl)
        link_directories(${CMAKE_BINARY_DIR}/lib)
    endif(NOT SYSTEM_STRLCPY)

    set(lib_to_link ${QT_LIBRARIES} ${QT_QTNETWORK_LIBRARY} ${main_libs})

    target_link_libraries(
        ${PROGRAM}
        ${lib_to_link}
    )

    if(NOT MSVC)
        ADD_CUSTOM_COMMAND(TARGET ${PROGRAM} POST_BUILD COMMAND sudo ARGS chown root:root ${PROGRAM})
        ADD_CUSTOM_COMMAND(TARGET ${PROGRAM} POST_BUILD COMMAND sudo ARGS chmod 4555 ${PROGRAM})
    endif(NOT MSVC)
endif(Qt4_FOUND AND BUILD_QT_GUI)
