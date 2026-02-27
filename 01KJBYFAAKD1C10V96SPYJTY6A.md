---
title: 20211116 - mcsema 编译 和 使用
confluence_page_id: 1573045
created_at: 2021-11-16T07:24:23+00:00
updated_at: 2021-11-17T07:18:17+00:00
---

```
mkdir mcsema-ve
virtualenv -p python3 mcsema-ve/
cd mcsema-ve
source bin/activate
 
export ALL_PROXY=socks5://mgm.ntidc.actionsky.com:10800
git clone --depth 1 --single-branch --branch v4.0.24 https://github.com/lifting-bits/remill.git
git clone --depth 1 --single-branch --branch master https://github.com/lifting-bits/mcsema.git

git clone --branch master https://github.com/lifting-bits/anvill.git
cd anvill && git checkout -b release_bc3183b bc3183b
cd ..
 
export CC="$(which clang-11)"
export CXX="$(which clang++-11)"

export PATH=/opt/cmake-3.22.0-rc2-linux-x86_64/bin/:$PATH
./remill/scripts/build.sh --llvm-version 11 --download-dir ./
pushd remill-build
sudo cmake --build . --target install
popd
 
mkdir anvill-build
pushd anvill-build
ln -s $(pwd)/../remill-build/install/usr/local/lib/cmake/remill/* ../remill-build/
ln -s $(pwd)/../remill-build/install/usr/local/lib/libremill* /lib/
mkdir /include
 
cmake -DVCPKG_ROOT=$(pwd)/../vcpkg_ubuntu-18.04_llvm-11_amd64 ../anvill
sudo cmake --build . --target install
popd
 
# 如使用LLVM-12, 此处会报错: 
/opt/mcsema-ve/anvill/anvill/src/Optimize.cpp:913:17: error: no member named 'createConstantPropagationPass' in namespace 'llvm'
  fpm.add(llvm::createConstantPropagationPass());
 
# 找到LLVM相关变更: https://github.com/llvm/llvm-project/commit/486ed885339d70cd71ee55567282a43cce28d763#diff-0ed76ab85585358d474a3bb51237cfca030ed5455dc273555eeed6c58df1e1b3
 
mkdir mcsema-build
pushd mcsema-build
cmake -DVCPKG_ROOT=$(pwd)/../vcpkg_ubuntu-18.04_llvm-11_amd64 ../mcsema
sudo cmake --build . --target install

pip install ../mcsema/tools
popd
 
``` 

注意事项: 

  1. 在virtualenv环境中使用, 会报错. 需切换出virtualenv环境
  2. 需先使用一次 idat64, 进行 license agreement, 才能继续进行命令

使用

```
mcsema-disass \
    --disassembler "/opt/idapro-7.6/idat64" \
    --arch amd64 \
    --os linux \
    --pie-mode \
    --binary ~/opt/mysql/8.0.25/bin/mysqld \
    --output /tmp/mysqld.cfg \
    --log_file /tmp/log
``` 

返回值为1, 没有有效信息

尝试手工运行anvill, 发现需要升级python3到python 3.8, 并使用IDAPro的切换脚本, 将IDA用的python切换至3.8

从IDA Pro界面上, 加载脚本: /usr/local/lib/python3.8/dist-packages/mcsema_disass-3.1.3.8-py3.8.egg/mcsema_disass/ida7/get_cfg.py, 可以看到具体的报错. 修正python模块的错误. 

再次运行

冲突: mcsema python脚本要求protobuf版本为3.2, 但 IDAPro会报错, 要求升级到3.19

用pip3将protobuf升级到最新版本, 并修改 ../mcsema/tools/setup.py 中的protobuf版本要求, 并重新安装 该python 模块

调整后, mcsema-disass可通过: 

```
mcsema-disass \
    --disassembler "/opt/idapro-7.6/idat64" \
    --arch amd64 \
    --os linux \
    --entrypoint main \
    --pie-mode \
    --rebase 535822336 \
    --binary /tmp/mysqlbinlog \
    --output /tmp/mysqlbinlog.cfg \
    --log_file /tmp/log
``` 

但 mcsema-lift 报错: 

```
mcsema-lift-11.0 \
    --arch amd64 \
    --os linux \
    --cfg /tmp/mysqlbinlog.cfg \
    --output /tmp/mysqlbinlog.bc \
    --explicit_args \
    --merge_segments \
    --name_lifted_sections
 
F1117 04:12:19.301162 17901 Lift.cpp:432] Check failed: native_val->getType() == decl.type (0x62d6b90 vs. 0x62d6b30)
*** Check failure stack trace: ***
    @          0x12ea23c  google::LogMessageFatal::~LogMessageFatal()
    @           0x753921  anvill::StoreNativeValue()
    @           0x62aa04  mcsema::(anonymous namespace)::GetOrCreateCallback()
    @           0x628c0f  mcsema::NativeFunction::Pointer()
    @           0x63ee80  mcsema::OptimizeModule()
    @           0x63c4bf  mcsema::LiftCodeIntoModule()
    @           0x650ee6  main
    @     0x7f5f8d39cbf7  __libc_start_main
    @           0x5ff12a  _start
    @              (nil)  (unknown)
Aborted (core dumped)
``` 

与单独使用anvill时, 报错一致

无法继续

环境信息:

  - 10.186.62.73
  - mcsema: /opt/mcsema-ve
  - anvill: /opt/anvill-2
