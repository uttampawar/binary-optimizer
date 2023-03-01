# Binary-optimizer
We will be using BOLT as a binary optimizer tool. You can read more at [Project BOLT](https://github.com/llvm/llvm-project/tree/main/bolt)

## Build an application
BOLT is currently incompatible with the ```-freorder-blocks-and-partition``` compiler option. Since GCC8 enables this option by default, you have to explicitly disable it by adding ```-fno-reorder-blocks-and-partition``` flag if you are compiling with GCC8 or above.

Note: Binaries should be linked with reloacations. Use ```-Wl,--emit-relocs,-znow``` linker flag with GCC or LLVM toolchain linker.

## Step 1: Collect Profile
The version of perf command used for the following steps has to support -F brstack option. We recommend using perf version 4.5 or later.

It's been recommended to collect approximately 1B instructions as reported by BOLT -dyno-stats option.

### For Applications
```
$ perf record -e cycles:u -j any,u --- <executable> <args>
```

### For a Service (background application)
```
$ perf record -e cycles:u -j any,u -a -o perf.data -- sleep 180
```

## Step 2: Convert profile to BOLT format. Assuming the perf data is collected in a file "perf.data".
```
$ perf2bolt -p perf.data -o perf.fdata <executable>
```

## Step 3: Optimize the binary with BOLT
```
$ llvm-bolt <executable> -o <executable>.bolt -data=perf.fdata -reorder-blocks=ext-tsp -reorder-functions=hfsort -split-functions -split-all-cold -split-eh -dyno-stats
```

## Multiple profiles
NOTE: You can collect multiple profiles (perf.fdata) and merge them using "merge-fdata" tool in the BOLT toolchain.






