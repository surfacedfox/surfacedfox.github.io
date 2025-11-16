+++
title = "Compiling Shaders(93,274) ..."
date = "2024-03-29"
+++
Speed up ShaderCompileWorker by boosting its priority. <!--more-->
***
We all know the dreadful compiling shaders bit. Seems like process priority affects how fast they're compiled, and Unreal defaults to "Below Normal". At the expense of not being able to use your computer for a bit if you set it to High, you can modify your BaseEngine.ini in Engine/Config to set the priority to whatever you want:

```
; Set process priority for ShaderCompileWorker (0 is normal)
WorkerProcessPriority=-1
```
Here, set it to any of the allowed ints to whatever suits your needs: -2 = Low, -1 = Below Normal, 0 = Normal, 1 = Above Normal, 2 = High.

As of UE5, some new settings pertaining to this exist, too. Their comments are pretty self-explanatory:
```
; Core count threshold.  Below this amount will use NumUnusedShaderCompilingThreads.  Above this threshold will use PercentageUnusedShaderCompilingThreads when determining the number of cores to reserve.
ShaderCompilerCoreCountThreshold=12
; Percentage of your available logical cores that will be reserved and NOT used for shader compilation
; 0 means use all your cores to compile Shaders
; 100 means use none of your cores to compile shaders (it will still use 1 core).
PercentageUnusedShaderCompilingThreads=50
```
For a machine with 24 threads, this new default has the side effect of changing worker count from 21 (24 - 3 unused defined by NumUnusedShaderCompilingThreads in BaseEngine.ini) in UE4 to 12 (24 * 50%) in UE5.
You can set PercentageUnusedShaderCompilingThreads=0 but that might starve your machine, or increase ShaderCompilerCoreCountThreshold to a value > to your thread count so you keep 3 threads for editor/game tasks.


Already got a shader compilation job running? No worries! Just run the following command in an elevated cmd (not powershell) to give those ShaderCompileWorkers the high priority treatment they deserve:

```
wmic process where name="ShaderCompileWorker.exe" call setpriority "high priority"
```

![shaders](shaders.jpeg " ")

All that said, more recent versions of Unreal have cut down on shader compilation by a lot. :3