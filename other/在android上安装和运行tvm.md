**文章目录**

[](#安装-TVM4J-Java-Frontend-for-TVM-Runtime "安装 TVM4J - Java Frontend for TVM Runtime")安装 TVM4J - Java Frontend for TVM Runtime
------------------------------------------------------------------------------------------------------------------------------

### [](#安装依赖环境 "安装依赖环境")安装依赖环境

JDK 1.6+，maven 3，LLVM(已安装)

#### [](#安装openjdk "安装openjdk")安装openjdk

<table><tbody><tr><td class="gutter"><pre>1<br></pre></td><td class="code"><pre>apt install openjdk-8-jdk<br></pre></td></tr></tbody></table>

#### [](#安装maven-3 "安装maven 3")安装maven 3

<table><tbody><tr><td class="gutter"><pre>1<br>2<br>3<br></pre></td><td class="code"><pre>wget http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.1/binaries/apache-maven-3.6.1-bin.tar.gz<br>mv apache-maven-3.6.1-bin.tar.gz /usr/local/<br>tar -zxvf apache-maven-3.6.1-bin.tar.gz<br></pre></td></tr></tbody></table>

编辑/etc/profile文件，添加如下内容

<table><tbody><tr><td class="gutter"><pre>1<br>2<br></pre></td><td class="code"><pre>export M2_HOME=/usr/local/apache-maven-3.6.1<br>export PATH=${M2_HOME}/bin:$PATH<br></pre></td></tr></tbody></table>

执行source /etc/profile使之生效。

### [](#安装TVM4J "安装TVM4J")安装TVM4J

<table><tbody><tr><td class="gutter"><pre>1<br>2<br>3<br></pre></td><td class="code"><pre>cd tvm<br>make jvmpkg<br>make jvminstall<br></pre></td></tr></tbody></table>

[](#安装Gradle "安装Gradle")安装Gradle
--------------------------------

<table><tbody><tr><td class="gutter"><pre>1<br>2<br>3<br>4<br>5<br>6<br></pre></td><td class="code"><pre>$ wget https://services.gradle.org/distributions/gradle-5.5.1-bin.zip<br>$ mkdir /opt/gradle<br>$ unzip -d /opt/gradle gradle-5.5.1-bin.zip<br>$ ls /opt/gradle/gradle-5.5.1<br>LICENSE  NOTICE  bin  getting-started.html  init.d  lib  media<br>$ export PATH=$PATH:/opt/gradle/gradle-5.5.1/bin<br></pre></td></tr></tbody></table>

[](#安装Android-Sdk-和-Android-Ndk "安装Android Sdk 和 Android Ndk")安装Android Sdk 和 Android Ndk
-----------------------------------------------------------------------------------------

下载对应的Android Sdk 和 Android Ndk安装包并解压，然后添加环境变量：

<table><tbody><tr><td class="gutter"><pre>1<br>2<br>3<br></pre></td><td class="code"><pre>export ANDROID_NDK=/root/android-ndk-r20<br>export ANDROID_HOME=/root/Android/Sdk<br>export PATH="$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools"<br></pre></td></tr></tbody></table>

[](#生成apk文件并安装到android设备 "生成apk文件并安装到android设备")生成apk文件并安装到android设备
--------------------------------------------------------------------

<table><tbody><tr><td class="gutter"><pre>1<br>2<br>3<br>4<br></pre></td><td class="code"><pre>cd /root/tvm/apps/android_rpc<br>gradle clean build<br>./dev_tools/gen_keystore.sh<br>./dev_tools/sign_apk.sh<br></pre></td></tr></tbody></table>

将android设备连接到电脑，然后执行

<table><tbody><tr><td class="gutter"><pre>1<br></pre></td><td class="code"><pre>$ANDROID_HOME/platform-tools/adb install app/build/outputs/apk/release/tvmrpc-release.apk<br></pre></td></tr></tbody></table>

在手机安装apk文件。

[](#安装交叉编译链 "安装交叉编译链")安装交叉编译链
-----------------------------

<table><tbody><tr><td class="gutter"><pre>1<br>2<br></pre></td><td class="code"><pre>cd /root/android-ndk-r20/build/tools/<br>./make-standalone-toolchain.sh --platform=android-24 --use-llvm --arch=arm64 --install-dir=/opt/android-toolchain-arm64<br></pre></td></tr></tbody></table>

可以通过`adb shell cat /proc/cpuinfo`命令查看android设备的CPU信息。

[](#运行benchmark "运行benchmark")运行benchmark
-----------------------------------------

在本地主机启动RPC Tracker：

<table><tbody><tr><td class="gutter"><pre>1<br></pre></td><td class="code"><pre>python3 -m tvm.exec.rpc_tracker<br></pre></td></tr></tbody></table>

然后在android设备上通过TVM RPC app 注册设备。

在本地主机新开tab查看所有的注册设备：

<table><tbody><tr><td class="gutter"><pre>1<br></pre></td><td class="code"><pre>python3 -m tvm.exec.query_rpc_tracker<br></pre></td></tr></tbody></table>

可以看到如下结果：

<table><tbody><tr><td class="gutter"><pre>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15<br></pre></td><td class="code"><pre>Tracker address 0.0.0.0:9190<br><br>Server List<br>----------------------------<br>server-address  key<br>----------------------------<br>10.136.213.102:34446    server:hikey970<br>----------------------------<br><br>Queue Status<br>--------------------------------<br>key        total  free  pending<br>--------------------------------<br>hikey970   1      0     1      <br>--------------------------------<br></pre></td></tr></tbody></table>

本地主机运行benchmark：

<table><tbody><tr><td class="gutter"><pre>1<br>2<br></pre></td><td class="code"><pre>cd ~/tvm/apps/benchmark<br>python3 arm_cpu_imagenet_bench.py --model mate10pro --rpc-key hikey970<br></pre></td></tr></tbody></table>

测试结果：

<table><tbody><tr><td class="gutter"><pre>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br></pre></td><td class="code"><pre>llvm -device=arm_cpu -model=kirin970 -target=arm64-linux-android -mattr=+neon &lt;class 'tvm.target.Target'&gt;<br>--------------------------------------------------<br>Network Name         Mean Inference Time (std dev)<br>--------------------------------------------------<br>mobilenet            46.32 ms            (5.53 ms)<br>mobilenet_v2         34.35 ms            (0.73 ms)<br>squeezenet_v1.0      59.82 ms            (5.40 ms)<br>squeezenet_v1.1      26.54 ms            (2.96 ms)<br>resnet-18            85.46 ms            (5.10 ms)<br>resnet-50            258.65 ms           (21.30 ms)<br>inception_v3         648.11 ms           (59.48 ms)<br>densenet-121         234.39 ms           (18.83 ms)<br></pre></td></tr></tbody></table>

<table><tbody><tr><td class="gutter"><pre>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br></pre></td><td class="code"><pre>llvm -device=arm_cpu -model=kirin970 -target=arm64-linux-android -mattr=+neon &lt;class 'tvm.target.Target'&gt;<br>--------------------------------------------------<br>Network Name         Mean Inference Time (std dev)<br>--------------------------------------------------<br>mobilenet            58.35 ms            (15.70 ms)<br>mobilenet_v2         36.98 ms            (2.00 ms)<br>squeezenet_v1.0      59.10 ms            (4.40 ms)<br>squeezenet_v1.1      25.90 ms            (0.37 ms)<br>resnet-18            91.61 ms            (2.64 ms)<br>resnet-50            282.38 ms           (24.16 ms)<br>inception_v3         694.44 ms           (41.54 ms)<br>densenet-121         276.71 ms           (9.20 ms)<br></pre></td></tr></tbody></table>

官方给出的运行结果：

|  | densenet-121 | inception-v3 | mobilenet | mobilenet-v2 | resnet-18 | resnet-50 | squeezenet-v1.0 | squeezenet-v1.1 | vgg-16 | vgg-19 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Raspberry Pi 3B | 610.2 | 2074.2 | 121.8 | 104.8 | 320.0 | 726.0 | 185.1 | 94.0 | 1772.0 | 2119.8 |
| Firefly RK3399 | 336.8 | 1304.4 | 77.9 | 64.8 | 158.6 | 403.2 | 94.3 | 48.2 | 903.5 | 1086.0 |
| Huawei P20 Pro | 179.7 | 444.7 | 41.3 | 33.4 | 77.4 | 232.5 | 51.4 | 26.0 | 486.3 | 729.4 |
| Google Pixel2 | 161.0 | 434.8 | 39.6 | 29.3 | 66.0 | 181.1 | 47.3 | 23.0 | 397.1 | 485.0 |
| Xilinx PYNQ | 2887.0 | 9691.7 | 721.4 | 513.3 | 1231.7 | 3585.5 | 913.0 | 478.3 | -1.0 | -1.0 |