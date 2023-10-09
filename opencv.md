```
https://blog.csdn.net/weixin_44750617/article/details/117629960
brew install ant

ant -version

brew install gcc git cmake pkg-config ffmpeg libgphoto2 libav libjpeg libpng libtiff libdc1394

mkdir build

cmake -DBUILD_SHARED_LIBS=OFF -DWITH_IPP=OFF -DCMAKE_INSTALL_PREFIX={/Users/yuankaiqiang/environment/opencv-4.5.3} ../

make -j4

make install
```

