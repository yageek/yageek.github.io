---

title: "Certificate Pinning issues with AFNetworking"
date: "2015-04-25"
comments: true
published: true
tags: 
    - pinning
    - ios
    - afnetworking
    - osx
---

The others day at work, we were confronted to a serious bug on the iOS app of a customer.
 Due to some security issues, we used https and a certificate key pinning method to avoid man in the middle attack.

To not reinvent the wheel, we use AFNetworking as our main network stack. This wonderfull API
(thanks [@mattt](https://twitter.com/mattt)), we can use three different
certificate pinning strategies with no effort.

During the development phase, the pinning was working really good out of the box and we 
were very confident for the production :)

Sadly, after releasing the app into production, we noticed a strange issue with the pinning process.

The delegate method `connection:willSendRequestForAuthenticationChallenge:` from the `AFURLConnectionOperation`
was not called everytime, even if the credential storage was not used...

My colleague fell on this [issue](https://github.com/AFNetworking/AFNetworking/issues/991) on Github that refers to an 
[Apple Technical Q&A](https://developer.apple.com/library/ios/qa/qa1727/_index.html). We discover the existence of a
low level cache regarding the TLS connection.

After reading that carefully, the problem was obvious.

We use AFNetworking to connect to our API. We also use an analytics tool that does not use AFNetworking and does not implement any certificate
pinning strategy.

During the development phase, we use a stubbed server that fakes the data of the client. In this environment, both URLS have a different hostname. 
But in production, both URLs have the same hostname, only the paths differ.

When the analytics tool speaks to the endpoint **api.endpoint.com/analytics**, if the certificate is valid, it accepts the certificate but does not "pin" it.
This answer will be cached for 10 minutes for any request reaching the **api.endpoint.com** hostname.

Then when AFNetworking will speak to **api.endpoint.com/api**, `CFNetwork` will use the previous cached response and will not call the 
`connection:willSendRequestForAuthenticationChallenge` and the certificate will not be pinned :(

To fix it, as preconised in the technical Q&A, we changed the analytics endpoint to prevent the use of the cache.

**Morality**: AFNetworking rocks. But are you only using the network through this library ? If not, check that some other component 
will not compromise your certificate pinning strategy.




