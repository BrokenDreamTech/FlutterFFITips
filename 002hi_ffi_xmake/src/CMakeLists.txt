cmake_minimum_required(VERSION 3.10)

project(hi_ffi_xmake_library VERSION 0.0.1 LANGUAGES C)

set(HI_LIB_NAME "hi")

if(CMAKE_SYSTEM_NAME MATCHES "OHOS" OR OHOS)
    message(STATUS "当前正在编译: 鸿蒙 (OHOS) 平台")
    # 添加xmake编译的hi
    add_library(hi SHARED IMPORTED)
    # 设置xmake编译的hi的导入规则
    set(XMAKE_OH_BUILD_OUTPUT_SO ${CMAKE_CURRENT_SOURCE_DIR}/xmake/build/harmony/${OHOS_ARCH}/lib${HI_LIB_NAME}.so)

    set_target_properties(hi PROPERTIES
        IMPORTED_NO_SONAME TRUE
        IMPORTED_LOCATION ${XMAKE_OH_BUILD_OUTPUT_SO}
    )

    # 定义鸿蒙标准的 so 存放路径
    set(OHOS_DEFAULT_LIBS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../ohos/libs/${OHOS_ARCH}")
    # 将so从你的 xmake 输出目录拷贝到编译输出目录
    file(COPY ${XMAKE_OH_BUILD_OUTPUT_SO} DESTINATION ${OHOS_DEFAULT_LIBS_DIR})

elseif(ANDROID)
    message(STATUS "当前正在编译: Android 平台")
    # 添加xmake编译的hi
    add_library(hi SHARED IMPORTED)
    # 设置xmake编译的hi的导入规则
    set_target_properties(hi PROPERTIES
        IMPORTED_NO_SONAME TRUE
        IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/xmake/build/android/${ANDROID_ABI}/lib${HI_LIB_NAME}.so
    )

elseif(WIN32)
    message(STATUS "当前正在编译: Windows 平台")

    #判断架构
    if(CMAKE_SIZEOF_VOID_P EQUAL 8)
        set(WIN_ABI_NAME x64)
    else()
        set(WIN_ABI_NAME x86)
    endif()

    # 添加xmake编译的hi
    add_library(hi SHARED IMPORTED)
    # 设置xmake编译的hi的导入规则
    set(XMAKE_WIN_BUILD_OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/xmake/build/windows/${WIN_ABI_NAME})
    set_target_properties(hi PROPERTIES
        IMPORTED_LOCATION ${XMAKE_WIN_BUILD_OUTPUT}/${HI_LIB_NAME}.dll
        IMPORTED_IMPLIB ${XMAKE_WIN_BUILD_OUTPUT}/${HI_LIB_NAME}.lib
        IMPORTED_NO_SONAME TRUE
    )
    # 将DLL 从你的 xmake 输出目录拷贝到编译输出目录
    file(COPY "${XMAKE_WIN_BUILD_OUTPUT}/${HI_LIB_NAME}.dll" DESTINATION "${CMAKE_BINARY_DIR}/runner/Debug")
    file(COPY "${XMAKE_WIN_BUILD_OUTPUT}/${HI_LIB_NAME}.dll" DESTINATION "${CMAKE_BINARY_DIR}/runner/Release")

elseif(CMAKE_SYSTEM_NAME MATCHES "Linux")
    message(STATUS "当前正在编译: Linux 平台")

elseif(APPLE)
    if(CMAKE_SYSTEM_NAME MATCHES "iOS")
        message(STATUS "当前正在编译: iOS 平台")
    else()
        message(STATUS "当前正在编译: macOS 平台")
    endif()

else()
    message(WARNING "未知的编译平台: ${CMAKE_SYSTEM_NAME}")
endif()


add_library(hi_ffi_xmake SHARED
  "hi_ffi_xmake.c"
)

set_target_properties(hi_ffi_xmake PROPERTIES
  PUBLIC_HEADER hi_ffi_xmake.h
  OUTPUT_NAME "hi_ffi_xmake"
)

target_compile_definitions(hi_ffi_xmake PUBLIC DART_SHARED_LIB)

#链接
target_link_libraries(hi_ffi_xmake hi)

if(CMAKE_SYSTEM_NAME MATCHES "OHOS" OR OHOS)

elseif(ANDROID)
  # Support Android 15 16k page size
  target_link_options(hi_ffi_xmake PRIVATE "-Wl,-z,max-page-size=16384")
elseif(WIN32)

elseif(CMAKE_SYSTEM_NAME MATCHES "Linux")

elseif(APPLE)
    if(CMAKE_SYSTEM_NAME MATCHES "iOS")

    else()

    endif()

else()

endif()

