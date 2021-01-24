---
title: "Handling Hybrid GPU's in Windows"
date: 2019-12-07
draft: false
---

Hybrid GPU's are everywhere these days and yet it is not obvious how to run your program using the discrete GPU.

I can think of 5 ways on how to get this done:

1. Your software is popular enough that the GPU manufacturer includes an "application profile" for your software in their driver. This profile is basically the GPU configuration/settings for an executable. When your software has no application profile, the default graphic card depends on the global GPU settings but it is most likely the integrated gpu.
2. You can manually create an application profile for your executable. The problem is that this profile will need to be manually created for each install. This may be useful if it's a silly program you alone are going to run, but not really an option if you plan to ship software, even if it's to family and friends.
3. Oddly enough you can trick the driver to use an application profile made for other software by simply renaming your executable to something like "Doom3.exe". Not really a solution but it's comforting to know simple tricks like these still exist in 2019.
4. Linking your program against nVidia's defined libraries such as "nvapi64.dll"(The list of libraries is defined in the manual at the bottom of the page). Calling HMODULE Lib =  LoadLibraryA("nvapi64.dll") in main function does the trick. This seems to be the old way of doing things, continue reading for a better solution.
5. The best solution i know is to include the following code at global scope.

```C++
#ifdef _WIN32
extern "C"
{
    // nVidia
    __declspec( dllexport ) unsigned long int NvOptimusEnablement = 0x00000001;
    // AMD
    __declspec( dllexport ) int AmdPowerXpressRequestHighPerformance = 1;
}
#endif
```

I have only tested the code on nVidia Optimus since it's the only thing i have.

If you would like to know more, you should read the following manuals for Hybrid GPU's, they are _really_ _really_ short (4-5 pages).

[nVidia's hybrid gpu manual](http://developer.download.nvidia.com/devzone/devcenter/gamegraphics/files/OptimusRenderingPolicies.pdf)

[AMD's hybrid gpu manual](https://gpuopen.com/amdpowerxpressrequesthighperformance/)
