# GCC 17 on M series macos tutorial
0. Install gcc-15, preferably through homebrew. If you are on arm64 (which you should be if you are following this), the directory it installs to should be:
```bash
# G++
/opt/homebrew/bin/g++-15

# GCC
/opt/homebrew/bin/gcc-15
```
Keep note of these if they are different on your system for whatever reason.

---

1. Make a new directory, call it something like `gcc-17-arm64-build`.
Inside there, make a `build`directory.
Should look like this:
```
./gcc-17-arm64-build
└── build
```
---
2. Clone [this repo](https://github.com/iains/gcc-darwin-arm64) INSIDE the (gcc-17-arm64-build)
```bash
cd gcc-17-arm64-build
git clone https://github.com/iains/gcc-darwin-arm64 
```
---
3. Once it finishes downloading, cd into it, and run this script:
```bash
cd gcc-17-arm64-build
./contrib/download_prerequisites 
# you may have to give it exec permissions i cant remember
```
This will download local copies of any prereqs needed. You can also route in your own but I find this more convinient on macOS, as gcc expects `gnu` flavoured versions of all these programs and will cause problems if you dont have those. Theres also version requirements for each avaliable on the gcc build instructions

---
4. go back to the build folder you made before
```bash
cd ../
cd ./build
```
---
5. Begin the configuration stage of the build, with the following script. 
Trust me on the options. I promise. You MUST specify the compilers here, (in the same command) else `gcc`'s build system may get confused and try to run /usr/bin/gcc, which apple has BRILLIANTLY symlinked to apple clangs' binary. 
```bash
CC=/opt/homebrew/bin/gcc-15 \
CXX=/opt/homebrew/bin/g++-15 \
../gcc-darwin-arm64/configure \
--prefix=/opt/gcc-17 \
--with-sysroot=$(xcrun --show-sdk-path) \
--enable-languages=c,c++ \
--disable-multilib \
--enable-checking=release \
--disable-bootstrap

# copy paste the above straight into your terminal 
```
**NOTE:**
if you want to be really safe, and let the compiler bootstrap itself (validating that everything is working correctly), you can remove the `disable-bootstrap` option. In my experience, this takes the build from 10 minutes to 1.5+ hours.

This will hopefully work. If not good luck, try and read the last few lines that bash spat out, and check gcc's build documentation. 

---
6. DO THE BUILD: in the build directory, run `make -j$(nproc)`; this is the actual build phase. Expect it to take a while.
```bash
# from ./build still:

# replace the $(nproc) with the number of physical threads you have if you know that.
# (eg. 10 on my 2021 M1 Macbook Pro)

make -j$(nproc)
# ... build process should take between 10-60 mins, potentially longer
# it took me 1.5 hours the first time with bootstrapping enabled, 
# and the second build was around 8 minutes whilst leaving my M1 MB pro idle
```
---
7. If the build finishes succesfully, congrats, that was the hard part done. Now just run:
```bash
# from ./build still:
# needs sudo for fukin with /opt/bin shit
sudo make install 
```

9. Done. You now have `gcc-17` installed in: `/opt/gcc-17/bin/gcc`.  Add it to your path or make an alias if you want to use it like `gcc-17` or wtv

> [!NOTE]
> This version calls itself gcc17 but in reality its more or less 16. You can also call it 16, by just replacing the prefix in the configuration command
