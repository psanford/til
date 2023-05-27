# Notes on decompiling a mono/xamarin apk

Extract the apk like you would normally: `apktool d foo.apk`

You'll find the mono dlls in `unknown/assemblies/*` with the primary dll likely named after the apk package.

The dlls are compressed so you need to decompress them before you can decompile them. There's a python script here: https://github.com/x41sec/tools/blob/master/Mobile/Xamarin/Xamarin_XALZ_decompress.py

Download the linux release of https://github.com/icsharpcode/AvaloniaILSpy

Open dll in ilspy

Right click on library and 'Save Code...' to decompile to source.

### Links

- https://www.x41-dsec.de/security/news/working/research/2020/09/22/xamarin-dll-decompression/
