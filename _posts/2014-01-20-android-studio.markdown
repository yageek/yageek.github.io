---

title: "Android Studio, Gradle and Android Annotations"
date: 2014-01-20 22:40:20 +0100
published: true
comments: true
categories:
- Android
- Gradle
- Android Studio
- Android Annotations
---

I switched for some days from iOS to Android to start a new project and refresh a little bit my knowledge about the Android toolchain.

I'm not a first hour user of Java and I have to admit that I do not follow all news about new Java tools or features. I discovered IntelliJ and Gradle only 8 months ago. This was a very good discover and makes me forget old and disapointing experiences with Ant, Eclipse/Netbeans and XML configuration. <troll> I really thought XML was a part of the language</troll>. The announcement of Android Studio at the last Google I/O sounded like a resurrection of some of old projects on my hard drive disk.

Another bad experience with Android was the lot of boilerplate Java you have to write anytime you code GUI stuffs. In French GNU/Linux magazine I discovered the amazing android annotated library android annotations which do a million more than just avoid you to call the *findViewById* method.
After a lot of time reading the Gradle documentation and browsing Stack Overflow topics about this building tool, I succeed in having a working environment with Android Studio, Gradle and AndroidAnnotations on my laptop (OSX 10.9). I'll try to share and explain my build.gradle in this post.

Pay attention to the version of Android Studio, Android SDK tools and Gradle you're using !

Here I am using **Android Studio v0.4.2**, **Android SDK tools 19.0.1** and **Gradle 1.9**.
I notice some problems by using the Gradle version provided with Android Studio. I rather download Gradle somewhere on my hard drive disk and configure Android Studio to use it (Android Studio -> Preferences).

{% highlight groovy%}
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:0.7.+'
    }
}
apply plugin: 'android'

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


def getSourceSetName(variant) {
    return new File(variant.dirName).getName();
}

android.applicationVariants.all { variant ->
    def aptOutputDir = project.file("build/source/apt")
    def aptOutput = new File(aptOutputDir, variant.dirName)
    println "****************************"
    println "variant: ${variant.name}"
    println "manifest:  ${variant.processResources.manifestFile}"
    println "aptOutput:  ${aptOutput}"
    println "****************************"

    android.sourceSets[getSourceSetName(variant)].java.srcDirs+= aptOutput.getPath()

    variant.javaCompile.options.compilerArgs += [
            '-processorpath', configurations.apt.getAsPath(),
            '-AandroidManifestFile=' + variant.processResources.manifestFile,
            '-s', aptOutput
    ]

    variant.javaCompile.source = variant.javaCompile.source.filter { p ->
        return !p.getPath().startsWith(aptOutputDir.getPath())
    }

    variant.javaCompile.doFirst {
        aptOutput.mkdirs()
    }
}
{% endhighlight %}

Let's try some explanations.
{% highlight groovy%}

buildscript {
    repositories {
        mavenCentral()
    }
 
    dependencies {
        classpath 'com.android.tools.build:gradle:0.7.+'
    }
}
apply plugin: 'android'

{% endhighlight %}

Until here no magic, we tell Gradle we need some specifics dependencies to build our project.

{% highlight groovy%}
repositories {
    mavenCentral()
}
configurations {
    apt
}
{% endhighlight %}

The `configuration` statement defines a *dependency group* [^1]. A named *dependency group* allows you to pick some informations later about the dependencies you define in this group. It is reprensented by an object implementing the `Configuration` interface [^2].

This interface inherits a method named `getAsPath` which returns the classpath of the dependencies defined in the group as a String. We'll use this method later to get the classpath of `androidannotation` annotation compiler.

We create a `apt` *dependency group*, to refer to the annotation processor dependencies.

{% highlight groovy%}
dependencies {
	compile 'com.android.support:appcompat-v7:+'
	compile 'org.androidannotations:androidannotations-api:3.0'
	apt 'org.androidannotations:androidannotations:3.0'
} 
{% endhighlight %}

Here we define the dependencies we need in each *dependency groups*. Within the `compile` group we add the libraries we need in our builded program. In the `apt` *dependency groups*, we add the library needed to process the androidannotations' annotations. 

{% highlight groovy%}
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
}{% endhighlight %}

Simply defines the android SDK we want to use and the minimal API version the application supports and the signing parameters.[^3] 

{% highlight groovy%}
def getSourceSetName(variant) {
    return new File(variant.dirName).getName();
}
{% endhighlight %}

We define a simple helper function to retrieve the android source sets variant name.

{% highlight groovy%}
android.applicationVariants.all { variant ->
    def aptOutputDir = project.file("build/source/apt")
    def aptOutput = new File(aptOutputDir, variant.dirName)
    println "****************************"
    println "variant: ${variant.name}"
    println "manifest:  ${variant.processResources.manifestFile}"
    println "aptOutput:  ${aptOutput}"
    println "****************************"
 
    android.sourceSets[getSourceSetName(variant)].java.srcDirs+= aptOutput.getPath()
 
    variant.javaCompile.options.compilerArgs += [
            '-processorpath', configurations.apt.getAsPath(),
            '-AandroidManifestFile=' + variant.processResources.manifestFile,
            '-s', aptOutput
    ]
 
    variant.javaCompile.source = variant.javaCompile.source.filter { p ->
        return !p.getPath().startsWith(aptOutputDir.getPath())
    }
 
    variant.javaCompile.doFirst {
        aptOutput.mkdirs()
    }
}
{% endhighlight %}

Let's divide this in smaller parts:

{% highlight groovy%}
android.applicationVariants.all { variant ->
	...
}
{% endhighlight %}

The changes are applied for all variants (Variant = Build Type + Product Flavor)[^4] of the application.

{% highlight groovy%}
android.applicationVariants.all { variant ->
def aptOutputDir = project.file("build/source/apt")
def aptOutput = new File(aptOutputDir, variant.dirName)

println "****************************"
println "variant: ${variant.name}"
println "manifest:  ${variant.processResources.manifestFile}"
println "aptOutput:  ${aptOutput}"
println "****************************"
{% endhighlight %}

We first define a outputDirectory for the generated sources. He're it is set to "build/source/apt" in the `aptOutput` variable and print some informations to STDOUT.

{% highlight groovy%}
android.sourceSets[getSourceSetName(variant)].java.srcDirs+= aptOutput.getPath()
{% endhighlight %}

We add the generated sources to the current variants sources' scope.

{% highlight groovy%}
variant.javaCompile.options.compilerArgs += [
	'-processorpath', configurations.apt.getAsPath(),
    '-AandroidManifestFile=' + variant.processResources.manifestFile,
    '-s', aptOutput
]
{% endhighlight %}

For each variant, we access to the gradle java compilation task [^5] of each variant and add some arguments to the compiler. Notice the call to `getAsPath` method as explained before to retrieve the classpath of the androidannotation's compiler processor. We also set up, the output of processint to `aptOutput`

{% highlight groovy%}
variant.javaCompile.source = variant.javaCompile.source.filter { p ->
        return !p.getPath().startsWith(aptOutputDir.getPath())
}
{% endhighlight %}

We are filtering the unnecessary files from the task of compiling java's task.

{% highlight groovy%}
variant.javaCompile.doFirst {
       aptOutput.mkdirs()
   }
{% endhighlight %}

Don't forget to create `aptOutput` directory before start the compilation and it's done.

[^1]: [Dependency group on Gradle Documentation](http://www.gradle.org/docs/current/userguide/dependency_management.html#sub:configurations)
[^2]: [Configuration Interface](http://www.gradle.org/docs/current/groovydoc/org/gradle/api/artifacts/Configuration.html)
[^3]: [Signing Configuration](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Signing-Configurations)
[^4]: [Build variants](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Build-Variants). See also the [youtube presentation of the gradle build system for android](http://youtu.be/LCJAgPkpmR0)
[^5]: [JavaCompile class](http://www.gradle.org/docs/current/groovydoc/org/gradle/api/tasks/compile/JavaCompile.html)