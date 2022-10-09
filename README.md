# Getting Started with Tomu
My experience learning how to write code for a [tomu](https://tomu.im/).

# MacOS Requirements
Tomu's [quickstart](https://github.com/im-tomu/tomu-quickstart) did not work 
"out of the box" for me. Here's what did work:


## install Arm toolchain
Tomu's instructions lead to a depreciated page arm.com that did not work
despite my best efforts. Using [brew](brew.sh) was easier:
```
brew tap osx-cross/arm
brew install arm-gcc-bin
```

## install make
Make is a component of the xcode package. So the first step is to install xcode.
However, in my sigular experience, this process was not smooth. xcode seemed to
install and reinstall itself a half-dozen times. 

Sometime after that, a MacOS prompted me through authorizing new packages under
the "Security & Privacy" settings page. 

1. Install [xcode](https://apps.apple.com/us/app/xcode/id497799835)
2. Go to "System Preferences" > "Security & Privacy" > "General" tab and there
will be a warning near the bottom of the tab where you have to approve the arm
and intel (?) tooling. 


## install dfu-utils
`dfu-util` will be used to to find the tomu and to upload the compiled program

    brew install dfu-util


# Fleshing out the dependencies (based on bare-minimum)
Tomu's quickstart guide has a [bare-minimum](https://github.com/im-tomu/tomu-quickstart/tree/master/bare-minimum)
project with deceivingly few files. However, its Makefile brings in paths from
parent directories (listed below). The next 3 sections of his guide will
explore them to find our their purpose and where their upstream source is
located (so we can keep them up to date).
- ../libopencm3
- ../include/toboot.h
- ../tomu-efm32hg309.ld


## Source of "libopencm3"
[libopencm3](https://github.com/libopencm3/libopencm3) is its own project. It
can be included in our project as a git submodule. We will also need to `make`
it so that some of files our project needs get created:
```sh
git submodule add https://github.com/libopencm3/libopencm3
make -C libopencm3
```


## Source of "toboot.h"
After doing some digging, I have found that toboot.h is nearly identical to
toboot/toboot-api.h from [im-tomu/toboot](https://github.com/im-tomu/toboot).
However, toboot-api.h has more commits and was last changed more recently. The
minor downside to toboot-api.h compared to toboot.h is that it is missing this
macro below. But, having made a note of that, we can include it in our program
directly.
```c
/// Use this macro to define a Toboot V2 configuration.  If unspecified,
/// your program will default to a legacy configuration, and will not have
/// access to features such as autoboot.
#define TOBOOT_CONFIGURATION(cfg)                                        \
    __attribute__ ((used, section(".vectors")))                          \
    const struct toboot_configuration __toboot_configuration = {         \
        .magic          = TOBOOT_V2_MAGIC,                               \
        .reserved_gen   = 0,                                             \
        .start          = 16,                                            \
        .config         = cfg,                                           \
        .lock_entry     = 0,                                             \
        .erase_mask_lo  = 0,                                             \
        .erase_mask_hi  = 0,                                             \
        .reserved_hash  = 0,                                             \
    }
```


## Source of "tomu-efm32hg309.ld"?  
I have no experience using linker scripts like this one. But it appears to be 
tied to the onboard EFM32HG309 chip and unlikely to change. I am unable to find
any further upstream source for this file except that is included in the root
folder for both [tomu-quickstart](https://github.com/im-tomu/tomu-quickstart) 
and [tomu-samples](https://github.com/im-tomu/tomu-samples).

Instead of cloning either of those repos, it seems safe to simply copy out this
one file instead (for convenience, it is also included in this repo)