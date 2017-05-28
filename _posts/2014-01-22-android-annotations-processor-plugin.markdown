---

title: "Android annotations processor plugin"
date: 2014-01-22 16:19:26 +0100
published: true
comments: true
tags: 
- Android
- Gradle
- Android Annotations
- Annotations
---

[The other day]({% post_url 2014-01-20-android-studio %}) I explained how to use Android Annotations with Gradle on Android Studio. There was a lot of Groovy to write before starting to program. I discover the [android-apt](https://bitbucket.org/hvisser/android-apt) plugin for Gradle which reduces my last post to boilerplate code. The *excilys* team advices to use it for [building android annotations with Gradle](https://github.com/excilys/androidannotations/wiki/Building-Project-Gradle).

This plugin does not target *android annotations* specifically but it helps you to work with android and annotation processors and make life easier on Android Studio. 

Starting a new android application with Gradle and *android annotations* is simple:

{% highlight groovy%}
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:0.7.+'
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.2'

    }
}
apply plugin: 'android'
apply plugin: 'android-apt'

repositories {
    mavenCentral()
}

configurations {
    apt
}


dependencies {
    compile 'com.android.support:appcompat-v7:+'
    compile 'org.androidannotations:androidannotations-api:3.0'
	
    apt 'org.androidannotations:androidannotations:3.0'
}

android {
    compileSdkVersion 19
    buildToolsVersion "19.0.1"

    defaultConfig {
        minSdkVersion 14
        targetSdkVersion 19
    }
    buildTypes {
        release {
            runProguard false
            proguardFile getDefaultProguardFile('proguard-android.txt')
        }
    }
    productFlavors {
        defaultFlavor {
            proguardFile 'proguard-rules.txt'
        }
    }
    signingConfigs {
        myConfig {
            storeFile file(project.property("MyProject.signing"))
            storePassword "xxxxxxxxx"
            keyAlias "yageek company"
            keyPassword "xxxxxxxxxx"
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.myConfig
        }
    }
}

apt {
    arguments {
        androidManifestFile variant.processResources.manifestFile
    }
}
{% endhighlight %}