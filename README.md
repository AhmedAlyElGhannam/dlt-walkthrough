# DLT Walkthrough

## `dlt-viewer`
### Setup
1. Download the source code of the latest release of `dlt-viewer` from COVESA's organization. The repo can be found [here](https://github.com/COVESA/dlt-viewer).
2. Extract it, navigate to the directory.
3. Install the following dependencies.
  ```bash
  sudo apt install build-essential
  sudo apt install qtcreator
  sudo apt install qtbase5-dev qtchooser qt5-qmake qtbase5-dev-tools
  sudo apt install libqt5serialport5-dev
  ```
4. Use CMake to build and install all the dependencies.
  ```bash
  cmake -S . -B build
  cmake --build build --parallel 12
  cd build
  sudo make install
  ```
5. The binary is found under `build/bin/` and named `dlt-viewer`.

### Create App Icon
1. Create a `.desktop` file with in the path `$HOME/.local/share/applications/` under the name `dlt-viewer.desktop` with the following contents:
  ```
  [Desktop Entry]
  Version=1.0
  Type=Application
  Name=DLT-Viewer
  Comment=Diagnostics & Log Trace Viewer App
  Exec=/home/nemesis/Playground/dlt-viewer-2.28.1/build/bin/dlt-viewer
  Icon=/home/nemesis/Playground/dlt-viewer-2.28.1/src/resources/icon/256x256/org.genivi.DLTViewer.png
  Terminal=false
  Categories=Utility;
  ```
2. Make this file executable using `chmod +x`.
3. Update desktop app database to make it show up immediately.
  ```bash
  update-desktop-database ~/.local/share/applications/
  ```
4. Profit!

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ecf5d647-d7e5-4cae-a75e-e213389b689f" />

## `dlt-daemon`

### Setup
1. Install the following dependencies.
  ```bash
  sudo apt-get install cmake zlib1g-dev libdbus-glib-1-dev build-essential libjson-c-dev
  ```
2. Download the source code of the latest release of `dlt-daemon` from COVESA's organization. The repo can be found [here]([https://github.com/COVESA/dlt-viewer](https://github.com/COVESA/dlt-daemon/tree/master)).
3. Extract the archive, navigate to the directory.
4. Use CMake to build and install all the dependencies.
  ```bash
  cmake -S . -B build -DCMAKE_POLICY_VERSION_MINIMUM=3.5 -DSOCKET_IPC=ON -DSHM=ON
  cmake --build build --parallel 12
  cd build
  sudo make install
  ```
5. The binary is found under `build/src/daemon/` and named `dlt-daemon`.

## Testing DLT Shenanigans
> This is a C++ app that produces dummy logs that are received via `dlt-daemon` through a named FIFO pipe (Shared Memory); which is then showed on `dlt-viewer`.

### C++ App
```cpp
#include <iostream>
#include <thread>
#include <chrono>
#include <cstdlib>
#include <ctime>
#include <dlt/dlt.h>

DLT_DECLARE_CONTEXT(dummyCtx);

int main()
{
    // initialize random seed
    std::srand(static_cast<unsigned int>(std::time(nullptr)));

    // register application and context
    DLT_REGISTER_APP("DMY1", "Dummy Data Generator");
    DLT_REGISTER_CONTEXT(dummyCtx, "DLOG", "Dummy Data Logs");

    for (int i = 0; i < 100; ++i) {

        int temp = 20 + std::rand() % 10;     // temperature 20–30°C
        int speed = std::rand() % 120;        // speed 0–120 km/h
        float voltage = 11.5f + static_cast<float>(std::rand() % 50) / 10.0f; // 11.5–16.5V


        DLT_LOG(dummyCtx, DLT_LOG_INFO,
                DLT_STRING("Dummy telemetry update:"),
                DLT_INT(temp),
                DLT_INT(speed),
                DLT_FLOAT32(voltage));


        DLT_LOG(dummyCtx, DLT_LOG_VERBOSE,
                DLT_STRING("Temp="), DLT_INT(temp),
                DLT_STRING(" Speed="), DLT_INT(speed),
                DLT_STRING(" Voltage="), DLT_FLOAT32(voltage));

        std::this_thread::sleep_for(std::chrono::seconds(1));
    }


    DLT_UNREGISTER_CONTEXT(dummyCtx);
    DLT_UNREGISTER_APP();

    return 0;
}
```

```cmake
cmake_minimum_required(VERSION 3.10)
project(dummy_dlt_sender)

find_package(PkgConfig REQUIRED)
pkg_check_modules(DLT REQUIRED automotive-dlt)

add_executable(dummy_dlt_sender main.cpp)
target_include_directories(dummy_dlt_sender PRIVATE ${DLT_INCLUDE_DIRS})
target_link_libraries(dummy_dlt_sender PRIVATE ${DLT_LIBRARIES})
```

### Running It
* Run `dlt-daemon` as follows:
  ```bash
  sudo dlt-daemon -d -t /tmp
  ```
* Use `chmod` on the created FIFO pipe in order for the app to access it.
  ```bash
  sudo chmod 666 /tmp/dlt
  ```
* Run the app after building it (it does not need special flags for building via CMake).
* Open `dlt-viewer`, press the `Connect all ECU's or create a new one`.
* Choose `TCP`, leave the port as it is since this is the default for `dlt-daemon`, then press `Ok`.
* You should see the logs now!.

### Demo
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/80454e36-b18d-43f7-8bc0-6a49dbb8d702" />
