# dlt-viewer-walkthrough

## Setup
1. Download the source code of the latest release of `dlt-viewer` from COVESA's organization. The repo can be found [here]().
2. Extract it, navigate to the directory.
3. Use CMake to build and install all the dependencies.
  ```bash
  cmake -S . -B build
  cmake --build build --parallel 12
  cd build
  sudo make install
  ```
4. The binary is found under `build/bin/` and named `dlt-viewer`.

## Create App Icon
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
