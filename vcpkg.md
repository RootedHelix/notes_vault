Related to [[Package Management]], [[C]]
#package_manager #vcpkg 

## **Install**
```shell
cd ~/projects
=git clone https://github.com/microsoft/vcpkg.git
cd vcpkg
```

**Create a `vcpkg.json` File in the Project Root**
```json
{  
  "name": "ai-json-validator",  
  "version-string": "0.1.0",  
  "dependencies": [  
    "yyjson"  
  ]  
}
```

**Set up Shell so `CMake` can find `vcpkg`**
```shell
export VCPKG_ROOT=~/projects/vcpkg
export PATH=$VCPKG_ROOT:$PATH

./bootstrap-vcpkg.sh -disableMetrics
```

- Note on the first export command that the path to `vcpkg` should be where you cloned the `vcpkg` repo

- Make sure to run the bootsrap with the `-disableMetrics` flags to ensure that no telemetry are collected for Microsoft

**Finally Run**
`cd` to the desires project with the `vcpkg.json` at the root

```shell
vcpkg install 
```

## **`Cmake`**

- Configure with `CMake` so that the proper build files are generated. 

```CMake
find_package(yyjson CONFIG REQUIRED)

target_link_libraries(ai_json_validator PRIVATE yyjson::yyjson)

```

