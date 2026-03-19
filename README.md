# NFsim v1.11 with vCell changes

## Build
On Linux/macOS:
``` shell
$ cmake -S . -B build -DNFSIM_ENABLE_MODEL_TESTS=ON
$ cmake --build build --config Release
$ ctest --test-dir build -C Release --output-on-failure
``` 

On Windows:
``` shell
> cmake -S . -B build -G "Visual Studio 17 2022" -A x64 -DNFSIM_ENABLE_MODEL_TESTS=ON
> cmake --build build --config Release
> ctest --test-dir build -C Release --output-on-failure
```

On Linux/macOS/Windows:
```shell
> ctest --test-dir build -C Release --output-on-failure
```
Note: `-C <config>` is required for multi-config generators to specify which build configuration to test.

## Clean
To remove build artifacts:
```shell
$ cmake --build build --target clean
```

To completely remove the build directory:
```shell
$ rm -rf build
```
(On Windows, use `rmdir /s /q build` in Command Prompt or `Remove-Item -Recurse -Force build` in PowerShell).