sudo apt-get install autoconf automake

sudo apt-get install libxmu-dev

if not bazel:

    download bazel from https://github.com/bazelbuild/bazel/releases

    chmod +x bazel-<version>-installer-linux-x86_64.sh

    ./bazel-<version>-installer-linux-x86_64.sh --prefix=/usr/local


change
    "//conditions:default": [],
to
    "//conditions:default": [
    "-fvisibility=hidden -fPIC"
    ],
in third_party/jpeg/jpeg.BUILD around line 41 inside 


./tensorflow/contrib/makefile/build_all_linux.sh

bazel clean --expunge

./configure

bazel build --config=opt --config=cuda //tensorflow/tools/pip_package:build_pip_package

cp bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg

sudo pip2 install /tmp/tensorflow_pkg/tensorflow-1.8.0-cp27-cp27mu-linux_x86_64.whl


bazel build -c opt --copt=-mavx --copt=-mavx2 --copt=-mfma --copt=-mfpmath=both --copt=-msse4.2 //tensorflow:libtensorflow_cc.so


# Compilar protobuf <- con la linea de más arriba está compilado en tensorflow/contrib/makefile/gen/protobuf
mkdir /tmp/proto

tensorflow/contrib/makefile/download_dependencies.sh

cd tensorflow/contrib/makefile/downloads/protobuf/

./autogen.sh

./configure --prefix=/tmp/proto

make -j4

make install


# Copiar headers de Eigen

mkdir /tmp/eigen

cd ../eigen

mkdir build

cd build

cmake -DCMAKE_INSTALL_PREFIX=/tmp/eigen/ ../

make install


# Copiar archivos de tensorflow

cd ../../../../../..


sudo cp -rf /tmp/proto/include/* /usr/local/include

sudo cp /tmp/proto/lib/* /usr/local/lib

sudo cp /tmp/proto/lib/pkgconfig/* /usr/local/lib/pkgconfig

sudo cp -rf /tmp/proto/bin/* /usr/local/bin/

sudo cp -rf /tmp/eigen/include/eigen3/* /usr/local/include

sudo cp -rf third_party /usr/local/include

sudo cp -rf bazel-genfiles/tensorflow /usr/local/include

sudo cp -rf tensorflow/c /usr/local/include/tensorflow/

sudo cp -rf tensorflow/cc /usr/local/include/tensorflow/

sudo cp -rf tensorflow/core /usr/local/include/tensorflow/

sudo cp -rf bazel-bin/tensorflow/*so /usr/local/lib


git clone https://github.com/opencv/opencv.git

cd opencv

git tag -l -n2

git checkout tags/3.4.1

mkdir build

cd build

cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local/opencv-3.4 -D WITH_TBB=ON -D BUILD_NEW_PYTHON_SUPPORT=ON -D WITH_V4L=ON -D INSTALL_C_EXAMPLES=OFF -D INSTALL_PYTHON_EXAMPLES=OFF -D BUILD_EXAMPLES=OFF -D WITH_QT=ON -D WITH_OPENGL=ON -D WITH_CUDA=ON -D ENABLE_FAST_MATH=1 -D CUDA_FAST_MATH=1 -D WITH_CUBLAS=1 -D WITH_NVCUVID=ON -D CUDA_GENERATION=Auto -D WITH_IPP=ON -DBUILD_PROTOBUF=OFF PROTOBUF_INCLUDE_DIR=/usr/local/include -DPROTOBUF_LIBRARY=/usr/local/lib/libprotobuf.so -DPROTOBUF_LIBRARY_DEBUG=/usr/local/lib/libprotobuf.so -DPROTOBUF_LITE_LIBRARY=/usr/local/lib/libprotobuf-lite.so -DPROTOBUF_LITE_LIBRARY_DEBUG=/usr/local/lib/libprotobuf-lite.so -DPROTOBUF_PROTOC_EXECUTABLE=/usr/local/bin/protoc -DPROTOBUF_PROTOC_LIBRARY=/usr/local/lib/libprotoc.so -DPROTOBUF_PROTOC_LIBRARY_DEBUG=/usr/local/lib/libprotoc.so -DBUILD_opencv_ts=0 -DBUILD_TESTS=OFF -DBUILD_opencv_dnn=OFF ..

make -j4

sudo make install

sudo cp unix-install/opencv.pc /usr/local/lib/pkgconfig/opencv3.4.pc

pkg-config --cflags --libs /usr/local/lib/pkgconfig/opencv3.4.pc