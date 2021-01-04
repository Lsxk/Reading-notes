# macOs编译ceph

## 1.安装依赖
```
brew install llvm
brew install snappy ccache cmake pkg-config nss
pip install cython Sphinx
```
## 2.环境变量设置
```
export PKG_CONFIG_PATH=/usr/local/Cellar/nss/3.60/lib/pkgconfig
export PATH="$HOME/Library/Python/2.7/bin:$PATH"

```
## 3.cmake config
src/CMakeLists.txt
```
#add_subdirectory(lua)
#test_big_endian(CEPH_BIG_ENDIAN)
set (CEPH_BIG_ENDIAN FALSE)
```

## 3.cmake options
```
-DBOOST_J=4
-DCMAKE_C_COMPILER=/usr/local/opt/llvm/bin/clang
-DCMAKE_CXX_COMPILER=/usr/local/opt/llvm/bin/clang++
-DCMAKE_EXE_LINKER_FLAGS="-L/usr/local/opt/llvm/lib"
-DENABLE_GIT_VERSION=OFF
-DSNAPPY_ROOT_DIR=/usr/local/Cellar/snappy/1.1.8
-DWITH_BABELTRACE=OFF
-DWITH_BLUESTORE=OFF
-DWITH_CCACHE=OFF
-DWITH_CEPHFS=OFF
-DWITH_KRBD=OFF
-DWITH_LIBCEPHFS=OFF
-DWITH_LTTNG=OFF
-DWITH_LZ4=OFF
-DWITH_MANPAGE=ON
-DWITH_MGR=OFF
-DWITH_MGR_DASHBOARD_FRONTEND=OFF
-DWITH_RADOSGW=OFF
-DWITH_RDMA=OFF
-DWITH_SPDK=OFF
-DWITH_SYSTEMD=OFF
-DWITH_TESTS=OFF
-DWITH_XFS=OFF
-DWITH_FUSE=OFF
-DWITH_LEVELDB=OFF
-DWITH_PYTHON3=OFF
```
