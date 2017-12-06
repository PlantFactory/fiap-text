## How to use Library

Let's use abundant libraries, which is the best merit of using Arduino.

### Installing the library

Following 3 are the methods to install the library.

* Copy directly to the libraries directory
* Download and install in ZIP format
* Install from library manager

#### Copy directly to the libraries directory

Arduino creates directories that manage sketches and per-user libraries.
`%USER_PROFILE%\Documents\Arduino` for Windows, `~/Arduino` for Linux and `~/Documents/Arduino` for macOS.
(It depends on the setting.) I think that there is a `libraries` directory in that directory, but here is the library installation location.

Create a directory of the library name here, set the header and source of the library, and finish.

#### Download in ZIP format and install

Generally Arduino's library is compressed and distributed in ZIP format. This format can be easily installed from Arduino menu.

Let's install the ADT 74x0 library as an example.

First, download the library. Arduino's library published on Github is often structurally installable as it is. Let's download from Github in ZIP format.
! [] (img / download-from-github.png)

Please select Arduino Sketch → Include Library → Install .ZIP Format Library.
! [] (img / add-zip.png)

Let's select the ZIP file you downloaded earlier.
! [] (img / select-zip.png)

If this message appears, it was successfully installed.
! [] (img / added-library-from-zip.png)

#### Install from library manager

Arduino has a library manager. It is convenient to be automatically notified when there is an update in the library. However, this corresponds to limited libraries. First search the library you want to install, if you did not hit, try ZIP format installation or direct installation.

To open Library Manager, click Sketch → Include Library → Manage Library.
! [] (img / enter-library-manager.png)

When the library manager opens, search the library and click the library you want to install, the version selection and install button will appear. After selecting the version, downloading and installing will start after pressing the install button.
! [] (img / install - from - library - manager.png)

### include library

Including the header file is necessary to use the library in sketching. You can also type `#include ...` manually, but if it is a library recognized by Arduino, you can add it automatically.

To include the library, open Sketch → Include Library. As the list of libraries is displayed, click on the library you want to include.
! [] (img / select-library.png)

Then, `#include ...` of the selected library is appended to the top of the sketch.
! [] (img / library-has-included.png)
