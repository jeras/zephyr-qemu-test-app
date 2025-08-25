# Zephyr app demo on ARM QEMU

With the intention of testing QEMU record/replay functionality.

The project was created following this tutorial:

https://dojofive.com/blog/using-the-qemu-emulator-with-zephyr-builds-and-vscode/

This repository is just the application mentioned in the tutorial.

To compile Zephyr from the application folder `.../zephyrproject/zephyr-qemu-test-app/`:

```sh
source ../.venv/bin/activate
west build -b qemu_cortex_a53
west build -t run
```

Use verbose mode `west -v` to get the details of the CLI for running QEMU.
Or better, search for `debugserver_qemu` in `build.ninja` in the build folder.

The tutorial was extended by adding verbose debug logging to GDB and
by adding QEMU record and replay configurations to the normal one.

https://qemu-project.gitlab.io/qemu/system/replay.html

While reverse time options are not recognized by the `cortex-debug` VSCode extension,
They are clearly advertized by the QEMU GDB stub:

```gdb
  [remote] Sending packet: $qSupported:multiprocess+;swbreak+;hwbreak+;qRelocInsn+;fork-events+;vfork-events+;exec-events+;vContSupported+;QThreadEvents+;no-resumed+;memory-tagging+#ec
  [remote] Received Ack
  [remote] Packet received: PacketSize=1000;qXfer:features:read+;ReverseStep+;ReverseContinue+;vContSupported+;multiprocess+
  [remote] packet_ok: Packet qSupported (supported-packets) is supported
```

## Debugging

The official method for running QEMU with a debug socket would be:

```sh
west debugserver
```

But it does not seem to work, an alternative is:

```sh
ninja -C build/ debugserver_qemu -v
```

To connect the Seer GUI to the socket run:

```sh
seergdb --gdb-program gdb-multiarch --project project.seer
```

### QEMU record/replay

Following the previous steps, the ELF file is `build/zephyr/zephyr.elf`.

Launch QEMU in `record` mode to file `record.bin`.

```sh
$ ~/zephyr-sdk-0.17.2/sysroots/x86_64-pokysdk-linux/usr/bin/qemu-system-aarch64 -global virtio-mmio.force-legacy=false -cpu cortex-a53 -nographic -machine virt,secure=on,gic-version=3 -net none -pidfile qemu.pid -chardev stdio,id=con,mux=on -serial chardev:con -mon chardev=con,mode=readline -icount shift=auto,rr=record,rrfile=record.bin -rtc clock=vm -S -gdb tcp::1234 -kernel build/zephyr/zephyr.elf
```

And in a separate shell run GDB with a script to create a recording.

```sh
gdb-multiarch -x scripts/gdb-record.scr
```

The QEMU `rrfile` file does not contain the a system state snapshot
or small grained system state diff for each instruction step.
Instead it requires a separate snapshot and the `rrfile` contains
only the record of nondeterministic input.

A snapshot file is needed to run the replay.
The snapshot file is created by running:

```sh
qemu-img create -f qcow2 scratch.qcow2 64M
```

Launch QEMU in `replay` mode from file `record.bin` (created in the previous example).
Compared to recording, replaying requires a snapshot file (`-blockdev`).

```sh
$ ~/zephyr-sdk-0.17.2/sysroots/x86_64-pokysdk-linux/usr/bin/qemu-system-aarch64 -global virtio-mmio.force-legacy=false -cpu cortex-a53 -nographic -machine virt,secure=on,gic-version=3 -net none -pidfile qemu.pid -chardev stdio,id=con,mux=on -serial chardev:con -mon chardev=con,mode=readline -icount shift=auto,rr=replay,rrfile=record.bin -rtc clock=vm -S -gdb tcp::1234 -kernel build/zephyr/zephyr.elf -blockdev driver=qcow2,node-name=replayhdhd,file.driver=file,file.filename=scratch.qcow2
```

Since the recording is short (only 8 instructions)
stepping through it will soon reach the end and terminate.
There is a [QEMU issue](https://gitlab.com/qemu-project/qemu/-/issues/3076)
intended for improving the behavior
of attempted stepping outside the recorded execution.

```sh
seergdb --gdb-program gdb-multiarch --project project.seer
```

This unfinished script can be used to automatically test record/replay features:

```sh
gdb-multiarch -x scripts/gdb-replay.scr
```

### Running GDB:

```sh
$ gdb-multiarch
(gdb) set logging enabled on
(gdb) set debug remote 1
(gdb) file build/zephyr/zephyr.elf
(gdb) target extended-remote localhost:50000
```

To run GDB with a `filename` script do:

```sh
$ gdb-multiarch -x filename
```

GDB scripts:
 
* [gdb-record.scr](scripts/gdb-record.scr)
* [gdb-replay.scr](scripts/gdb-replay.scr)

## QEMU

```sh
./configure
make qemu-system-aarch64
make qemu-system-arm
make qemu-system-x86_64
make qemu-system-riscv32
make qemu-system-riscv64
```

To Compile functiona tests:

```sh
make check-functional-aarch64

$ make check-functional-mips
5/6 qemu:func-thorough+func-mips-thorough+thorough / func-mips-mips_replay        OK               1.75s   1 subtests passed
```



To compile a debug version of QEMU:

```
./configure --enable-debug
qemu-system-... -D -d ...
```


$qSupported:multiprocess+;swbreak+;hwbreak+;qRelocInsn+;fork-events+;vfork-events+;exec-events+;vContSupported+;QThreadEvents+;QThreadOptions+;no-resumed+;memory-tagging+;xmlRegisters=i386

## GDB

./configure \
--with-auto-load-dir=$debugdir:$datadir/auto-load \
--with-auto-load-safe-path=$debugdir:$datadir/auto-load \
--with-expat \
--without-libunwind-ia64 \
--with-lzma \
--with-debuginfod \
--with-curses \
--without-guile \
--without-amd-dbgapi \
--enable-source-highlight \
--enable-threading \
--enable-tui \
--with-system-readline \
--enable-targets=all

lsb-release xz-utils autoconf libtool gettext bison dejagnu flex procps gobjc texinfo texlive-base
libexpat1-dev libncurses-dev libreadline-dev zlib1g-dev liblzma-dev libzstd-dev libbabeltrace-dev libipt-dev libsource-highlight-dev libxxhash-dev libmpfr-dev pkg-config python3-dev libdebuginfod-dev libc6-dbg
