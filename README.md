[![Build Status](https://travis-ci.org/PX4/jMAVSim.svg?branch=master)](https://travis-ci.org/PX4/jMAVSim)
[![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/DrTon/jMAVSim?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

   Thank the origin authors for such a good platform for simulating!

12.9
    Now the swarm branch can work well I thought, further test is needed though.
    'Cause when datalink in my Px4 Firmware(v1.6.5-rpi_custom branch) is on, as well as my PC simulating 4 quads, they can bring down its network connection.
    
    安装的方法我删了一个，下面的就能用了
    测试的时候我没有关飞机的数据链，然后这样4架飞机就能崩掉我的虚拟机的网络连接，前提是四架飞机都在飞
    
    总体应该是能用了，改变里面src/.../Simulator.java的SWARM_NUM可以改变最大的飞机数量，现在是20
    
    其实方法很简单，就是把数字改成数组
    然后飞机的启动脚本必须一架飞机一个，程序可以共用一个，启动脚本里要改变飞机的MAVLink UDP端口以及飞机的MAV_SYS_ID，如果用了我的数据链也得改启动脚本的数据链，这个可以具体看我的Firmware(v1.6.5-rpi_custom branch)里的posix-configs/STIL/init/ekf2/swarm里面的脚本的区别


Simple multirotor simulator with MAVLink protocol support

### Installation ###

Requirements:
 * Java 6 or newer (JDK, http://www.oracle.com/technetwork/java/javase/downloads/index.html)

 * Java3D and JOGL/JOAL jars, including native libs for Linux (i586/64bit), Windows (i586/64bit) and Mac OS (universal) already included in this repository, no need to install it.

Clone repository and initialize submodules:
```
git clone https://github.com/PX4/jMAVSim.git
git submodule init
git submodule update
```

Install prerequisites via HomeBrew:

```
brew install ant
```

// Tested, method below can funtion well

#### Alternate build / run / distribute

Compile:
```
ant
```

Run:
```
java -cp lib/*:out/production/jmavsim.jar me.drton.jmavsim.Simulator
```

Some shells (e.g. tcsh) will try to expand `*`, so use `\*` instead:
```
java -cp lib/\*:out/production/jmavsim.jar me.drton.jmavsim.Simulator
```

On **Windows** use `;` instead of `:` in -cp:
```
java -cp lib/*;out/production/jmavsim.jar me.drton.jmavsim.Simulator
```


### Troubleshooting ###

#### Java 3D

jMAVSim uses java3d library for visualization.
It was discontinued for long time, but now maintained again and uses JOGL backend.
All necessary jars with java classes and native binaries (Linux/Mac OS/Windows) included in this repo, no need to install java3d manually.
But need to make sure that java doesn't use any other deprecated version of java3d.
For more info related to java3d see this article: https://gouessej.wordpress.com/2012/08/01/java-3d-est-de-retour-java-3d-is-back/

On **Mac OS** java may use deprecated version of java3d as extension, if you get following error:
```
JavaVM WARNING: JAWT_GetAWT must be called after loading a JVM
AWT not found
Exception in thread "main" java.lang.NoClassDefFoundError: apple/awt/CGraphicsDevice
	at javax.media.j3d.GraphicsConfigTemplate3D.<clinit>(GraphicsConfigTemplate3D.java:55)
...
```

Then add `-Djava.ext.dirs=` option to command line when starting:
```
java -Djava.ext.dirs= -cp lib/*:out/production/jmavsim.jar me.drton.jmavsim.Simulator
```

#### Serial port

Serial port access is common problem. Make sure to hardcode correct port in Simulator.java:
```
serialMAVLinkPort.open("/dev/tty.usbmodem1", 230400, 8, 1, 0);
```
(Baudrate for USB ACM ports (that PX4 uses) has no effect, you can use any value)

Usually port is:
```
Mac OS: /dev/tty.usbmodem1
Linux: /dev/ttyACM0
Windows: COM15
```

On **Linux** you may also get `Permission denied` error, add your user to `dialout` group and relogin, or just run as root.

#### UDP

UDP port used to connect ground station, e.g. qgroundcontrol.
jMAVSim in this case works as bridge between ground station and autopilot (behavior can be configured of course).
Make sure that jMAVSim and ground station use the same ports.
In qgroundcontrol (or another GCS) you also need to add target host in UDP port configuration (localhost:14555), so both ends will know to which port they should send UDP packets.

### Development ###

The simulator configuration is hardcoded in file `src/me/drton/jmavsim/Simulator.java`. Critical settings like port names or IP addresses can be provided as commandline arguments.

New vehicle types (e.g. non standard multirotors configurations) can be added very easily.
(But for fixed wing you will need some more aerodynamics knowledge).
See files under `src/me/drton/jmavsim/vehicle/` as examples.

The camera can be placed on any point, including gimabal, that can be controlled by autopilot, see `CameraGimbal2D` class and usage example (commented) in Simulator.java.

Sensors data can be replayed from real flight log, use `LogPlayerSensors` calss for this.

Custom vehicle visual models in .obj format can be used, edit this line:
```
AbstractMulticopter vehicle = new Quadcopter(world, "models/3dr_arducopter_quad_x.obj", "x", 0.33 / 2, 4.0, 0.05, 0.005, gc);
```

Custom MAVLink protocols can be used, no any recompilation needed, just specify XML file instead of `custom.xml` here:
```
MAVLinkSchema schema = new MAVLinkSchema("mavlink/message_definitions/common.xml");
```

It's convinient to start the simulator from IDE. Free and powerful "IntelliJ IDEA" IDE recommended, project files for it are already included, just open project file `jMAVSim.ipr` and right-click -> Run `Simulator`.
