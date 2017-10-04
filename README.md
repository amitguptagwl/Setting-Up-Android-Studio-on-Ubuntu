# Setting-Up-Android-Studio-on-Ubuntu
Setting Up Android Studio and all dependencies on Ubuntu with troubleshooting

I have faced so many issues while installing Android Studio on my Ubuntu 17.04 64-bit machine. I have done many hacks to get it finally worked. Hence sharing all the steps required to setup Android Studio successfully and running a project with that.

## Terminologies

* **AS**- Android Studio. It is an IDE which will help you to make, build, and run your project.
* **AVD**- Android Virtual Device. Consider it as a virtual android mobile running on your machine. You'll require a good graphics support to use it.
* **Emulator**- Requires to run an AVD on your machine.

## Install JDK
  
```
$sudo add-apt-repository ppa:webupd8team/java
$sudo apt-get update
$sudo apt-get install oracle-java7-installer
#You can replace java7 with java8, or java9
$sudo apt-get install oracle-java7-set-default
#Replace java7 with whatever the version you specified in above command
```

**Alert** : I have observed that Android studio faces some issues with java9 currently. So better to install jdk8. Or if you face `NoClassDefFoundError: javax/xml/bind/annotation/XmlSchema` then install java8 instead of java9.

## Install Android Studio

[Download](http://developer.android.com/sdk/index.html) the package. Place it on desire location. You are gonna need it to install Android Studio and run it every time. So better not to leave it in `/tmp` or `~/Downloads` folder. Now Navigate to `android-studio/bin/` in and run Android Studio from terminal using

```
./studio.sh
```

On first run it'll install Android Studio on your machine. 

Alternatively you can install it from command prompt

```
$sudo apt-add-repository ppa:paolorotolo/android-studio 
$sudo apt-get update
$sudo apt-get install android-studio
```

If you don't change the path then the SDK will be installed in `~/Android/Sdk`.

### Update Android Studio

Whenever you open Android Studio, update it first. So you'll have to less problems while running your project. 

1. Go to Configure > Settings > Updates OR File > Settings > Updates
2. Check "Check for updates in channel"
3. Set the channel to Stable Channel unless you're feeling adventurous.
4. Check Now

## Check hardware support for virtualization

Intel's KVM is required for Better AVD Performance. Check if your processor supports hardware virtualization;

```
$egrep -c '(vmx|svm)' /proc/cpuinfo
```

If the output is non-zero, then your machine supports virtualization. Otherwise you'll have to use actual Android device to test your application. Even if the output is 0 , it doesn't mean virtualization is enabled. Let's check it;
```
$sudo apt-get install cpu-checker
$kvm-ok
```
If you see:

```
INFO: Your CPU supports KVM extensions
INFO: /dev/kvm exists
KVM acceleration can be used 
```
Then virtualization is enabled and ready to use otherwise you'll have to enable it from BIOS if you see following;

```
INFO: KVM is disabled by your BIOS
HINT: Enter your BIOS setup and enable Virtualization Technology (VT),
and then hard poweroff/poweron your system
KVM acceleration can NOT be used
```
Recheck the above command once you enable VT from BIOS settings.

Now install **KVM** and add your local user account to the group kvm and libvirtd.

```
$sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils
$sudo adduser your_user_name kvm
$sudo adduser your_user_name libvirtd
```

In Ubuntu 16.10 and above, the group has been renamed to `libvirt` instead of `libvirtd`. In most of the cases, your user name will already be added to `libvirtd`. Above steps will create one more user in your account "Libvirt Qemu".

**Alert** Reboot your machine before going further.

Android Studio Requires `mksdcard` utility to create AVD. If you're running 64-bit Linux OS. It won't have these packages installed by default. Install those packages.
```
$sudo apt-get install libstdc++6:i386 libgcc1:i386 zlib1g:i386 libncurses5:i386
```

Let's verify installation in Terminal using following commans;
```
sudo virsh -c qemu:///system list
```
If you see:

```
Id Name                 State
----------------------------------
```

installation was successful. 

## Install SDK Platforms, tools, system images

Open the android Studio from `android-studio/bin/studio.sh`. Open the SDK manager from tool panel or from File > Settings > Appearance & Behaviour > System Settings > Android SDK. Check following in the below image and install them using "Apply" button.

![Android Studio SDK Manager platform settings](/img/platforms.png?raw=true "Android Studio SDK Manager platform settings")
![Android Studio SDK Manager tools settings](/img/tools.png?raw=true "Android Studio SDK Manager tools settings")

Now open AVD (Android Virtual Device) Manager from the tool palate or from Tools > Android > AVG Manager

If you already added soem device before you'll see the window like this otherwise empty window asking to create your first AVD.

![Android Virtual Device Manager](/img/avd.png?raw=true "Android Virtual Device Manager")

Create Virtual Device. It'll open a window to configure virtual drive. 

![Android Virtual Device Manager Configuration](/img/avd_config.png?raw=true "Android Virtual Device Manager Configuration")

Select the type of device. You can create your own device too. And then click next. It'll ask you to select the image. If you haven't downloaded any image before, you can't go next. I have already downloaded 3 images that you can see in below screen.

![Android Virtual Device Manager System image](/img/avd_sys_img.png?raw=true "Android Virtual Device Manager System image")

Click next > Final. Once selected images are downloaded you can check them in `~\Android\Sdk\system-images` folder. You can check all the AVDs in "~/.android/avd " folder or using following command;

```
~/Android/Sdk/tools/emulator -list-avds
```

## Running Emulator

When you will run your application it'll automatically open the emulator with specified AVD. However to open it from command line;
```
$~/Android/Sdk/tools/emulator @<AVD name>
```

or

```
$export ANDROID_SDK_ROOT=~/Android/Sdk/ && emulator '@'`emulator -list-avds`
```

Once the AVD is started using emulator you can even install an APK on that
```
$~/Android/Sdk/platform-tools/adb install <path_to_your_APK>
```
## Troubleshooting
### Project not found
Android studio makes project folder by default in home directory until you specify another location. Ensure that you are not running android studio from any other user or from root or sudo. Otherwise it'll create the project directory in home folder of particular user.

### Error Installing APKs
If you see following while running your application

```
09:10	Gradle build finished in 468ms
09:11	Session 'MainActivity': Error Installing APKs
```
then go to `File > Settings > Build, Execution, Deployment > Instant Run` and uncheck "Enable Instant Run to ..."

![Android Studio SDK Manager Instant Run settings](/img/instant_run.png?raw=true "Android Studio SDK Manager Instant Run settings")

### Faulty Emulator
If you see following error while running emulator

```
libGL error: unable to load driver: i965_dri.so
libGL error: driver pointer missing
:
:
```
then first check if `/usr/lib64/libstdc++.so.6` presents. If not install following;

```
$sudo apt-get install lib64stdc++6 mesa-utils
```

Now replace with the file comes with Android emulator.

```
$mv ~/Android/Sdk/emulator/lib64/libstdc++/libstdc++.so.6 ~/Android/Sdk/emulator/lib64/libstdc++/libstdc++.so.6.bak
$sudo ln -s /usr/lib64/libstdc++.so.6 ~/Android/Sdk/emulator/lib64/libstdc++
```

### Device offline
Whenever you get "error: device offline" means that connection with emulator & adb bridge has been broken. stops & start adb bridge again with following commands
```
adb kill-server

adb start-server
```

## References:

* https://github.com/uw-it-aca/spacescout-android/wiki/1.-Setting-Up-Android-Studio-on-Ubuntu
* https://stackoverflow.com/questions/4974568/how-do-i-launch-the-android-emulator-from-the-command-line
* https://stackoverflow.com/questions/38450717/session-app-error-while-installing-apk
