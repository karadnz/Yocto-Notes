
## Bitbake

In Yocto / OpenEmbedded, the build engine is implemented by the bitbake program.

- **Bitbake** is a task scheduler, like make .
- **Bitbake** parses text files to know what it has to build and how .
- It is written in Python (need Python 3 on the development host).

## Recipes

- The main kind of text file parsed by bitbake is **recipes**, each describing a specific software component.
- Each **Recipe** describes how to fetch and build a software component: e.g. a program, a library or an image.
- They have a specific syntax.
- Bitbake can be asked to build any **recipe**, building all its dependencies automatically beforehand.

![[Pasted image 20240403163959.png]]



## Tasks

The build process implemented by a recipe is split in several **tasks**

- Each **task** performs a specific step in the build.
- Examples: fetch, configure, compile, package.
- **Tasks** can depend on other tasks (including on tasks of other recipes).

![[Pasted image 20240403164125.png]]

## Metadata & Layers

- The input to bitbake is collectively called **metadata**.
- **Metadata** includes configuration files, recipes, classes and include files.
- **Metadata** is organized in **layers**, which can be composed to get various components
	- A **layer** is a set of recipes, configurations files and classes matching a common purpose. 
		- For Texas Instruments board support, the `meta-ti-bsp` layer is used. 
	- Multiple **layers** are used for a project, depending on the needs.
- `openembedded-core` is the core **layer**
	- All other layers are built on top of `openembedded-core`.
	- It supports the ARM, MIPS (32 and 64 bits), PowerPC, RISC-V and x86 (32 and 64 bits) architectures.
	- It supports QEMU emulated machines for these architectures.

## Poky

- **Poky** has several meanings.
- **Poky** is a git repository that is assembled from other gir repositories: `bitbake`, `openembedded-core`, `yocto-docs` and `meta-yocto`
- **Poky** is the reference distro provided by the Yocto Project
- `meta-poky` is the layer providing the **Poky** reference distro.

![[Pasted image 20240403165326.png]]

- To download the Poky reference system: `git clone -b kirkstone https://git.yoctoproject.org/git/poky` .
- A new version is released every 6 months, and maintained for 7 months.
- LTS versions are maintained for 4 years, and announced before their release.
- Each release has a codename such as kirkstone or honister, corresponding to a release number.

### Poky source tree

- `bitbake/` Holds all scripts used by the bitbake command. Usually matches the stable release of the BitBake project.
- `documentation/` All documentation sources for the Yocto Project documentation. Can be used to generate nice PDFs.
- `meta/` Contains the OpenEmbedded-Core metadata.
- `meta-skeleton/` Contains template recipes for BSP and kernel development.
- `meta-poky/` Holds the configuration for the Poky reference distribution.
- `meta-yocto-bsp/` Configuration for the Yocto Project reference hardware board support package.
- `LICENSE` The license under which Poky is distributed (a mix of GPLv2 and MIT).
- `oe-init-build-env` Script to set up the OpenEmbedded build environment. It will create the build directory.
- `scripts/` Contains scripts used to set up the environment, development tools, and tools to flash the generated images on the target.

## Yocto project

- The **Yocto Project** is not used as a finite set of layers and tools.
- Instead, it provides a **common base** of tools and layers on top of which custom and specific layers are added, depending on your target.