# Setup #

## requirements: ##

qemu-system-x86_64 installed.

This will not run inside another virtual host (virtualbox, vmware, etc).

## clone buildroot to any directory outside of this one: ##

```sh
	$ git clone git://git.buildroot.net/buildroot ../buildroot
```

## export variables: ##

Use absolute paths:

```sh
	$ export GBSIMDIR=/path/to/gbsim

	$ export BR2_EXTERNAL=/path/to/ara_buildroot_x86_64
```


# Build #

go to buildroot cloned directory and run:

```sh
	$ make ara_qemu_x86_64_staging_next_defconfig
	$ make
```

grab a cup of coffee!!!

# Run #

- in the top directory of buildroot where you have run make:

```sh
$ qemu-system-x86_64  -M pc -kernel output/images/bzImage \
-drive file=output/images/rootfs.ext2,if=ide \
-append "root=/dev/sda console=ttyS0,115200" -net nic,model=rtl8139 -net user \
-nographic -s
```

- when prompt appear, login as root, no password need and you can run:
```sh
	$ gbsim -h /tmp/gbsim -v
```
- Then in the /etc are some manifest files to copy to /tmp/gbsim/hotplug-module/

There already an alias for the above command:

g

Example output:
```sh
Welcome to Buildroot
buildroot login: root
$ g
hotplug_basedir /tmp/gbsim
verbose 1
file system registered
[I] GBSIM: USB gadget created
read descriptors
read strings
[D] GBSIM: USB BIND
usb 1-1: new high-speed USB device number 2 using dummy_hcd
configfs-gadget gadget: high-speed config #1: config
[D] GBSIM: USB ENABLE
[D] GBSIM: Start SVC/CPort endpoints
[D] GBSIM: Module -> AP CPort 0 SVC GB_SVC_TYPE_PROTOCOL_VERSION request
[R] GBSIM: DUMP -> 0a 00 01 00 01 00 00 00 00 01
usb 1-1: Unknown endpoint type found, address 81
[D] GBSIM: AP -> Module 1 CPort 0 SVC GB_SVC_TYPE_PROTOCOL_VERSION response
[R] GBSIM: DUMP -> 0a 00 01 00 81 00 00 00 00 01
[D] GBSIM: svc_handler_response: Version major-0 minor-1
[D] GBSIM: Module -> AP CPort 0 SVC GB_SVC_TYPE_SVC_HELLO request
[R] GBSIM: DUMP -> 0b 00 01 00 02 00 00 00 55 47 05
[D] GBSIM: AP -> Module 1 CPort 0 SVC GB_SVC_TYPE_SVC_HELLO response
[R] GBSIM: DUMP -> 08 00 01 00 82 00 00 00
```

- It is also possible to launch gbsim and setup a simulated module for each
available greybus protocol. This can be done even if another protocol setup is
already running and will replace the simulated module by the new one.

- example to start sdio:
	```sh
	Welcome to Buildroot
	buildroot login: root
	$ sdio
	```
- available commands:
	```sh
	. sdio
	. lights
	. gpio
	. i2c
	. pwm
	. power-supply
	. uart
	. loopback
	```

- To stop qemu do:

	Ctrl-a-x

# Modifying #

* in BR2_EXTERNAL_ARA_PATH/board/ara/overlay/etc is the etc directory with the init
  script to load modules and manifestos.

* rebuild individual components (all in buildroot directory):

	- kernel / greybus:
	```sh
		$ make linux-rebuild; make
	```

	- gbsim:
	```sh
		$ make gbsim-dirclean; make gbsim; make
	```

	- change kernel configuration:
	```sh
		$ make linux-menuconfig
	```

	- save current kernel configuration to repository:
	```sh
		$ make linux-update-defconfig
	```

go to step Run.

# Debuging #

To attach a gdb you can use a gdbinit file already in the overlay:
```sh
	$ cd /path/to/buildroot/output/build/linux-staging-next
	$ gdb -x $BR2_EXTERNAL/board/ara/qemu_x86_64/gdbinit
```

or if you prefer:
```sh
	$ cgdb -- -x $BR2_EXTERNAL/board/ara/qemu_x86_64/gdbinit
```

at the last stage of boot it is printout the file and address of the greybus
modules, you can add them to the gdbinit as they should not change that much or
copy/paste to the gdb prompt every time you run it, ex:

```sh
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-audio-apbridgea.o 0xffffffffa003f000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-audio-gb.o 0xffffffffa0043000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-audio-manager.o 0xffffffffa0047000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-bootrom.o 0xffffffffa004c000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-es2.o 0xffffffffa0051000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-firmware.o 0xffffffffa005c000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-gbphy.o 0xffffffffa0064000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-gpio.o 0xffffffffa0069000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-hid.o 0xffffffffa006e000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-i2c.o 0xffffffffa0073000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-light.o 0xffffffffa0077000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-log.o 0xffffffffa007d000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-loopback.o 0xffffffffa0081000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-power-supply.o 0xffffffffa008b000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-pwm.o 0xffffffffa0090000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-raw.o 0xffffffffa0094000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-sdio.o 0xffffffffa0098000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-spi.o 0xffffffffa009d000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-spilib.o 0xffffffffa0058000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-uart.o 0xffffffffa00a1000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-usb.o 0xffffffffa00a8000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/gb-vibrator.o 0xffffffffa00ac000
add-symbol-file /path/to/buildroot/output/build/linux-staging-next/drivers/staging/greybus/greybus.o 0xffffffffa0024000
```
