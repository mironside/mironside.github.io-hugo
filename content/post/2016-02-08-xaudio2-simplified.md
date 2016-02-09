+++
date = "2016-02-08T23:12:05-06:00"
title = "XAudio2, Simplified"

+++

XAudio2 versions are a mess across Windows 7, 8 and 10.
https://msdn.microsoft.com/en-us/library/windows/desktop/ee415802.aspx

XAudio2.h has changed and even broken through each version of Windows which makes it non-trivial to use.

- Windows 7
  - XAudio2.h is not included in the Windows 7 Kit, the June 2010 DirectX SDK needs to be installed
  - XAudio2Create is an inline function from the XAudio2 header which creates the XAudio2 instance using COM
  - The June 2010 DirectX SDK Redistributible needs to be installed on the user's machine to ensure the XAudio2 dll is on the user's machine
- Windows 8
  - XAudio2.h is in the Windows 8 Kit
  - Windows 8 includes XAudio2 dlls for all versions of XAudio2, up to XAudio2_8.dll
  - Windows 8 (and up) added a parameter to CreateMasteringVoice and removed a few functions from IXAudio2
- Windows 10
  - XAudio2.h is in the Windows 10 Kit
  - Windows 10 includes XAudio2 dlls for all versions of XAudio2, up to XAudio2_9.dll
  - Compiling the XAudio2 API as C has been broken

Depending on which version of Windows you are compiling __on__ and which version you are compiling __for__ you may need to install and set a specific set of compiler options.

This is not great.

I noticed SDL2 provides it's own XAudio2.h and this seems like a great solution to minimize the configuration necessary to use XAudio2.
http://hg.libsdl.org/SDL/file/c5e3a4f88be9/src/audio/xaudio2/SDL_xaudio2.h

To support Windows 7 and above I only need the XAudio2.7 api from the June 2010 DirectX SDK.  I only use XAudio2 to create a mastering voice and source voice for submitting raw audio data so I can also strip out all the extra api I don't need.

To use the custom header it simply needs to be included.  No paths, no compiler configuration.  The XAudio2Create call is a COM macro so xaudio2.lib doesn't even need to be linked.  The only dependency is that the June 2010 DirectX SDK Redistributible needs to be installed to run on Windows 7 or below.  Windows 8 and up include the correct dlls and will work out of the box.

Below is the stripped XAudio2 header I use for my own projects hardcoded to compile as C.

[xaudio2.h](/static/xaudio2.h)