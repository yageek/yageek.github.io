---

title: "Geotagging JPEG with Core Foundation"
date: 2013-01-12
comments: true
categories:
 - jpeg
 - gps
 - cocoa
 - core foundation
---

<div class='post'>
I was searching a way to edit or add GPS metadata of JPEG files without using an external library inside a Mac Application. After looking on internet and reading the ImageIOKit documentation, I found an interesting <a href="http://lists.apple.com/archives/aperture-dev/2008/Jul/msg00000.html" target="_blank">post on the Apple Mailing List</a> talking about this topic and an old bug related to it. It includes a code sample that is apparently working now (Mac Os X 10.8). &nbsp;I just wanted to add some details about it :<br /><div><ul><li>The values corresponding to the kCGImagePropertyGPSLatitudeRef/kCGImagePropertyGPSLongitudedRef keys are expected to be of type NSString.</li><li>The values corresponding to the kCGImagePropertyGPSLatitude/kCGImagePropertyGPSLongitude keys are expected to be of type NSNumber.</li><script src="https://gist.github.com/4518092.js"></script> </ul></div></div>
