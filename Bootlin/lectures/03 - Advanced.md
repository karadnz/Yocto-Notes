
60 - 86

advanced build usage and configuration:

- Variable operators and overrides. 
- Select package variants. 
- Manually add packages to the generated image.
- Run specific tasks with BitBake.

a little reminder:

-  A **Recipe** describes how to fetch, configure, compile and install a software component (application, library, …).
- These **tasks** can be run independently (if their dependencies are met).
- All the available packages in the project layer are not selected by default to be built and included in the images.
- Some packages may provide the same functionality, e.g. OpenSSH and Dropbear.


## Variables

### Overview

- The OpenEmbedded build system uses **configuration variables** to hold information.
- Variable *names* are in upper-case by convention, e.g. `CONF_VERSION`
- Variable *values* are strings
- To make configuration easier, it is possible to prepend, append or define these variables in a conditional way.
- Variables defined in **Configuration Files** have a **global** scope 
	- Files ending in `.conf`
- Variables defined in **Recipes** have a **local** scope
	- Files ending in `.bb`, `.bbappend and` `.bbclass`
- Recipes can also access the global scope
- All variables can be overridden in `$BUILDDIR/conf/local.conf`

### Operators

Various operators can be used to assign values to configuration variables:

- `=` expand the value when using the variable
- `:=` immediately expand the value
- `+=` append (with space)
- `=+` prepend (with space)
- `.=` append (without space)
- `=.` prepend (without space)
- `?=` assign if no other value was previously assigned
- `??=` same as previous, with a lower precedence

Operator Caveats:

- The operators apply their effect during parsing.
- Example:
- ![[Pasted image 20240404105737.png]]
- The parsing order of files is difficult to predict, no assumption should be made about it.
- To avoid the problem, avoid using `+=,` `=+,` `.=` and `=.` in `$BUILDDIR/conf/local.conf`. Always use overrides (see following sections).

### Overrides

- Bitbake **overrides** allow appending, prepending or modifying a variable at expansion time, when the variable’s value is read
- Overrides are written as ` <VARIABLE>:<override> = "some_value"`
- A different syntax was used before Honister (3.4), with no retrocompatibility: `<VARIABLE>_<override> = "some_value"`

#### Overrides to modify variable values

- The append override adds **at the end** of the variable (without space).
	- `IMAGE_INSTALL:append = " dropbear"` adds dropbear to the packages installed on the image.
- The prepend override adds **at the beginning** of the variable (without space).
	- `FILESEXTRAPATHS:prepend := "${THISDIR}/${PN}:"` adds the folder to the set of paths where files are located (in a recipe).
- The remove override removes all occurrences of a value within a variable. 
	- `IMAGE_INSTALL:remove = "i2c-tools"`

#### Overrides for conditional assigment

- Append the machine name to only define a configuration variable for a given machine.
- It tries to match with values from `OVERRIDES` which includes MACHINE, `SOC_FAMILY`, and more.
- If the override is in `OVERRIDES`, the assignment is applied, otherwise it is ignored.

```bash
OVERRIDES="arm:armv7a:ti-soc:ti33x:beaglebone:poky" 

KERNEL_DEVICETREE:beaglebone = "am335x-bone.dtb" // This is applied KERNEL_DEVICETREE:dra7xx-evm = "dra7-evm.dtb" // This is ignored
```
(idk why  obsidian refuses to render # as comment, i used // instead)


- The most specific assignment takes precedence.
- Example:
	- `IMAGE_INSTALL:beaglebone = "busybox mtd-utils i2c-tools"`
	- `IMAGE_INSTALL = "busybox mtd-utils"`
- If the machine is beaglebone: 
	- `IMAGE_INSTALL = "busybox mtd-utils i2c-tools"`
- Otherwise:
	- `IMAGE_INSTALL = "busybox mtd-utils"`


#### Combining Overrides

- The previous methods can be combined.
- If we define:
	- `IMAGE_INSTALL = "busybox mtd-utils"`
	- `IMAGE_INSTALL:append = " dropbear"`
	- `IMAGE_INSTALL:append:beaglebone = " i2c-tools"`
- The resulting configuration variable will be:
	- `IMAGE_INSTALL = "busybox mtd-utils dropbear i2c-tools"` if the machine being built is beaglebone.
	- `IMAGE_INSTALL = "busybox mtd-utils dropbear"` otherwise.

### Order of variable Assigment

![[Pasted image 20240404111649.png]]

### bitbake-getvar

- `bitbake-getvar` can be used to understand and debug how variables are assigned
- `bitbake-getvar <VARIABLE>`
- Lists each configuration file touching the variable, the pre-expansion value and the final value

![[Pasted image 20240404111827.png]]
(todo: find a way to properly display code blocks in obsidian)


## Package Variants

### Intro

- Some packages have the same purpose, and only one can be used at a time.
- The build system uses **virtual packages** to reflect this. A virtual package describes functionalities and several packages may provide it.
- Only one of the packages that provide the functionality will be compiled and integrated into the resulting image.

### Variant Examples

- The virtual packages are often in the form `virtual/<name>`
- Example of available virtual packages with some of their variants:
	- `virtual/bootloader`: u-boot, u-boot-ti-staging…
	- `virtual/kernel`: linux-yocto, linux-yocto-tiny, linux-yocto-rt, linux-ti-staging… 
	- `virtual/libc`: glibc, musl, newlib • virtual/xserver: xserver-xorg


### Package Selection

- Variants are selected thanks to the `PREFERRED_PROVIDER` configuration variable.
- The package names **have to** suffix this variable.
- Examples:
	- `PREFERRED_PROVIDER_virtual/kernel ?= "linux-ti-staging" `
	- `PREFERRED_PROVIDER_virtual/libgl = "mesa"`

### Version Selection

- By default, Bitbake will try to build the provider with the highest version number, from the highest priority layer, unless the recipe defines `DEFAULT_PREFERENCE = "-1"`
- When multiple package versions are available, it is also possible to explicitly pick a given version with `PREFERRED_VERSION`.
- The package names **have to** suffix this variable.
- `%` can be used as a wildcard.
- Example:
	- `PREFERRED_VERSION_nginx = "1.20.1" `
	- `PREFERRED_VERSION_linux-yocto = "5.14%"`

## Selection of Packages to Install

- The set of packages installed into the image is defined by the target you choose (e.g. `core-image-minimal`).
- It is possible to have a custom set by defining our own target, and we will see this later.
- When developing or debugging, adding packages can be useful, without modifying the recipes.
- Packages are controlled by the `IMAGE_INSTALL` configuration variable.


## The Power of BitBake

### Common BitBake options

- BitBake can be used to run a full build for a given target with `bitbake [target]`
	- target is a recipe name, possibly with modifiers, e.g. `-native `
	- `bitbake ncurses` 
	- `bitbake ncurses-native`
- But it can be more precise, with additional options:
	- `-c <task>` execute the given task
		- `-s` list all available recipes and their versions
		- `-f` force the given task to be run by removing its stamp file world keyword for all recipes

### BitBake Examples

- `bitbake -c listtasks virtual/kernel`
	- Gives a list of the available tasks for the recipe providing the package `virtual/kernel`. Tasks are prefixed with do_.
- `bitbake -c menuconfig virtual/kernel `
	- Execute the task menuconfig on the recipe providing the `virtual/kernel` package.
- `bitbake -f dropbear `
	- Force the dropbear recipe to run all tasks.
- `bitbake --runall=fetch core-image-minimal `
	- Download all recipe sources and their dependencies.
- For a full description: `bitbake --help`


### Shared State Cache

- **BitBake** stores the output of each task in a directory, the shared state cache.
- This cache is used to speed up compilation.
- Its location is defined by the `SSTATE_DIR` variable and defaults to `build/sstate-cache`.
- Over time, as you compile more recipes, it can grow quite big. It is possible to clean old data with:

```bash
$ find sstate-cache/ -type f -atime +30 -delete
```

- This removes all files that have last been accessed more than 30 days ago (for example).

go see lab02