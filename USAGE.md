Source: README.md at https://github.com/HenrikBengtsson/brother-ptouch-label-printer-on-linux/
_These are Henrik Bengtsson's notes on how to use the `ptouch-print` tool. (Dated 2024-05-04)_

# Use Brother P-touch Label Printer on Linux

These are notes on how to use a Brother P-touch D450 ([PT-D450])
from Linux, which is connected via USB.  They are written around the
**[ptouch-print]** command-line tool for communicating with the label
printer.

## Gist

* The label printer is connected to the computer via a USB A-to-Micro
  B cable.

* **ptouch-print** communicates with the printer directly over
  USB. There is no need to install printer drivers (but Ubuntu might
  still do so).

* **ptouch-print** can send a monochrome PNG file to the label
  printer.

* **ptouch-print** can generate a monochrome PNG file from a sequence
  of multi-line plain text, images, and padding command-line options.

* **ptouch-print** can send the sequence directly to the label
  printer, but it's hard to predict what it will look like. Better to
  always output to a PNG for preview.

* **ptouch-print** can handle multiple monochrome PNG files as input,
  e.g. either to be print one after each other or to output a new
  monochrome PNG file. You can add separation by adding `--pad
  <pixels>` between each `--image <file>` option.

* There will always be 20-30 mm of blank tape wasted at the very front
  of each print when using these printers. For the PT-D450, it's 23
  mm. This is because the (manual) cutter is this distance away from
  the print head. It's physically impossible to avoid this and has
  nothing to which printer software you use. To save label tape, try
  to print multiple labels (images) per print.
  
* **ptouch-print** can merge multiple monochrome PNG images into one
  long PNG image, which helps save tape.

* **ptouch-print** needs to build from source, which is
  straightforward if basic compilee tools are already installed.



## Usage

```sh
$ ptouch-print --help
usage: ptouch-print [options] <print-command(s)>
options:
	--debug			enable debug output
	--font <file>		use font <file> or <name>
	--fontsize <size>	Manually set fontsize
	--writepng <file>	instead of printing, write output to png file
print commands:
	--image <file>		print the given image which must be a 2 color
				(black/white) png
	--text <text>		Print 1-4 lines of text.
				If the text contains spaces, use quotation marks
				around it.
	--cutmark		Print a mark where the tape should be cut
	--pad <n>		Add n pixels padding (blank tape)
other commands:
	--version		show version info (required for bug report)
	--info			show info about detected tape
	--list-supported	show printers supported by this version
```



## Supported label printers

```sh
$ ptouch-print --version
ptouch-print version v1.5.r20.ga51fcf9 by Dominic Radermacher

$ ptouch-print --list-supported
Supported printers (some might have quirks)
	PT-2420PC
	PT-2450PC
	PT-1950
	PT-2700
	PT-1230PC
	PT-2430PC
	PT-2730
	PT-H500
	PT-E500
	PT-P700
	PT-P750W
	PT-D410
	PT-D450
	PT-D460BT
	PT-D600
	PT-D610BT
	PT-P710BT
```


## Examples

### Query printer for information

```sh
$ ptouch-print --info
PT-D450 found on USB bus 1, device 8
maximum printing width for this tape is 120px
media type = 01 (Laminated tape)
media width = 18 mm
tape color = 01 (White)
text color = 08 (Black)
error = 0000
```

### Pre-generate PNG image

```sh
$ ptouch-print --text "John Doe" --writepng johndoe-name.png
PT-D450 found on USB bus 1, device 8
choosing font size=94

$ file johndoe-name.png
johndoe.png: PNG image data, 569 x 120, 1-bit colormap, non-interlaced
```

<img src="images/johndoe-name.png" style="border: 1px solid #666"/>


```sh
$ ptouch-print --text "John Doe" "+1 123-456-7890" --writepng johndoe-phone.png
PT-D450 found on USB bus 1, device 8
choosing font size=48

$ file johndoe-phone.png 
johndoe-phone.png: PNG image data, 561 x 120, 1-bit colormap, non-interlaced
```

<img src="images/johndoe-phone.png" style="border: 1px solid #666"/>


```sh
$ ptouch-print --text "John Doe" "john.doe@example.org" "+1 123-456-7890" --writepng johndoe.png
PT-D450 found on USB bus 1, device 11
choosing font size=32

$ file johndoe.png 
johndoe-all.png: PNG image data, 493 x 120, 1-bit colormap, non-interlaced
```

<img src="images/johndoe.png" style="border: 1px solid #666"/>



Note that `ptouch-print` has to be able to communicate with the
printer even if it is only generating a PNG file.


### Print PNG image

```sh
$ ptouch-print --image johndoe.png
PT-D450 found on USB bus 1, device 11
max_pixels=128, offset=4
```

### Print multiple PNG images at once (saves tape)

The label printer will always feed the tape forward after each print
so that the cutter won't cut into the printed content. If you print
many labels, one by one, this way, you end up wasting a lot of label
tape.  To avoid this, you can print multiple labels at the same time.
For example,

```sh
$ ptouch-print --image alice.png --pad 10 --image bob.png --pad 10 --image carol.png
qPT-D450 found on USB bus 1, device 11
max_pixels=128, offset=4
```

## Man pages

<!-- COLUMNS=72 man ptouch-print -->

```sh
PTOUCH-PRINT(1)           ptouch printer util          PTOUCH-PRINT(1)

NAME
       ptouch-print  - a command line tool to use Brother Ptouch label
       printers

SYNOPSIS
       ptouch-print [options] <print command options>

DESCRIPTION
       ptouch-print is a command line tool that can be used  to  print
       labels with several Brother Ptouch label printers.

OPTIONS
       General  options, font selection options and optput options are
       valid for the whole printout and should  be  given  only  once.
       Each duplication will override the previous argument value.

       Any of the printing command options can be given more than once
       in any order, and will be executed in the given order.

   General options
       --debug
              Enables printing of debugging information.

   Font selection options
       --fontsize <size>
              Disable auto-detection of the font size  and  force  the
              font  size  to  the  size given as argument. Please note
              that text will be cut off if you  specify  a  too  large
              font size here.

       --font <fontname>
              Set the font to the fontname given as argument.

   Output control options
       --writepng <file>
              Instead of printing to the label printer, write the out‐
              put in a PNG image file (which can be printed later  us‐
              ing the --image printing command option).

   Printing command options
       --text text
              Print  the  given text at the current position. Text in‐
              cluding spaces must be enclosed in question  marks.   To
              print a text in multiple lines, give multiple text argu‐
              ments.  Also see the EXAMPLES section.

       --image image.png
              Print the image file at the current position. The  image
              file  must  be  a  two color, palette based image in PNG
              format.

       --cutmark
              Print a cutmark (dashed line) at the current position.

       --pad <n>
              Print <n> pixels of white space at the current position.
              Useful  for  printers that would otherwise cut off parts
              of the printed text or image.

   Getting help and display information
       -?, --help
              Print a help message and exit.

       --version
              Display version information and exit.

       --info Show info about the tape detected (like  printing  width
              etc.) and exit.

       --list-supported
              List all supported printers

DEFAULTS
       The default font used is 'DejaVuSans'.

       The  default  fontsize  is auto-detected, depending on the used
       tape width and font.

EXIT STATUS
       0      Successful program execution.

       1      Usage syntax error.

       5      Printer device could not been opened.

EXAMPLES
       ptouch-print --text 'Hello World'
              Print the text 'Hello World' in one line

       ptouch-print --text 'Hello' 'World'
              Print the text 'Hello World' in two  lines  ('Hello'  in
              the first line and 'World' in the second line).

       ptouch-print --image icon.png
              Print   the  image  contained  in  the  PNG  image  file
              'icon.png'.  The image file must be  palette  based  PNG
              format with two colors.

AUTHOR
       Written   by   Dominic   Radermacher  (dominic@familie-raderma‐
       cher.ch).

       Also      see       https://dominic.familie-radermacher.ch/pro‐
       jekte/ptouch-print/

1.5                           2023-03-13               PTOUCH-PRINT(1)
```


## Installation

### Requirements

Installation requirements:

 * Linux
 * Administrative permissions - for setting the [SUID flag] on the built
   `ptouch-print` executable so that any user can run it without `sudo`
 * C compiler, e.g. GCC
 * [CMake] (on Debian/Ubuntu/Pi OS: `sudo apt install cmake`; RedHat/Fedora: `sudo yum install cmake`)
 * [gettext] (on Debian/Ubuntu/Pi OS: `sudo apt install gettext`; RedHat/Fedora: `sudo yum install gettext`)
 * [LibGD] (on Debian/Ubuntu/Pi OS: `sudo apt install libgd-dev`; RedHat/Fedora: `sudo yum install gd-devel`)
 * [libusb] v1.0 (on Debian/Ubuntu/Pi OS: `sudo apt install libusb-1.0-0-dev`; RedHat/Fedora: `sudo yum install libusb-devel`)
 * ...?


Run-time requirements:

 * Linux
 * Non-privileged user rights (requires setting SUID flag)



### Build

```sh
$ git clone https://git.familie-radermacher.ch/linux/ptouch-print.git
$ cd ptouch-print
$ ./build.sh
-- The C compiler identification is GNU 12.2.0
...
[100%] Linking C executable ptouch-print
[100%] Built target ptouch-print

$ ls -l build/ptouch-print 
-rwxr-xr-x 1 pi pi 64328 May  5 06:23 build/ptouch-print

$ build/ptouch-print --version
ptouch-print version v1.5.r20.ga51fcf9 by Dominic Radermacher
```



### Install

#### System-wide installation

Simply invoke make:

```sh
$ sudo make -C build/ install
```

This installs `ptouch-print` centrally on the machine, and loads
[udev] rules to grant access to USB devices, so that `ptouch-print`
can be used without `sudo`.


#### User installation

If you do not want to installed `ptouch-print` system-wide, you can
copy the executable and manual files as the current user in the
desired destination:

```sh
$ prefix=$HOME/software/ptouch-print
$ mkdir -p "$prefix"/{bin,share/man/man1}
$ cp build/ptouch-print "$prefix/bin/"
$ cp ptouch-print.1 "$prefix/share/man/man1"
```

Then install the [udev] rules, and load them to grant users access to
USB devices:

```sh
$ sudo cp udev/90-usb-ptouch-permissions.rules /etc/udev/rules.d/
$ sudo udevadm control --reload-rules
```

_Comment_: You might have to reboot the machine, before this works.


Update `PATH` and `MANPATH` as below, e.g. in `~/.bashrc`.

```sh
$ prefix=$HOME/software/ptouch-print
$ PATH="$prefix/bin:$PATH"
$ MANPATH="$prefix/share/man:$MANPATH"
```

Alternative, if you have [Lmod] set up, you can copy
[modulefiles/ptouch-print/1.5.lua] to your `MODULEPATH` and just do:

```sh
$ module load ptouch-print
```

With this in place, the executable and the manual page should be
found;

```sh
$ command -v ptouch-print
/home/alice/software/ptouch-print/bin/ptouch-print
$ man -w ptouch-print
/home/alice/software/ptouch-print/share/man/man1/ptouch-print.1
```




## Troubleshooting

If you get:

```sh
$ build/ptouch-print --info
PT-D450 found on USB bus 1, device 8
libusb_open error :LIBUSB_ERROR_ACCESS
```

This is due to lack of permissions to access the printer over USB.
The user executing the `ptouch-print` command needs to granted 
access rights, which can be done by installing udev rules as explained above.
An alternative can be done by prefixing the call using `sudo`, e.g. `sudo
build/ptouch-print --info`.
Note: using udev rules is more secure as the binary file may be altered to
perform malicious thing.


## Appendix

### Installation log

```sh
$ cmake --version | head -1
cmake version 3.25.1

CMake suite maintained and supported by Kitware (kitware.com/cmake).

$ $ git log -1 | head -3
commit a51fcf98f879b2505ea002d9b7385ba143c2febc
Author: Dominic Radermacher <dominic@familie-radermacher.ch>
Date:   Thu Apr 18 09:29:38 2024 +0200

$ ./build.sh 
-- The C compiler identification is GNU 12.2.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Found Gettext: /usr/bin/msgmerge (found version "0.21") 
Found GD: YES
-- Looking for gdImagePng in /usr/lib/arm-linux-gnueabihf/libgd.so
-- Looking for gdImagePng in /usr/lib/arm-linux-gnueabihf/libgd.so - found
-- Found ZLIB: /usr/lib/arm-linux-gnueabihf/libz.so (found version "1.2.13") 
-- Found PNG: /usr/lib/arm-linux-gnueabihf/libpng.so (found version "1.6.39") 
-- Looking for gdImageJpeg in /usr/lib/arm-linux-gnueabihf/libgd.so
-- Looking for gdImageJpeg in /usr/lib/arm-linux-gnueabihf/libgd.so - found
-- Found JPEG: /usr/lib/arm-linux-gnueabihf/libjpeg.so (found version "62") 
-- Looking for gdImageGif in /usr/lib/arm-linux-gnueabihf/libgd.so
-- Looking for gdImageGif in /usr/lib/arm-linux-gnueabihf/libgd.so - found
-- Found GD: /usr/lib/arm-linux-gnueabihf/libgd.so
-- Found Git: /usr/bin/git (found version "2.39.2") 
-- Found PkgConfig: /usr/bin/pkg-config (found version "1.8.1") 
-- Found Intl: built in to C library  
-- Checking for module 'libusb-1.0'
--   Found libusb-1.0, version 1.0.26
-- Configuring done
-- Generating done
-- Build files have been written to: /home/pi/repositories/other/ptouch-print/build
-- Found Git: /usr/bin/git (found version "2.39.2") 
[  0%] Built target git-version
[ 33%] Building C object CMakeFiles/ptouch-print.dir/src/libptouch.c.o
In file included from /home/pi/repositories/other/ptouch-print/src/libptouch.c:30:
/home/pi/repositories/other/ptouch-print/src/libptouch.c: In function ‘ptouch_send’:
/home/pi/repositories/other/ptouch-print/include/gettext.h:67:41: warning: format ‘%ld’ expects argument of type ‘long int’, but argument 4 has type ‘size_t’ {aka ‘unsigned int’} [-Wformat=]
   67 | # define gettext(Msgid) ((const char *) (Msgid))
      |                                         ^
/home/pi/repositories/other/ptouch-print/src/libptouch.c:33:14: note: in expansion of macro ‘gettext’
   33 | #define _(s) gettext(s)
      |              ^~~~~~~
/home/pi/repositories/other/ptouch-print/src/libptouch.c:182:33: note: in expansion of macro ‘_’
  182 |                 fprintf(stderr, _("write error: could send only %i of %ld bytes\n"), tx, len);
      |                                 ^
[ 66%] Building C object CMakeFiles/ptouch-print.dir/src/ptouch-print.c.o
/home/pi/repositories/other/ptouch-print/src/ptouch-print.c: In function ‘print_img’:
/home/pi/repositories/other/ptouch-print/src/ptouch-print.c:95:30: warning: format ‘%ld’ expects argument of type ‘long int’, but argument 2 has type ‘size_t’ {aka ‘unsigned int’} [-Wformat=]
   95 |         printf("max_pixels=%ld, offset=%d\n", max_pixels, offset);
      |                            ~~^                ~~~~~~~~~~
      |                              |                |
      |                              long int         size_t {aka unsigned int}
      |                            %d
In file included from /home/pi/repositories/other/ptouch-print/src/ptouch-print.c:32:
/home/pi/repositories/other/ptouch-print/src/ptouch-print.c: In function ‘main’:
/home/pi/repositories/other/ptouch-print/include/gettext.h:88:25: warning: statement with no effect [-Wunused-value]
   88 |     ((void) (Domainname), (const char *) (Dirname))
      |     ~~~~~~~~~~~~~~~~~~~~^~~~~~~~~~~~~~~~~~~~~~~~~~~
/home/pi/repositories/other/ptouch-print/src/ptouch-print.c:512:9: note: in expansion of macro ‘bindtextdomain’
  512 |         bindtextdomain("ptouch-print", "/usr/share/locale/");
      |         ^~~~~~~~~~~~~~
/home/pi/repositories/other/ptouch-print/include/gettext.h:85:34: warning: statement with no effect [-Wunused-value]
   85 | # define textdomain(Domainname) ((const char *) (Domainname))
      |                                 ~^~~~~~~~~~~~~~~~~~~~~~~~~~~~
/home/pi/repositories/other/ptouch-print/src/ptouch-print.c:513:9: note: in expansion of macro ‘textdomain’
  513 |         textdomain("ptouch-print");
      |         ^~~~~~~~~~
[100%] Linking C executable ptouch-print
[100%] Built target ptouch-print
```

### Getting compilation errors?


#### Compilation error: Could not find GD library

If `./build.sh` produces:

```sh
Found GD: NO
CMake Error at cmake/FindGD.cmake:109 (MESSAGE):
  Could not find GD library
```

then make sure to install [LibGD] (see above).


#### Compilation error: No package 'libusb-1.0' found

If `./build.sh` produces:

```sh
-- Checking for module 'libusb-1.0'
--   No package 'libusb-1.0' found

CMake Error at /usr/share/cmake-3.22/Modules/FindPkgConfig.cmake:603 (message):
  A required package was not found
```

then make sure to install [libusb] (see above).


#### Too old cmake version

If you get the following error when compiling:

```sh
CMake Error: The following variables are used in this project, but they are set to NOTFOUND.
Please set them or make sure they are set and tested correctly in the CMake files:
Intl_LIBRARY (ADVANCED)
    linked by target "ptouch-print" in directory /home/alice/repositories/other/ptouch-print

-- Configuring incomplete, errors occurred!
See also "/home/alice/repositories/other/ptouch-print/build/CMakeFiles/CMakeOutput.log".
```

you need to update CMake. I got the above error on Ubuntu 20.04 (April
2020), which comes with cmake 3.16.3. After [installing cmake] 3.25.1,
and wiping the `build/` folder, the compilation succeeded.

As a reference, Ubuntu 22.04 (April 2022) comes with cmake 3.22.1, and
PiOS 12 (November 2023) comes with cmake 3.25.1.

If you have trouble upgrading CMake, then you can always try by
installing version 1.5-r6 from 2022-11-09 that I know compiles with
cmake 3.16.3.  To do this, check out commit
[#71396e8ff1](https://git.familie-radermacher.ch/linux/ptouch-print.git/commit/?id=71396e8ff1f92f70bf67584a9b65315229fedfb6)
and build from that version, i.e.

```sh
$ git clone https://git.familie-radermacher.ch/linux/ptouch-print.git
$ git checkout 71396e8ff1
$ cd ptouch-print
$ ./build.sh
```

and continue to install following the instructions above.  That will
give you:

```r
$ ptouch-print --version
ptouch-print version v1.5-r6-g71396e8 by Dominic Radermacher
```



[PT-D450]: https://www.brother-usa.com/products/ptd450
[ptouch-print]: https://git.familie-radermacher.ch/linux/ptouch-print.git
[CMake]: https://cmake.org/
[installing cmake]: https://cmake.org/download/
[gettext]: https://www.gnu.org/software/gettext/
[LibGD]: https://libgd.github.io/
[libusb]: https://libusb.info/
[Lmod]: https://lmod.readthedocs.io/en/latest/
[modulefiles/ptouch-print/1.5.lua]: modulefiles/ptouch-print/1.5.lua
[SUID flag]: https://en.wikipedia.org/wiki/Setuid
[udev]: https://en.wikipedia.org/wiki/Udev
