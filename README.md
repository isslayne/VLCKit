# VLCKit
### 编译构建

适用于编译 iOS、tvOS版，macOS未测试，所使用VLCKit版本号为`3.3.17`，VLC版本号为`3.0.16`

1、运行项目`buildMobileVLCKit.sh`,若构建tvOS版本,则执行`buildMobileVLCKit.sh -t -f`编译并构建framework。

2、再次编译或执行自定义修改版本时，可注释掉`buildMobileVLCKit.sh`中该部分内容

```shell
if [ "$NONETWORK" != "yes" ]; then
    if ! [ -e vlc ]; then
        git clone https://code.videolan.org/videolan/vlc.git --branch 3.0.x --single-branch vlc
        info "Applying patches to vlc.git"
        cd vlc
        git checkout -B localBranch ${TESTEDHASH}
        git branch --set-upstream-to=3.0.x localBranch
        git am ${ROOT_DIR}/Resources/MobileVLCKit/patches/*.patch
        if [ $? -ne 0 ]; then
            git am --abort
            info "Applying the patches failed, aborting git-am"
            exit 1
        fi
        cd ..
    else
        cd vlc
        git fetch --all
        git reset --hard ${TESTEDHASH}
        git am ${ROOT_DIR}/Resources/MobileVLCKit/patches/*.patch
        cd ..
    fi
fi
```

3、自定义live555，本项目中所使用live555库修正了部分RTSP直播SDP解析错误问题。

```c++
// /liveMedia/MediaSession.cpp
if ((sscanf(sdpLine, "m=%s %hu RTP/AVP %u",
		mediumName, &subsession->fClientPortNum, &payloadFormat) == 3 ||
    sscanf(sdpLine, "m=%s %hu MP2T/AVP %u",
		mediumName, &subsession->fClientPortNum, &payloadFormat) == 3 ||
	 sscanf(sdpLine, "m=%s %hu/%*u RTP/AVP %u",
		mediumName, &subsession->fClientPortNum, &payloadFormat) == 3)
	&& payloadFormat <= 127) {
      protocolName = "RTP";
    }
```

### 构建问题

1、taglib error

修改tablib/tablib文件夹内的`CMakeLists.txt`

```sh
if(HAVE_BOOST_BYTESWAP OR HAVE_BOOST_ATOMIC OR HAVE_BOOST_ZLIB)
  include_directories(${Boost_INCLUDE_DIR})
endif()

include_directories(${Boost_INCLUDE_DIR}) 修改为
include_directories(${Boost_INCLUDE_DIRS})
```

其他方案（未测试）：

```sh
FIND_PACKAGE(Boost)
IF (Boost_FOUND)
    INCLUDE_DIRECTORIES(${Boost_INCLUDE_DIR})
    ADD_DEFINITIONS( "-DHAS_BOOST" )
ENDIF()

#cmake 会自动设置 BOOST_INCLUDE_DIR, BOOST_LIBRARYDIR 和 BOOST_ROOT

#如何boost 不是安装在默认位置，则需要添加路径， 类似path的做法。 且必须放在FIND_PACKAGE 前面

SET(CMAKE_INCLUDE_PATH ${CMAKE_INCLUDE_PATH} "C:/win32libs/boost")
SET(CMAKE_LIBRARY_PATH ${CMAKE_LIBRARY_PATH} "C:/win32libs/boost/lib")

#或者

set(Boost_ADDITIONAL_VERSIONS "1.55.0" "1.63.0")
# set(Boost_USE_STATIC_LIBS        ON)
set(Boost_USE_MULTITHREADED      ON)
# set(Boost_USE_STATIC_RUNTIME    OFF)
find_package(Boost 1.55.0 REQUIRED system filesystem)
if(Boost_FOUND)
    message(STATUS "boost include path is : ${Boost_INCLUDE_DIRS}")
    message(STATUS "boost library path is : ${Boost_LIBRARY_DIRS}")
    message(STATUS "boost libraries is : ${Boost_LIBRARIES}")
    include_directories(${Boost_INCLUDE_DIRS})
    link_directories(${Boost_LIBRARY_DIRS})
else()
    message(WARNING "boost not found.")
endif()
```



2、编译过程中警告

编译时会出现大量warning信息，不用理会，等待编译完成即可。
