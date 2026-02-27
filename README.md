# NFsim v1.11 with vCell changes

## Build
``` shell
$ mkdir build && cd build
$ cmake ..
$ cmake --build . --target NFsim
``` 
## Run tests
Starts with a smoke test, then runs each of the models in the models directory.

On Linux/macOS:
```shell
$ ctest --output-on-failure
```

On Windows (using multi-config generators like Visual Studio):
```shell
$ ctest -C Release --output-on-failure
```
Note: `-C <config>` is required for multi-config generators to specify which build configuration to test.