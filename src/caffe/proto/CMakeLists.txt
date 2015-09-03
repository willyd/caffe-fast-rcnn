# generate protobuf sources
file(GLOB proto_files *.proto)
# Use CMAKE_CURRENT_BINARY_DIR since it causes build issues on Windows 
# The appropriate include folder is added in Protobuf.cmake so that every project
# can find caffe.pb.h
# See https://github.com/BVLC/caffe/issues/2987
caffe_protobuf_generate_cpp_py(${proto_gen_folder} proto_srcs proto_hdrs proto_python ${proto_files})

# include python files either to force generation
add_library(proto STATIC ${proto_hdrs} ${proto_srcs} ${proto_python})

set(Caffe_LINKER_LIBS proto ${Caffe_LINKER_LIBS}) # note, crucial to prepend!
caffe_default_properties(proto)

install(TARGETS proto EXPORT CaffeTargets DESTINATION lib)
install(FILES ${proto_hdrs} DESTINATION include/caffe/proto)
file(WRITE ${PROJECT_BINARY_DIR}/__init__.py)
list(APPEND proto_python ${PROJECT_BINARY_DIR}/__init__.py)
install(PROGRAMS ${proto_python} DESTINATION python/caffe/proto)