set(NEBULA_INCLUDE_DIRS
        "${PROJECT_SOURCE_DIR}/lib/include"
        "${PROJECT_BINARY_DIR}/lib/include")

include_directories(${NEBULA_INCLUDE_DIRS})

set(SAMPLES
        hello-world
        )

foreach(SAMPLE ${SAMPLES})
    add_executable(${SAMPLE}
            ${SAMPLE}.c)

    target_link_libraries(${SAMPLE}
            helloworld)

    add_dependencies(${SAMPLE} helloworld)
    set_target_properties(${SAMPLE} PROPERTIES DEFINE_SYMBOL NEBULA_LIB_DLL)
endforeach()