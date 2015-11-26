# vmscripts

This is a small collection of scripts I wrote to ease the burden of using
VirtualBox VMs. They're pretty tiny, so I'm putting them in the public domain.

Assuming you have a suitably-configured (see below) VM called `casimir`, you
can start it with `runvm casimir`. The `runvm` script will pipe console output
to the terminal and turn the VM off when it gets a `SIGINT` (from control-c,
for example). The first `SIGINT` sends an ACPI power button event to allow the
VM to shut down cleanly. A second `SIGINT` sends a `poweroff` command to shut
down harder, which is useful if the VM isn't stopping on its own.

Once the VM is running and has networking set up, `ping $(vmip casimir)` and
`sshvm casimir` will do what you expect. (`sshvm casimir -t tmux -u2CC`
combined with iTerm's tmux support is particularly handy.)

## Host setup

I wrote these for myself, so I haven't made any attempt to make them
cross-platform or anything. As far as I know, they should work on any OSX
system with VirtualBox and `socat` installed.

## VM setup

The scripts make a few assumptions about the VMs:

 * There is a network adapter that the host can reach with an IP in the
   `196.168.0.0/16` range. (Hardcoded in `vmip`, but easy enough to change.)

 * VirtualBox Guest Additions are installed in the VM so `vmip` (and `sshvm`)
   can find the relevant IP.

 * There is a serial port configured on `COM1`. `runvm` will set its mode to
   `Host Pipe` pointing to a suitable temp file.

 * The guest operating system writes its console log to the abovementioned
   serial port. I configure my guests to not start `getty` there, because the
   `socat` process runs in the background and doesn't accept input, making the
   login prompt useless and confusing.

To set up the serial console on Debian Jessie, edit `/etc/default/grub` and
replace the `GRUB_CMDLINE_LINUX_DEFAULT` entry with the following:

    GRUB_CMDLINE_LINUX_DEFAULT="console=tty0 console=ttyS0,115200n8"

I also set `GRUB_TERMINAL=serial` to see the grub screen (even though I can't
interact with it) and `GRUB_TIMEOUT=1` to minimise the startup time. Neither of
these are strictly required, though.

To disable `getty` on the serial terminal:

    systemctl mask serial-getty@ttyS0.service

## TODO

Make some of this stuff more robust. I haven't really tested any edge cases or
weird states, but there are almost certainly some potential bad behaviours in
`runvm`.
