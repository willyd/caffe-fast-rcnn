# --[ Proto

add_subdirectory(proto)

# --[ Caffe library

# creates 'test_srcs', 'srcs', 'test_cuda', 'cuda' lists
caffe_pickup_caffe_sources(${PROJECT_SOURCE_DIR})

if(HAVE_CUDA)
  # define preprocessor macro so that we will not include the generated forcelink header
  list(APPEND _additional_definitions -DBUILDING_CAFFE_LIB)
  # get definitions from import libraries as they are not forwarded to nvcc
  # see http://www.cmake.org/Bug/view.php?id=14201
  foreach(_lib ${Caffe_LINKER_LIBS})
    if(TARGET ${_lib})
        get_target_property(_compiled_defs ${_lib} INTERFACE_COMPILE_DEFINITIONS)
        if(_compiled_defs)
            foreach(_compiled_def ${_compiled_defs})
                list(APPEND _additional_definitions -D${_compiled_def})
            endforeach()
        endif()
    endif()
  endforeach()
  caffe_cuda_compile(cuda_objs ${cuda} OPTIONS ${_additional_definitions})
  list(APPEND srcs ${cuda_objs} ${cuda})
endif()

add_library(caffe ${srcs})
target_link_libraries(caffe proto ${Caffe_LINKER_LIBS})
caffe_default_properties(caffe)

# ---[ Visual Studio specifics
#  configure headers for dynamic or static linking on windows
unset(CAFFE_DLLEXPORT)
unset(CAFFE_DLLIMPORT)
unset(CAFFE_INCLUDE_FORCELINK)
if(MSVC)
    if(BUILD_SHARED_LIBS)
        set(CAFFE_DLLEXPORT __declspec(dllexport))
        set(CAFFE_DLLIMPORT __declspec(dllimport))
    else()
        get_filename_component(forcelink_header_name ${forcelink_header} NAME)
        set(CAFFE_INCLUDE_FORCELINK "#      include \"caffe/${forcelink_header_name}\"")
        # add a post build command to generate a header file that will
        # force the linker to include the statically registered layers
        # in a final executable.
        include(${PROJECT_SOURCE_DIR}/cmake/GenerateForceLinkSymbolsHeader.cmake)    
        add_generate_force_link_symbols_header_post_build_command(caffe ${forcelink_header} caffe_force_link)        
        # install the generated file
        install(FILES ${forcelink_header} DESTINATION include/caffe)
    endif()        
endif()

configure_file("${PROJECT_SOURCE_DIR}/cmake/Templates/export.hpp.in" 
                ${export_header} @ONLY)
# define preprocessor macro so that we will not include the generated forcelink header
target_compile_definitions(caffe PRIVATE -DBUILDING_CAFFE_LIB)

# ---[ Tests
 add_subdirectory(test)

# ---[ Install
install(DIRECTORY ${Caffe_INCLUDE_DIR}/caffe DESTINATION include)
install(FILES ${export_header} DESTINATION include/caffe)

install(TARGETS caffe EXPORT CaffeTargets DESTINATION lib)




