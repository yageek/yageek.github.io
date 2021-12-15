---

title: How to automate the installation of HTTP certificates on XCode 7
date: "2016-02-05"
comments: true
published: true
tags: 
    - ios
    - certificates
    - iostrust 
    - simcert
    - swift
---

Few months ago, I wrote a Ruby gem named [iostrust](https://github.com/yageek/iostrust) to automate the installation of trusted certificate on the different iOS simulators.

This is really helpful when you have a development process that includes continuous integration.  
In our case, we perform some tests with [calabash](https://calaba.sh) each time someone commits to git and before starting the tests, we reset the simulator. As we are using HTTP pinning, we need to find a way to install automatically the certificates within the simulators.

*iostrust* does not work with XCode 7 anymore. I spent some time to find a solution.

I first wanted to follow the same strategy (see [previous post]({% post_url 2015-04-26-certificate-pinning-ios-simulators %})) but I seemed to be really complex.

I noticed then that using the *Accessibility API* on OSX was giving access to the UI elements of the simulator and that it was possible to interact with them.

With this remarks plus some commands to launch/close the simulator application,  I wrote a small Swift program named **simcert** that should solve the problem on XCode 7.

![example](https://media.githubusercontent.com/media/yageek/simcert/develop/sim_cert.gif)

One .pkg file and some instructions are available on [GitHub](https://github.com/yageek/simcert)
