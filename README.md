# NFsim v1.11 with vCell changes

## Build
On Linux/macOS:
``` shell
$ cmake -S . -B build
$ cmake --build build --config Release
``` 

On Windows:
``` shell
> cmake -S . -B build -G "Visual Studio 17 2022" -A x64
> cmake --build build --config Release
```

## Test
On Linux/macOS/Windows:
```shell
$> ctest --test-dir build -C Release --output-on-failure
```
Note: `-C <config>` is required for multi-config generators to specify which build configuration to test.

## Clean
To remove build artifacts:
```shell
$ cmake --build build --target clean
```

To completely remove the build directory:
On Linux:
```shell
$ rm -rf build
```
On Windows:
```shell
cmd> rmdir /s /q build
ps1> Remove-Item -Recurse -Force build
```
