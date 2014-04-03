# supermin-wrapper

This program is intended to be friendly to a Continuous Integration / Delivery build pipeline. It should be used after a versioned RPM repository has been produced in order to create reproducible machine images, suitable for use as Phoenix (immutable) servers. As much as possible, it allows one to capture configuration in source control. Where it can't be, it allows the use of scriptlets to generate it.

A wrapper around supermin that creates virtual machine images (disk images) based on a set of repositories. Machine settings are captured in source control and are defined using a mixture of RPM repositories, packages to install, local files to install and programs to generate files to install.

## Syntax Notes
The syntax `${variable}` is used in this document to refer to variables. These refer to the values set when calling supermin-wrapper:-

See `supermin-wrapper -h` for more detailed help. For reference, the variables we refer to are:-

* `${configPath}`, which is the location of all configuration (machines, etc)
* `${cachePath}`, which is the location of a cache, normally at `${configPath}/cache`
* `${pathsPath}`, which defines a configuration folder containing the paths to all binaries needed, normally at `${configPath}/paths`

The `${cachePath}` should contain a .gitignore file with the value `*` in it, so that cached data is not accidentally checked in.

The variable `${machine}` is used to refer to a machine. The value is the machine's hostname (DNS label) without domain. We don't currently validate that it is DNS-valid.

## Machine templates
Many aspects of a machine's profile are common. To enable this, you can symlink most per-machine files to a common one. By convention, this common folder is at `${configPath}/machine=template`, but it really doesn't matter what it's called, and you could conceivably have several.

## Defining a new machine
Machines are defined in a folder at `${configPath}/${machine}`.

### Packages
A machine is defined as a set of RPM packages. The packages to install are in a text file at `${configPath}/${machine}/packages`. They are listed one-per-line (LF line endings), without version numbers or architectures. For example, a file containing:-

    bash
    coreutils

Will install the packages `bash` and `coreutils`, and all their dependencies. The package `setup` is always installed regardless.

The `packages` file may be a symlink. The source of these packages is controlled by Yum and Repository Configuration (see below).

#### Yum and Repository Configuration
RPM packages are installed using `yum`. A special configuration is used per-machine, to ensure isolation from the build machine. This needs to exist either as a folder or a symlink to a folder at `${configPath}/${machine}/yum`. Conventionally, this is symlinked to `${configPath}/machine-template/yum` as it rarely changes per-machine. This folder contains contents as follows:-

    yum.conf.template  (file template)
    repositories       (folder)
    plugins            (folder, normally empty)

#### `yum.conf.template`
This file is a template `yum.conf`, which has variables marked `${configMachinePath}` and `${cacheMachinePath}` substituted before it is used (standard `$YUM0`, etc variables don't work). There should not normally be any need to deviate from that supplied in `${configPath}/machine-template/yum/yum.conf.template`.

#### `repositories`
This folder contains yum repo configuration snippets (end `.repo`) (examples can be find in `/etc/yum.repos.d` on most RedHat-derived systems). It is recommended that these configuration snippets be changed from those supplied in `${configPath}/machine-template/yum/repositories` to point to a versioned repository produced by a Continuous Deployment / Integration build pipeline.

### Init scripts
XXXX TODO

### Additional Files
Additional files are installed after the RPM packages as overlays onto the image. There are two main kinds:-

* devices, and
* files, known as root-overlays

The reason for the split is that devices (specifically, block device files, character device files and FIFO files) can not be stored in Git and other source control systems.

#### Devices
Devices are stored for each machine in the file `${configPath}/${machine}/devices`. They are listed one-per-line (LF line endings) with a space-delimited format. For example:-

    /dev/ram0 b 660 1 0
    /dev/tty0 c 620 4 0
    /dev/xconsole p 0755

The format is `device path`, `device type`, `permissions`, `major` and `minor`. The `device type` is as follows:-

* `b` a block device
* `c` a character device
* `p` a FIFO

The `major` and `minor` are positive decimal integers and are zero-based. FIFO devices do not have a `major` and `minor`; specifying them will cause the machine image to fail to build.

The `devices` file may be a symlink.

#### Root Overlays
Root overlays are hierarchal folders that will overlay in `/` in the build machine image.

Root overlays are folders in `${configPath}/${machine}/root-overlays

Due to the design of the supermin-wrapper, any files starting with a period (`.`) (also known as hidden files) in the root of the overlay are not copied. This ensures things like `.gitignore` files are not copied in. Ordinarily, this shouldn't be an issue, because it is exceedingly rare for hidden files to exist in the root (`/`) of a Linux server.

Please note that most source control systems do not preserve file owners or users or permissions, apart from execute.


### Generator Scriptlets


## Where are things written to?

### Debugging Output

## Clearing the cache
The wrapper uses a cache per machine. To remove the cache for a machine, delete the folder
`${cachePath}/${machine}`. To remove