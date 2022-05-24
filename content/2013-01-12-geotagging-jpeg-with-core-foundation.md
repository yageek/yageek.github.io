+++
title="Geotagging JPEG with Core Foundation"
date="2013-01-12"
[taxonomies]
tags = ["development", "help", "foundation"]
categories = ["cocoa", "apple"]
+++

I was searching a way to edit or add GPS metadata of JPEG files without using an external library inside a Mac Application. After looking on internet and reading the ImageIOKit documentation, I found an interesting [post on the Apple Mailing List]("http://lists.apple.com/archives/aperture-dev/2008/Jul/msg00000.html) talking about this topic and an old bug related to it. It includes a code sample that is apparently working now (Mac Os X 10.8).

I just wanted to add some details about it:

-   The values corresponding to the `kCGImagePropertyGPSLatitudeRef`/`kCGImagePropertyGPSLongitudedRef` keys are expected to be of type `NSString`.

-   The values corresponding to the `kCGImagePropertyGPSLatitude/kCGImagePropertyGPSLongitude` keys are expected to be of type `NSNumber`.


```objective-c
 CGImageSourceRef originalImageSource = CGImageSourceCreateWithURL((CFURLRef)imageURL, NULL);  
  //get gps metadata  
  NSDictionary *metadata = (NSDictionary *)CGImageSourceCopyPropertiesAtIndex(originalImageSource,0,NULL);  
  //make the metadata dictionary mutable so we can add properties to it  
  NSMutableDictionary *metadataAsMutable = [[metadata mutableCopy]autorelease];  
  [metadata release];  
  //get mutable gps data  
  NSMutableDictionary *GPSDictionary = [[[metadataAsMutable objectForKey:(NSString *)kCGImagePropertyGPSDictionary]mutableCopy]autorelease];  
  if(!GPSDictionary)  
  {  
     GPSDictionary = [NSMutableDictionary dictionary];  
   }  
  //Upate latitude longitude
  // latRef and lonRef are NSString
    [GPSDictionary setObject:[NSNumber numberWithFloat:lat] forKey:(NSString*)kCGImagePropertyGPSLatitude];
    [GPSDictionary setObject:latRef forKey:(NSString*)kCGImagePropertyGPSLatitudeRef];
      
    [GPSDictionary setObject:[NSNumber numberWithFloat:lon] forKey:(NSString*)kCGImagePropertyGPSLongitude];
    [GPSDictionary setObject:lonRef forKey:(NSString*)kCGImagePropertyGPSLongitudeRef];
 
  //add our modified GPS data back into the image's metadata  
  [metadataAsMutable setObject:GPSDictionary forKey:(NSString *)kCGImagePropertyGPSDictionary];  
  NSLog(@"Updated metadata: %@", metadataAsMutable);  
   // Create a new image source that reads the TIFF data that we get from our NSImage.  
   NSData *imageData = [newImage TIFFRepresentation];  
   CGImageSourceRef finalImageSource = CGImageSourceCreateWithData((CFDataRef)imageData, NULL);  
   // Create an image destination. This will take the image data from our source, and write it along with the metadata we read above  
   // into a file in the correct format.  
   CGImageDestinationRef imageDestination = CGImageDestinationCreateWithURL((CFURLRef)imageURL, imageType, 1, NULL);  
   CGImageDestinationAddImageFromSource(imageDestination, finalImageSource, 0, (CFDictionaryRef)metadataAsMutable);  
   if (!CGImageDestinationFinalize(imageDestination))  
   {  
         // The sample code doesn't do any specific error handling in this case. This would be an appropriate  
         // place to notify the user, put up an alert, etc.  
   }  
   ```