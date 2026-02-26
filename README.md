# NFsim v1.11 with vCell changes

## Build
``` shell
$ mkdir build && cd build
$ cmake ..
$ cmake --build . --target NFsim
``` 
## Run tests
Starts with a smoke test, then runs each of the models in the models directory
```shell
$ ctest --ouptut-on-failure
 ```