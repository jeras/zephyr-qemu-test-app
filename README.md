# Zephyr app demo on ARM QEMU

With the intention of testing QEMU record/replay functionality.

The project was created following this tutorial:

https://dojofive.com/blog/using-the-qemu-emulator-with-zephyr-builds-and-vscode/

This repository is just the application mentioned in the tutorial.

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

