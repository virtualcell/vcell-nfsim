# NFsim v1.11 with vCell changes

## Build
On Linux/macOS:
``` shell
$ mkdir build && cd build
$ cmake ..
$ cmake --build . --target NFsim
``` 

On Windows:
``` shell
> cmake -S . -B build -G "Visual Studio 17 2022" -A x64 -DNFSIM_ENABLE_MODEL_TESTS=ON
> cmake --build build --config Release
> ctest --test-dir build -C Release --output-on-failure
```

On Linux/macOS:
```shell
$ ctest --output-on-failure
```

On Windows (using multi-config generators like Visual Studio):
```shell
> ctest -C Release --output-on-failure
```
Note: `-C <config>` is required for multi-config generators to specify which build configuration to test.