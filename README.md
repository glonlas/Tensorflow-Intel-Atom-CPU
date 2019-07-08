# TensorFlow 1.14.0, Python 3.7, NO AVX, NO CUDA, Ubuntu 18.04

**Wheel Specs**  

| TF  | OS | Py | SSE3 | SSE4.1 | SSE4.2 | AVX | CPU |
| --- | -- | -- | ---- | ------ | ------ | --- | --- | 
|1.14.1 | Ubuntu 18.04 | 3.7 | ✅ | ✅ | ✅ | ❌ | Intel(R) Atom(TM) CPU C2338 @ 1.74GHz (Silvermont) |

**Download link**  
TBD (compiling in progress)

# How to compile Tensorflow 1.14.0 without AVX for Intel Atom C2338 CPU on Ubuntu 18.04

This page explains how to build from scratch Tensorflow 1.14.0 without AVX.
_From https://www.tensorflow.org/install/source and https://github.com/yaroslavvb/tensorflow-community-wheels_

This package is built on :
- **CPU Type:** `Intel(R) Atom(TM) CPU C2338 @ 1.74GHz`  
To get you CPU Type `grep -m 1 'model name' /proc/cpuinfo`
- **march:** `silvermont`  
To get your march `gcc -march=native -Q --help=target|grep march`


## 0. Prerequisite
Your environment is on Ubuntu 18.04, and Python 3.6 or 3.7 is installed. 

```bash
> python3 --version
Python 3.7.3
```

## 1. Open a screen, Create VirtualEnv and Install Python requirements

We are starting by opening a screen because compiling Tensorflow will take ages. On this dual core Atom processor even after 6 hours it is still compiling. The screen will prevent interuption of the script when you will loose your shell.
```bash
screen -R tensorflow
```
_Note: if you lost your connection, connect again to your host then execute again `screen -R tensorflow`. You will get back to were you stopped_

```bash
apt install build-essential python3.6-dev python3.7-dev 

export tffolder="tensorflow-1.14.0"
mkdir ~/$tffolder && cd ~/$tffolder
python3 -m venv .venv
source .venv/bin/activate

pip install six numpy wheel setuptools mock future>=0.17.1
pip install keras_applications>=1.0.8 --no-deps
pip install keras_preprocessing>=1.0.5 --no-deps
pip install absl-py astor gast google_pasta numpy opt_einsum protobuf tensorboard tensorflow_estimator termcolor wrapt
```

## 2. Install Bazel 0.25.2 or lower (not above)
_From https://docs.bazel.build/versions/master/install-ubuntu.html#install-with-installer-ubuntu_
_The snippet below is for the version 0.27.1, please update it with the current version you will find on their [official repo here](https://github.com/bazelbuild/bazel/releases)_

Note from Bazel: _Please downgrade your bazel installation to version 0.25.2 or lower to build TensorFlow! To downgrade: download the installer for the old version (from https://github.com/bazelbuild/bazel/releases) then run the installer._

```bash
apt-get install pkg-config zip g++ zlib1g-dev unzip wget
export bazelversion="0.25.2"
wget https://github.com/bazelbuild/bazel/releases/download/$bazelversion/bazel-$bazelversion-installer-linux-x86_64.sh
chmod +x bazel-$bazelversion-installer-linux-x86_64.sh
./bazel-$bazelversion-installer-linux-x86_64.sh
```
## 3. Prepare Tensorflow 1.14.0
```bash
git clone https://github.com/tensorflow/tensorflow.git
cd tensorflow
git checkout r1.14
```

## 4. Build the pip package

_This script will help to select what flag you want_
```bash
wget https://raw.githubusercontent.com/yaroslavvb/stuff/master/configure_tf.sh
```
/root/$tffolder/.venv/bin/python
gcc 7.4.0

-march=silvermont -mcx16 -mssse3 -msse4.1 -msse4.2 -mpopcnt -mno-avx

bazel build --config=opt --copt=-march=silvermont --copt=-mcx16 --copt=-mssse3 --copt=-msse4.1 --copt=-msse4.2 --copt=-mpopcnt --copt=-mno-avx -k //tensorflow/tools/pip_package:build_pip_package

### 4.1 Build Tensorflow

```bash
export flags="--config=opt --copt=-march=silvermont --copt=-mcx16 --copt=-mssse3 --copt=-msse4.1 --copt=-msse4.2 --copt=-mpopcnt --copt=-mno-avx -k"
export tag="silvermont"
export date="20190707"
./configure
bazel build $flags //tensorflow/tools/pip_package:build_pip_package
```
_Note: Wait until +5,000 files are compiled_

### 4.2 Build the package

```bash
./bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
mkdir -p ~/$tffolder/tfbins/$date.$tag
cp `find /tmp/tensorflow_pkg -type f ` ~/$tffolder/tfbins/$date.$tag
bazel test $flags //tensorflow/...
bazel test $flags -j 1 //tensorflow/...
bazel build $flags //tensorflow/...
```

### 4.3 Build Wheel file and 
```bash
export wheel=`find ~/$tffolder/tfbins/$date.$tag -type f`
export basename=`find ~/$tffolder/tfbins/$date.$tag -type f -printf "%f\n"`
cd ~/$tffolder/tfbins/$date.$tag
fullname=$date.$tag.$basename
ln -s $basename $fullname
```

### 4.4 Install the pip package
```bash
pip install /tmp/tensorflow_pkg/tensorflow-version-tags.whl
```

## 5. Wrap-up
### 5.1 Deactivate your virtual env
```bash
deactivate
```

### 5.2 remove your compiling files
```bash
cd ~ && rm -R ~/$tffolder
```

### 5.3 Close your screen
```bash 
exit
```