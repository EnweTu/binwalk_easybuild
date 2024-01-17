# Binwalk_easybuild

本项的目标是解决在较新的Linux发行版（例如 Debian 12）上安装binwalk的依赖（例如 sasquatch）时报错的问题。

第一个错误类似下面这样:

```
cc -g -O2  -I. -I./LZMA/lzma465/C -I./LZMA/lzmalt -I./LZMA/lzmadaptive/C/7zip/Compress/LZMA_Lib -D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE -D_GNU_SOURCE -DCOMP_DEFAULT=\"gzip\" -Wall -Werror  -DGZIP_SUPPORT -DLZMA_SUPPORT -DXZ_SUPPORT -DLZO_SUPPORT -DXATTR_SUPPORT -DXATTR_DEFAULT   -c -o unsquashfs.o unsquashfs.c
unsquashfs.c: In function ‘read_super’:
unsquashfs.c:1835:5: error: this ‘if’ clause does not guard... [-Werror=misleading-indentation]
 1835 |     if(swap)
      |     ^~
unsquashfs.c:1841:9: note: ...this statement, but the latter is misleadingly indented as if it were guarded by the ‘if’
 1841 |         read_fs_bytes(fd, SQUASHFS_START, sizeof(struct squashfs_super_block),
      |         ^~~~~~~~~~~~~
cc1: all warnings being treated as errors
make: *** [<builtin>: unsquashfs.o] Error 1
```

关于这个问题，在[这里](https://github.com/devttys0/sasquatch/issues/48)有讨论很多解决方案。但是我认为，问题的本质是编译时使用了`-Wall`选项，将所有的警告都转为了错误，导致编译中断。所以，我的解决方案是去掉`-Wall`选项。

第二个错误类似下面这样：


```
g++ -O3 -Wall -c  -I ../../../ ZLib.cpp
In file included from ../LZMA/LZMADecoder.h:6,
                 from ZLib.cpp:53:
../LZMA/LZMADecoder.h: In member function ‘virtual ULONG NCompress::NLZMA::CDecoder::Release()’:
../LZMA/../../../Common/MyCom.h:159:32: warning: this ‘if’ clause does not guard... [-Wmisleading-indentation]
  159 | STDMETHOD_(ULONG, Release)() { if (--__m_RefCount != 0)  \
      |                                ^~
../LZMA/../../../Common/MyCom.h:166:3: note: in expansion of macro ‘MY_ADDREF_RELEASE’
  166 |   MY_ADDREF_RELEASE
      |   ^~~~~~~~~~~~~~~~~
../LZMA/../../../Common/MyCom.h:173:28: note: in expansion of macro ‘MY_UNKNOWN_IMP_SPEC’
  173 | #define MY_UNKNOWN_IMP1(i) MY_UNKNOWN_IMP_SPEC( \
      |                            ^~~~~~~~~~~~~~~~~~~
../LZMA/LZMADecoder.h:196:3: note: in expansion of macro ‘MY_UNKNOWN_IMP1’
  196 |   MY_UNKNOWN_IMP1(
      |   ^~~~~~~~~~~~~~~
../LZMA/../../../Common/MyCom.h:160:24: note: ...this statement, but the latter is misleadingly indented as if it were guarded by the ‘if’
  160 |   return __m_RefCount; delete this; return 0; }
```

这个问题与第一个问题一样，也是通过去掉`-Wall`选项解决。


第三个错误类似下面这样：

```
g++   ./LZMA/lzmalt/*.o unsquashfs.o unsquash-1.o unsquash-2.o unsquash-3.o unsquash-4.o swap.o compressor.o unsquashfs_info.o gzip_wrapper.o lzma_wrapper.o ./LZMA/lzma465/C/Alloc.o ./LZMA/lzma465/C/LzFind.o ./LZMA/lzma465/C/LzmaDec.o ./LZMA/lzma465/C/LzmaEnc.o ./LZMA/lzma465/C/LzmaLib.o xz_wrapper.o lzo_wrapper.o read_xattrs.o unsquashfs_xattr.o -lpthread -lm -lz -L./LZMA/lzmadaptive/C/7zip/Compress/LZMA_Lib -llzmalib  -llzma  -llzo2 -o sasquatch
/usr/bin/ld: unsquash-1.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: multiple definition of `verbose'; unsquashfs.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: first defined here
/usr/bin/ld: unsquash-2.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: multiple definition of `verbose'; unsquashfs.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: first defined here
/usr/bin/ld: unsquash-3.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: multiple definition of `verbose'; unsquashfs.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: first defined here
/usr/bin/ld: unsquash-4.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: multiple definition of `verbose'; unsquashfs.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: first defined here
/usr/bin/ld: compressor.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: multiple definition of `verbose'; unsquashfs.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: first defined here
/usr/bin/ld: unsquashfs_info.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: multiple definition of `verbose'; unsquashfs.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: first defined here
/usr/bin/ld: lzma_wrapper.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: multiple definition of `verbose'; unsquashfs.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: first defined here
/usr/bin/ld: read_xattrs.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: multiple definition of `verbose'; unsquashfs.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: first defined here
/usr/bin/ld: unsquashfs_xattr.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: multiple definition of `verbose'; unsquashfs.o:/tmp/sasquatch/squashfs4.3/squashfs-tools/error.h:34: first defined here
collect2: error: ld returned 1 exit status
make: *** [Makefile:298: sasquatch] Error 1
```

这个问题在[这里](https://github.com/devttys0/sasquatch/issues/36)有解释和解决方案，我这里采用了[@mzpqnxow](https://github.com/devttys0/sasquatch/pull/46)的方法。