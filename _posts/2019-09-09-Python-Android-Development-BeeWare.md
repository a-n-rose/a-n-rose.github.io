---
layout: post
title: "Python Android Development: Trying out Beeware Tutorial"
date: 2019-09-09
---

I am working my way through this <a href='https://towardsdatascience.com/tools-to-run-python-on-android-9060663972b4'>list of tools</a> for building Android applications with Python programs. 

I am trying out this <a href='https://briefcase.readthedocs.io/en/latest/tutorial/getting-started.html'>Getting Started</a> guide, followed by this <a href='https://briefcase.readthedocs.io/en/latest/tutorial/tutorial-0.html'>tutorial</a> for Beeware and have achieved 67% success in the tutorial. So.. fail thus far.

The latest error for my 'helloworld' project:

```
FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':transformClassesWithDexBuilderForDebug'.
> com.android.build.api.transform.TransformException: com.android.builder.dexing.DexArchiveBuilderException: com.android.builder.dexing.DexArchiveBuilderException: 
Failed to process /home/<user>/.../helloworld/android/libs/python-android-support.jar
```

To get this far I overcame(?) the following issues in aims of following the guide:

## Couldn't get sdkmanager to update or run emulator commands

These commands didn't work:
```
$ sdkmanager --update
$ emulator -list-avds
```

### What helped:

I Created a development environment bash script for my Linux os which I ran each time I used AndroidStudio (upon restart these get erased):

script name: setup_dev_env.sh
```
export PATH=$PATH:/home/<user>/Android/Sdk/tools/
export PATH=$PATH:/home/<user>/Android/Sdk/platform-tools/
export PATH=$PATH:/home/<user>/Android/Sdk/tools/bin
export ANDROID_SDK_HOME=$PATH:/home/<user>/Android/Sdk/
export ANDROID_HOME=$PATH:/home/<user>/Android/
```
I kept this script in my project directory and ran it so:

```
$ source setup_dev_env.sh
```
## Could not run emulator

Even though I could then find out which devices had been created for the emulator to emulate (by creating them in the AVD manager in AnroidStudio), I could not run this command:
```
$ emulator @<device_name>
```
### What helped:

In order to get this to work, I had to change directories to:
```
$ cd /home/<user>/Android/Sdk/tools/ 
$ ./emulator @<device_name>
```
Note: I have a Linux machine, therefore I need './' before 'emulator'.

## AndroidStudio: user does not have access to /dev/kvm

When using AndroidStudio and creating virtual devices, my user didn't have access to the AVD in AndroidStudio.

### What helped:

This <a href="https://blog.chirathr.com/android/ubuntu/2018/08/13/fix-avd-error-ubuntu-18-04/">resource</a> helped deal with that problem:

```
$ sudo apt install qemu-kvm
$ sudo adduser <user> kvm
$ sudo chown <user> /dev/kvm
```

If you have Linux, you can find out your user/username by typing:

```
$ whoami
```

## ERROR: toga-gtk 0.3.0.dev14 has requirement toga-core==0.3.0.dev14,
## but you'll have toga-core 0.3.0.dev11 which is incompatible

### What helped:

Nothing. Ignoring it I guess. 

When running the Beeware tutorial, I kept getting this error, even though I had 'toga-core==0.3.0.dev14' installed. I just ignore it.


## AndroidStudio couldn't find 'local.properties'. 

### What helped:

I had to copy the local.properties file from AndroidStudioProjects to where my local project (titled 'helloworld') was:
```
$ cp /home/<user>/AndroidStudioProjects/MyFirstApp/local.properties /home/<user>/.../helloworld/android/
```

## More errors... 

I got a couple more errors and I can't remember what they were. They weren't self-explanatory though. 

### What helped:

Make adjustments to the gradle.build script in my project directory:

'/home/ <user> /.../helloworld/android/build.gradle'

I changed the 'buildscript' item and added the 'allprojects' item:
```
buildscript {
    repositories {
        jcenter()
        maven { url = "https://plugins.gradle.org/m2/" }
        maven { url  = 'https://maven.google.com' }
        mavenCentral()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:3.2.0-alpha12'  // updated plugin version
    }
}

allprojects {
    repositories {
        jcenter()
        maven {
            url "https://maven.google.com"
        }
    }
}
```

# Current State: Annoyed.

I am currently getting through 67% of the build and now getting an error about python-android-support.jar. I'm a bit sick of this though. 

```
FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':transformClassesWithDexBuilderForDebug'.
> com.android.build.api.transform.TransformException: com.android.builder.dexing.DexArchiveBuilderException: com.android.builder.dexing.DexArchiveBuilderException: 
Failed to process /home/airos/Desktop/github/a-n-rose/app-development/beeware/helloworld/android/libs/python-android-support.jar
```

If I get any further I'll update this post. 
