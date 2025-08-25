# Zephyr app demo on ARM QEMU

With the intention of testing QEMU record/replay functionality.

The project was created following this tutorial:

https://dojofive.com/blog/using-the-qemu-emulator-with-zephyr-builds-and-vscode/

This repository is just the application mentioned in the tutorial.

To compile Zephyr from the application folder `.../zephyrproject/zephyr-qemu-test-app/`:

```sh
source ../.venv/bin/activate
west build -d build-qemu_cortex_m3 -b qemu_cortex_m3
west build -d build-qemu_cortex_m3 -t run
west build -d build-qemu_cortex_r5 -b qemu_cortex_r5
west build -d build-qemu_cortex_r5 -t run
west build -d build-qemu_cortex_a53 -b qemu_cortex_a53
west build -d build-qemu_cortex_a53 -t run
west build -d build-qemu_x86 -b qemu_x86
west build -d build-qemu_x86 -t run
west build -d build-qemu_x86 -t debugserver_qemu
west build -d build-qemu_x86_64 -b qemu_x86_64
west build -d build-qemu_x86_64 -t run
west build -d build-qemu_riscv32 -b qemu_riscv32
west build -d build-qemu_riscv32 -t run
west build -d build-qemu_riscv64 -b qemu_riscv64
west build -d build-qemu_riscv64 -t run
```

Use verbose mode `west -v` to get the details of the CLI for running QEMU.
Or better, search for `debugserver_qemu` in `build.ninja` in the build folder.
Some targets like `i386` might clear the screen during execution,
in this case pipe the standard output into a log file (`... > qemu.log`).

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

## CLI tests

### ARM

Following the previous steps, the ELF file is `build/zephyr/zephyr.elf`.

Launch QEMU in `record` mode to file `record.bin`.

```sh
qemu-system-arm -cpu cortex-m3 -machine lm3s6965evb -nographic -semihosting-config enable=on,target=native -gdb "tcp::50000" -S -kernel build-qemu_cortex_m3/zephyr/zephyr.elf -icount shift=auto,rr=record,rrfile=record-qemu_cortex_m3.bin
```

Launch QEMU in `replay` mode from file `record.bin` (created in the previous example).

```sh
qemu-system-arm -cpu cortex-m3 -machine lm3s6965evb -nographic -semihosting-config enable=on,target=native -gdb "tcp::50000" -S -kernel build-qemu_cortex_m3/zephyr/zephyr.elf -icount shift=auto,rr=replay,rrfile=record-qemu_cortex_m3.bin
```

### i386

Record:
```sh
~/VLSI/qemu/build/qemu-system-i386 -m 32 -cpu qemu32,+nx,+pae -machine q35 -device isa-debug-exit,iobase=0xf4,iosize=0x04 -no-reboot -nographic -machine acpi=off -net none -pidfile qemu.pid -chardev stdio,id=con,mux=on -serial chardev:con -mon chardev=con,mode=readline -icount shift=auto,rr=record,rrfile=record-qemu_x86.bin -rtc clock=vm -kernel build-qemu_x86/zephyr/zephyr.elf -gdb "tcp::50000" -S
```
Replay:
```sh
~/VLSI/qemu/build/qemu-system-i386 -m 32 -cpu qemu32,+nx,+pae -machine q35 -device isa-debug-exit,iobase=0xf4,iosize=0x04 -no-reboot -nographic -machine acpi=off -net none -pidfile qemu.pid -chardev stdio,id=con,mux=on -serial chardev:con -mon chardev=con,mode=readline -icount shift=auto,rr=replay,rrfile=record-qemu_x86.bin -rtc clock=vm -kernel build-qemu_x86/zephyr/zephyr.elf -gdb "tcp::50000" -S
```
```
qemu-system-i386 -cpu qemu32,+nx,+pae -machine q35 -device isa-debug-exit,iobase=0xf4,iosize=0x04 -nographic -semihosting-config enable=on,target=native -gdb "tcp::50000" -S -kernel build-qemu_x86/zephyr/zephyr.elf -icount shift=auto,rr=replay,rrfile=record-qemu_x86.bin
qemu-system-i386 -cpu qemu32,+nx,+pae -machine q35 -device isa-debug-exit,iobase=0xf4,iosize=0x04 -nographic -semihosting-config enable=on,target=native -gdb "tcp::50000" -S -kernel build-qemu_x86/zephyr/zephyr.elf -icount shift=auto,rr=replay,rrfile=record-qemu_x86.bin
```
-machine q35

### Cortex-A53

#### Record

```sh
$ ~/zephyr-sdk-0.17.2/sysroots/x86_64-pokysdk-linux/usr/bin/qemu-system-aarch64 -global virtio-mmio.force-legacy=false -cpu cortex-a53 -nographic -machine virt,secure=on,gic-version=3 -net none -pidfile qemu.pid -chardev stdio,id=con,mux=on -serial chardev:con -mon chardev=con,mode=readline -icount shift=auto,rr=record,rrfile=record-qemu_cortex_a53.bin -rtc clock=vm -S -gdb tcp::1234 -kernel /home/izi/zephyrproject/TestApp/build-qemu_cortex_a53/zephyr/zephyr.elf

gdb-multiarch -x scripts/gdb-record.scr
```

```sh
$ ~/zephyr-sdk-0.17.2/sysroots/x86_64-pokysdk-linux/usr/bin/qemu-system-aarch64 -global virtio-mmio.force-legacy=false -cpu cortex-a53 -nographic -machine virt,secure=on,gic-version=3 -net none -pidfile qemu.pid -chardev stdio,id=con,mux=on -serial chardev:con -mon chardev=con,mode=readline -icount shift=auto,rr=replay,rrfile=record-qemu_cortex_a53.bin -rtc clock=vm -S -gdb tcp::1234 -kernel /home/izi/zephyrproject/TestApp/build-qemu_cortex_a53/zephyr/zephyr.elf

gdb-multiarch -x scripts/gdb-record.scr
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
