set(PYBIND_DEPS pybind python proto_desc memory executor fleet_wrapper box_wrapper nccl_wrapper prune
  feed_fetch_method pass_builder parallel_executor profiler layer tracer engine scope_pool
  analysis_predictor imperative_profiler nccl_context imperative_flag)

if(WITH_PYTHON)
  list(APPEND PYBIND_DEPS py_func_op)
endif()

if (WITH_DISTRIBUTE)
  list(APPEND PYBIND_DEPS communicator)
endif()

set(PYBIND_SRCS
  pybind.cc
  exception.cc
  protobuf.cc
  const_value.cc
  reader_py.cc
  fleet_wrapper_py.cc
  box_helper_py.cc
  nccl_wrapper_py.cc
  data_set_py.cc
  imperative.cc
  ir.cc
  inference_api.cc)

if (WITH_DISTRIBUTE)
  list(APPEND PYBIND_SRCS communicator_py.cc)
endif()

if(WITH_PYTHON)
  if(WITH_AMD_GPU)
    hip_library(paddle_pybind SHARED
      SRCS ${PYBIND_SRCS}
      DEPS ARCHIVE_START ${PYBIND_DEPS}
      ${GLOB_OP_LIB} ${GLOB_OPERATOR_DEPS} ARCHIVE_END)
  else()
    cc_library(paddle_pybind SHARED
      SRCS ${PYBIND_SRCS}
      DEPS ${PYBIND_DEPS}
      ${GLOB_OP_LIB} ${GLOB_OPERATOR_DEPS})
    if(NOT APPLE AND NOT WIN32)
      target_link_libraries(paddle_pybind rt)
    endif(NOT APPLE AND NOT WIN32)
  endif(WITH_AMD_GPU)

  get_property (os_dependency_modules GLOBAL PROPERTY OS_DEPENDENCY_MODULES)
  target_link_libraries(paddle_pybind ${os_dependency_modules})

endif(WITH_PYTHON)
