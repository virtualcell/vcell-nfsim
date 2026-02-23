# NFsim v1.11 with vCell changes

## Build
``` shell
$ mkdir build && cd build
$ cmake ..
$ cmake --build . --target NFsim
``` 
## Run smoke tests
```shell
$ cd tests/smoke
$ ./smoke.sh ../../build/bin/NFsim
 ```