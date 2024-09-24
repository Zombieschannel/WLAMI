# WLAMI
Windows-Linux-Android-MacOS-iOS SFML project

Project template for Windows, Linux, Android, MacOS and iOS.

Compilation for Windows and Linux can be done with Visual Studio.

Linux compilation is done with WSL in Visual Studio.

Compilation for Android can be done with Android Studio.

Compilation for MacOS can be done from the command line.

Compilation for iOS can be done from Xcode.

## Setup

All external assets (images, audio, fonts) must be placed inside the `assets/` folder.

The `src/` folder is used for `.cpp`, `.h`, and optionally `.rc` or `.icns` files. 

You'll need to specify your SFML directories in CMakeLists.txt.

The `versions/` folder is ignored by Git and can be used for saving previous releases.

The `res/` folder is used by Android Studio for icons and styles.
