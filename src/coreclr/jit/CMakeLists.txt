set(CMAKE_INCLUDE_CURRENT_DIR ON)

include_directories("./jitstd")
include_directories("../inc")

if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  add_compile_options($<$<COMPILE_LANGUAGE:CXX>:-fpermissive>)
  add_compile_options(-Wno-error)
endif()

add_compile_options($<$<CXX_COMPILER_ID:MSVC>:-W4>)

function(create_standalone_jit)

  set(oneValueArgs TARGET OS ARCH)
  set(multiValueArgs ADDITIONAL_DESTINATIONS)
  set(options)
  cmake_parse_arguments(TARGETDETAILS "${options}" "${oneValueArgs}" "${multiValueArgs}" ${ARGN})

  if(TARGETDETAILS_OS STREQUAL "unix_osx")
    if (NOT (TARGETDETAILS_ARCH STREQUAL "arm64"))
      message(FATAL_ERROR "Only arm64 Apple has a special ABI, use just unix for x64 Mac OS." )
    endif()
    set(JIT_ARCH_LINK_LIBRARIES gcinfo_unix_arm64)
  else()
    set(JIT_ARCH_LINK_LIBRARIES gcinfo_${TARGETDETAILS_OS}_${TARGETDETAILS_ARCH})
  endif()

  if(TARGETDETAILS_ARCH STREQUAL "x64")
    set(JIT_ARCH_SOURCES ${JIT_AMD64_SOURCES})
  elseif((TARGETDETAILS_ARCH STREQUAL "arm") OR (TARGETDETAILS_ARCH STREQUAL "armel"))
    set(JIT_ARCH_SOURCES ${JIT_ARM_SOURCES})
  elseif(TARGETDETAILS_ARCH STREQUAL "x86")
    set(JIT_ARCH_SOURCES ${JIT_I386_SOURCES})
  elseif(TARGETDETAILS_ARCH STREQUAL "arm64")
    set(JIT_ARCH_SOURCES ${JIT_ARM64_SOURCES})
  else()
    clr_unknown_arch()
  endif()

  if (TARGETDETAILS_ADDITIONAL_DESTINATIONS STREQUAL "")
    add_jit(${TARGETDETAILS_TARGET})
  else()
    add_jit(${TARGETDETAILS_TARGET} ADDITIONAL_DESTINATIONS "${TARGETDETAILS_ADDITIONAL_DESTINATIONS}")
  endif()

  set_target_definitions_to_custom_os_and_arch(${ARGN})
  set_target_properties(${TARGETDETAILS_TARGET} PROPERTIES IGNORE_FEATURE_MERGE_JIT_AND_ENGINE TRUE)

  target_compile_definitions(${TARGETDETAILS_TARGET} PRIVATE FEATURE_NO_HOST)
  target_compile_definitions(${TARGETDETAILS_TARGET} PRIVATE SELF_NO_HOST)
  if(FEATURE_READYTORUN)
    target_compile_definitions(${TARGETDETAILS_TARGET} PRIVATE FEATURE_READYTORUN_COMPILER)
  endif(FEATURE_READYTORUN)

  if ((TARGETDETAILS_ARCH STREQUAL "x64") OR (TARGETDETAILS_ARCH STREQUAL "arm64") OR ((TARGETDETAILS_ARCH STREQUAL "x86") AND NOT (TARGETDETAILS_OS STREQUAL "unix")))
    target_compile_definitions(${TARGETDETAILS_TARGET} PRIVATE FEATURE_SIMD)
    target_compile_definitions(${TARGETDETAILS_TARGET} PRIVATE FEATURE_HW_INTRINSICS)
  endif ()
endfunction()

if (CLR_CMAKE_TARGET_ARCH_AMD64 OR CLR_CMAKE_TARGET_ARCH_ARM64 OR (CLR_CMAKE_TARGET_ARCH_I386 AND NOT CLR_CMAKE_HOST_UNIX))
  add_compile_definitions($<$<NOT:$<BOOL:$<TARGET_PROPERTY:IGNORE_DEFAULT_TARGET_ARCH>>>:FEATURE_SIMD>)
  add_compile_definitions($<$<NOT:$<BOOL:$<TARGET_PROPERTY:IGNORE_DEFAULT_TARGET_ARCH>>>:FEATURE_HW_INTRINSICS>)
endif ()

# JIT_BUILD disables certain PAL_TRY debugging features
add_definitions(-DJIT_BUILD)

if(CLR_CMAKE_TARGET_WIN32)
  set(JIT_RESOURCES Native.rc)
endif(CLR_CMAKE_TARGET_WIN32)

set( JIT_SOURCES
  alloc.cpp
  assertionprop.cpp
  bitset.cpp
  block.cpp
  layout.cpp
  codegencommon.cpp
  codegenlinear.cpp
  compiler.cpp
  copyprop.cpp
  disasm.cpp
  earlyprop.cpp
  ee_il_dll.cpp
  eeinterface.cpp
  emit.cpp
  error.cpp
  fgbasic.cpp
  fgdiagnostic.cpp
  fgehopt.cpp
  fgflow.cpp
  fginline.cpp
  fgopt.cpp
  fgprofile.cpp
  fgstmt.cpp
  flowgraph.cpp
  gcdecode.cpp
  gcencode.cpp
  gcinfo.cpp
  gentree.cpp
  gschecks.cpp
  hashbv.cpp
  hwintrinsic.cpp
  hostallocator.cpp
  indirectcalltransformer.cpp
  importer.cpp
  inline.cpp
  inlinepolicy.cpp
  instr.cpp
  jitconfig.cpp
  jiteh.cpp
  jithashtable.cpp
  jittelemetry.cpp
  lclmorph.cpp
  lclvars.cpp
  lir.cpp
  liveness.cpp
  loopcloning.cpp
  lower.cpp
  lsra.cpp
  lsrabuild.cpp
  morph.cpp
  objectalloc.cpp
  optcse.cpp
  optimizer.cpp
  patchpoint.cpp
  phase.cpp
  rangecheck.cpp
  rationalize.cpp
  redundantbranchopts.cpp
  regalloc.cpp
  register_arg_convention.cpp
  regset.cpp
  scopeinfo.cpp
  sideeffects.cpp
  sm.cpp
  smdata.cpp
  smweights.cpp
  ssabuilder.cpp
  ssarenamestate.cpp
  stacklevelsetter.cpp
  treelifeupdater.cpp
  typeinfo.cpp
  unwind.cpp
  utils.cpp
  valuenum.cpp
)

# Add header files to Visual Studio vcxproj, not required for unixes
# change has effect on editor experience and has no impact on build
if (CLR_CMAKE_TARGET_WIN32)
  set( JIT_HEADERS
    ../inc/corinfo.h
    ../inc/corjit.h
    ../inc/corjitflags.h
    ../inc/corjithost.h
    _typeinfo.h
    alloc.h
    arraystack.h
    bitset.h
    layout.h
    bitsetasshortlong.h
    bitsetasuint64.h
    bitsetasuint64inclass.h
    bitsetops.h
    bitvec.h
    block.h
    blockset.h
    codegen.h
    codegeninterface.h
    compiler.h
    compiler.hpp
    compilerbitsettraits.h
    compilerbitsettraits.hpp
    compmemkind.h
    compphases.h
    dataflow.h
    decomposelongs.h
    disasm.h
    emit.h
    emitdef.h
    emitfmts.h
    emitinl.h
    emitjmps.h
    emitpub.h
    error.h
    gentree.h
    gtlist.h
    gtstructs.h
    hashbv.h
    host.h
    hostallocator.h
    hwintrinsic.h
    ICorJitInfo_API_names.h
    ICorJitInfo_API_wrapper.hpp
    inline.h
    inlinepolicy.h
    instr.h
    instrs.h
    jit.h
    jitconfig.h
    jitconfigvalues.h
    jitee.h
    jiteh.h
    jitexpandarray.h
    jitgcinfo.h
    jithashtable.h
    jitpch.h
    jitstd.h
    jittelemetry.h
    lir.h
    loopcloning.h
    loopcloningopts.h
    lower.h
    lsra_reftypes.h
    lsra.h
    namedintrinsiclist.h
    objectalloc.h
    opcode.h
    phase.h
    rangecheck.h
    rationalize.h
    regalloc.h
    register_arg_convention.h
    register.h
    reglist.h
    regset.h
    sideeffects.h
    simd.h
    simdashwintrinsic.h
    simdintrinsiclist.h
    sm.h
    smallhash.h
    smcommon.h
    smopenum.h
    ssabuilder.h
    ssaconfig.h
    ssarenamestate.h
    stacklevelsetter.h
    target.h
    tinyarray.h
    titypes.h
    treelifeupdater.h
    typelist.h
    unwind.h
    utils.h
    valuenum.h
    valuenumfuncs.h
    valuenumtype.h
    varset.h
    vartype.h
  )

  if (CLR_CMAKE_TARGET_ARCH_ARM64 OR CLR_CMAKE_TARGET_ARCH_ARM)
    list (APPEND JIT_HEADERS
      emitarm.h
      emitarm64.h
      emitfmtsarm.h
      emitfmtsarm64.h
      hwintrinsic.h
      hwintrinsiclistarm64.h
      instrsarm.h
      instrsarm64.h
      registerarm.h
      registerarm64.h
      simdashwintrinsiclistarm64.h)
  elseif (CLR_CMAKE_TARGET_ARCH_AMD64 OR CLR_CMAKE_TARGET_ARCH_I386)
    list (APPEND JIT_HEADERS
      emitfmtsxarch.h
      emitxarch.h
      hwintrinsiclistxarch.h
      hwintrinsic.h
      instrsxarch.h
      simdashwintrinsiclistxarch.h)
  endif ()
endif(CLR_CMAKE_TARGET_WIN32)

# Define all the architecture-specific source files

set( JIT_AMD64_SOURCES
  codegenxarch.cpp
  emitxarch.cpp
  lowerxarch.cpp
  lsraxarch.cpp
  simd.cpp
  simdashwintrinsic.cpp
  simdcodegenxarch.cpp
  targetamd64.cpp
  unwindamd64.cpp
  hwintrinsicxarch.cpp
  hwintrinsiccodegenxarch.cpp
)

set( JIT_ARM_SOURCES
  codegenarmarch.cpp
  codegenarm.cpp
  decomposelongs.cpp
  emitarm.cpp
  lowerarmarch.cpp
  lsraarmarch.cpp
  lsraarm.cpp
  targetarm.cpp
  unwindarm.cpp
)

set( JIT_I386_SOURCES
  codegenxarch.cpp
  decomposelongs.cpp
  emitxarch.cpp
  lowerxarch.cpp
  lsraxarch.cpp
  simd.cpp
  simdashwintrinsic.cpp
  simdcodegenxarch.cpp
  targetx86.cpp
  unwindx86.cpp
  hwintrinsicxarch.cpp
  hwintrinsiccodegenxarch.cpp
)

set( JIT_ARM64_SOURCES
  codegenarmarch.cpp
  codegenarm64.cpp
  emitarm64.cpp
  lowerarmarch.cpp
  lsraarmarch.cpp
  lsraarm64.cpp
  simd.cpp
  simdashwintrinsic.cpp
  targetarm64.cpp
  unwindarm.cpp
  unwindarm64.cpp
  hwintrinsicarm64.cpp
  hwintrinsiccodegenarm64.cpp
)

if(CLR_CMAKE_TARGET_ARCH_AMD64)
  set(JIT_ARCH_SOURCES ${JIT_AMD64_SOURCES})
elseif(CLR_CMAKE_TARGET_ARCH_ARM)
  set(JIT_ARCH_SOURCES ${JIT_ARM_SOURCES})
elseif(CLR_CMAKE_TARGET_ARCH_I386)
  set(JIT_ARCH_SOURCES ${JIT_I386_SOURCES})
elseif(CLR_CMAKE_TARGET_ARCH_ARM64)
  set(JIT_ARCH_SOURCES ${JIT_ARM64_SOURCES})
else()
  clr_unknown_arch()
endif()

set(SOURCES
  ${JIT_SOURCES}
  ${JIT_HEADERS}
  ${JIT_RESOURCES}
)

convert_to_absolute_path(SOURCES ${SOURCES})
convert_to_absolute_path(JIT_ARCH_SOURCES ${JIT_ARCH_SOURCES})

# Also convert the per-architecture sources to absolute paths, if the subdirs want to use them.

convert_to_absolute_path(JIT_AMD64_SOURCES ${JIT_AMD64_SOURCES})
convert_to_absolute_path(JIT_ARM_SOURCES ${JIT_ARM_SOURCES})
convert_to_absolute_path(JIT_I386_SOURCES ${JIT_I386_SOURCES})
convert_to_absolute_path(JIT_ARM64_SOURCES ${JIT_ARM64_SOURCES})

set(JIT_DLL_MAIN_FILE ${CMAKE_CURRENT_LIST_DIR}/dllmain.cpp)

if(CLR_CMAKE_TARGET_WIN32)
  set(CLRJIT_EXPORTS ${CMAKE_CURRENT_LIST_DIR}/ClrJit.exports)
  set(JIT_EXPORTS_FILE ${CMAKE_CURRENT_BINARY_DIR}/ClrJit.exports.def)
  preprocess_file (${CLRJIT_EXPORTS} ${JIT_EXPORTS_FILE})

  set(JIT_CORE_SOURCES ${SOURCES})
  set(JIT_DEF_FILE ${JIT_EXPORTS_FILE})
else()
  set(CLRJIT_EXPORTS ${CMAKE_CURRENT_LIST_DIR}/ClrJit.PAL.exports)

  set(JIT_EXPORTS_FILE ${CMAKE_CURRENT_BINARY_DIR}/clrjit.exports)
  generate_exports_file(${CLRJIT_EXPORTS} ${JIT_EXPORTS_FILE})

  if(CLR_CMAKE_TARGET_LINUX OR CLR_CMAKE_TARGET_FREEBSD OR CLR_CMAKE_TARGET_NETBSD OR CLR_CMAKE_TARGET_SUNOS)
    # This is required to force using our own PAL, not one that we are loaded with.
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Xlinker -Bsymbolic")
  endif()

  set_exports_linker_option(${JIT_EXPORTS_FILE})
  set(JIT_EXPORTS_LINKER_OPTION ${EXPORTS_LINKER_OPTION})
  set(JIT_CORE_SOURCES ${SOURCES})
endif()

add_custom_target(jit_exports DEPENDS ${JIT_EXPORTS_FILE})

set(JIT_LINK_LIBRARIES
   utilcodestaticnohost
)

set(JIT_ARCH_LINK_LIBRARIES
   gcinfo
)

if(CLR_CMAKE_HOST_UNIX)
    list(APPEND JIT_LINK_LIBRARIES
       mscorrc
       coreclrpal
       palrt
    )
else()
    list(APPEND JIT_LINK_LIBRARIES
       ${STATIC_MT_CRT_LIB}
       ${STATIC_MT_VCRT_LIB}
       kernel32.lib
       advapi32.lib
       ole32.lib
       oleaut32.lib
       uuid.lib
       user32.lib
       version.lib
       shlwapi.lib
       bcrypt.lib
       crypt32.lib
       RuntimeObject.lib
    )
endif(CLR_CMAKE_HOST_UNIX)

# Shared function for generating JIT
# optional arguments: ADDITIONAL_DESTINATIONS path
function(add_jit jitName)

    set_source_files_properties(${JIT_EXPORTS_FILE} PROPERTIES GENERATED TRUE)

    add_library_clr(${jitName}
        SHARED
        ${JIT_CORE_SOURCES}
        ${JIT_ARCH_SOURCES}
        ${JIT_DEF_FILE}
        ${JIT_DLL_MAIN_FILE}
    )

    if(CLR_CMAKE_TARGET_WIN32)
        target_compile_definitions(${jitName} PRIVATE FX_VER_INTERNALNAME_STR=${jitName}.dll)
    endif(CLR_CMAKE_TARGET_WIN32)

    target_include_directories(${jitName} PRIVATE ${JIT_SOURCE_DIR})
    target_precompile_headers(${jitName} PRIVATE "$<$<COMPILE_LANGUAGE:CXX>:jitpch.h>")

    add_dependencies(${jitName} jit_exports)

    set_property(TARGET ${jitName} APPEND_STRING PROPERTY LINK_FLAGS ${JIT_EXPORTS_LINKER_OPTION})
    set_property(TARGET ${jitName} APPEND_STRING PROPERTY LINK_DEPENDS ${JIT_EXPORTS_FILE})

    target_link_libraries(${jitName}
        ${JIT_LINK_LIBRARIES}
        ${JIT_ARCH_LINK_LIBRARIES}
    )

    # add the install targets
    install_clr(TARGETS ${jitName} ${ARGN})
endfunction()

set(JIT_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})

if (FEATURE_MERGE_JIT_AND_ENGINE)
  add_subdirectory(crossgen)
endif (FEATURE_MERGE_JIT_AND_ENGINE)

# Creates a static library "clrjit_static" to link into the VM.
add_subdirectory(static)

if (CLR_CMAKE_TARGET_OSX AND CLR_CMAKE_TARGET_ARCH_ARM64)
  set(TARGET_OS_NAME unix_osx) # Apple Arm64 has a special ABI, distinguish it.
elseif (CLR_CMAKE_TARGET_UNIX)
  set(TARGET_OS_NAME unix)
else()
  set(TARGET_OS_NAME win)
endif()

create_standalone_jit(TARGET clrjit OS ${TARGET_OS_NAME} ARCH ${ARCH_TARGET_NAME} ADDITIONAL_DESTINATIONS sharedFramework)

# Enable profile guided optimization
add_pgo(clrjit)

if (CLR_CMAKE_BUILD_SUBSET_ALLJITS AND NOT CLR_CROSS_COMPONENTS_BUILD)
  if (CLR_CMAKE_TARGET_ARCH_ARM64 OR CLR_CMAKE_TARGET_ARCH_AMD64)
    create_standalone_jit(TARGET clrjit_unix_arm64_${ARCH_HOST_NAME} OS unix ARCH arm64)
    create_standalone_jit(TARGET clrjit_unix_osx_arm64_${ARCH_HOST_NAME} OS unix_osx ARCH arm64)
    create_standalone_jit(TARGET clrjit_unix_x64_${ARCH_HOST_NAME} OS unix ARCH x64)
    create_standalone_jit(TARGET clrjit_win_arm64_${ARCH_HOST_NAME} OS win ARCH arm64)
    create_standalone_jit(TARGET clrjit_win_x64_${ARCH_HOST_NAME} OS win ARCH x64)
  endif (CLR_CMAKE_TARGET_ARCH_ARM64 OR CLR_CMAKE_TARGET_ARCH_AMD64)

  create_standalone_jit(TARGET clrjit_unix_armel_${ARCH_HOST_NAME} OS unix ARCH armel)
  create_standalone_jit(TARGET clrjit_unix_arm_${ARCH_HOST_NAME} OS unix ARCH arm)
  create_standalone_jit(TARGET clrjit_win_arm_${ARCH_HOST_NAME} OS win ARCH arm)
  create_standalone_jit(TARGET clrjit_win_x86_${ARCH_HOST_NAME} OS win ARCH x86)
else()
  if (CLR_CMAKE_TARGET_UNIX)
    create_standalone_jit(TARGET clrjit_unix_${ARCH_TARGET_NAME}_${ARCH_HOST_NAME} OS unix ARCH ${ARCH_TARGET_NAME})
  endif(CLR_CMAKE_TARGET_UNIX)
endif (CLR_CMAKE_BUILD_SUBSET_ALLJITS AND NOT CLR_CROSS_COMPONENTS_BUILD)


if (CLR_CMAKE_TARGET_WIN32 AND CLR_CMAKE_PGO_INSTRUMENT)
  # Copy PGO dependency to target dir
  set(PGORT_DLL "pgort140.dll")
  find_path(PGORT_DIR ${PGORT_DLL} REQUIRED)
  _install(FILES "${PGORT_DIR}/${PGORT_DLL}" DESTINATION ${CMAKE_INSTALL_PREFIX})
  _install(FILES "${PGORT_DIR}/${PGORT_DLL}" DESTINATION ${CMAKE_INSTALL_PREFIX}/sharedFramework)
endif ()