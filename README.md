# Guide how to launch Genshin Impact (or other games) on VM

Usual Genshin Impact have anticheat system that prevent the game launch from Wine or Virtual Machine. However, you can "hide" that you using VM. It's not enought to trick all software, but enought to allow Mihoyo/Hoyoverse games (and maybe others).

## Prerequisitions

I assume you are intermediate Linux user and you understand how to install software on your system via command line. It is not step by step guide, so please be creative.

You need:

1. Second GPU (I use Radeon RX 560 2GB). It should be connected to the same or second monitor. *There are "tricks" that allow use the same GPU or monitor, but it requires different setup that I have and I didn't try that or they didn't worked as expected (like [looking glass](https://looking-glass.io/) )*
2. Free SSD or fast HDD with at least 120GB for OS and the game. It might be even more if you need more games or software in windows.
3. Second keyboard/mouse. I'm also had to use second USB sound card (the cheapest ones) to output sound without latency.
4. At least 16Gb RAM or better in the system, I use 48Gb ones. We should be able to dedicate at least 8Gb of its to VM.
5. CPU should support virtualization. *I have intel ivy bridge IRBS ones*

## Installing software.

Here comes the google search. I'm using Gentoo Linux, you may use Ubuntu, Debian, Arch, Steam/OS, or whatever else, so feel free to experiments with.

1. Install KVM/QEMU the emulator. Setup it with https://wiki.gentoo.org/wiki/QEMU
2. Install virt-manager (with GUI). Setup it with https://wiki.gentoo.org/wiki/Virt-manager . You might need to see also https://wiki.gentoo.org/wiki/Libvirt , *but I didn't remember I change something from it.*
3. Download virtio-win iso version at least 0.1.225 - it's a drivers for windows guest vm. *In Gentoo I can just emerge app-emulation/virtio-win*
4. Of course, obtain Windows installer iso somewhere. *I'm using custom pack of Windows 11*

## Formatting disk

From root shell do:

`parted /dev/sdt`
`help`
`print`

Where /dev/sdt is your free disk. If disk does not have disk table, create it with `mktable gpt` Create separate partition with something like

`mkpart guestvm 0% 100%`, where guestvm - sugar name of your partition to simplify its connection.

## Prepare the second gpu for passthough

It's the theme where there are many user guides, so feel free to google it. For example:

https://wiki.gentoo.org/wiki/GPU_passthrough_with_libvirt_qemu_kvm

## Creating VM

We will use virtio drivers where it's possible because they have minimum overhead and therefore maximum performance. Launch virt-manager GUI and start creating new virtual machine. Then do drudgery:

File->New VM->Local Install Media->Forward->(You should select Windows ISO)->(Disable automatic detection of ISO and select windows 11 from search list)->(uncheck mark "enable storage", we will setup it later)->**Check mark "Customise configuration before install"**! ->Finish

And in customise screen we must add via button in left down corner:

1. Storage -> virtio-win iso from step 3 as SATA CDROM device.
2. Storage -> check "select or create custom device" -> in the field write manually path to your vm partition `/dev/disk/by-partlabel/guestvm` -> device type: disk device -> **bus type:virtio**

![Overview](screenshots/create_storage.png?raw=true)

Check that you have UEFI/Q35 selected in overview page.

![Overview](screenshots/overview.png?raw=true)

![Overview](screenshots/overview_2.png?raw=true)

Then you can begin installation (button in the top left corner).

## Windows installation

Note: Sometimes boot order of UEFI can be incorrect. In this case you should force shutoff your VM, then launch it and press ESC button on your keyboard then you'll see TianoCore UEFI setup menu.

Chose the selective (manual) Windows installation, then when you the drive formatting menu appears **you should install virtio drivers from virtio-win iso** (press "Load" the icon of dvd, then chose the appropriate version of driver "Red Hat Virtio SCSI Controller").

After finishing the base installation, open Explorer then open virtio-win disc again and install all other virtio drivers. When they will be installed, reboot guest system, then shutdown it.

## Disabling the hypervisor

Generally you should follow the really good guide from Michael Hampton:

https://superuser.com/a/1389159

I'll just retell it. Instead of using virsh, you may edit XML in the "Details" view of virt-manager. The CPU section will look like this:

```
  <cpu mode='host-model' check='partial'>
    <model fallback='allow'/>
  </cpu>
```

You need to add an element to remove the hypervisor CPU feature, causing it to look like this:

```
  <cpu mode='host-model' check='partial'>
    <model fallback='allow'/>
    <feature policy='disable' name='hypervisor'/>
  </cpu>
```

Now you also need to disable the hypervisor CPUID leaves. This is done by adding a new element inside the <features> element.

Just above ```  </features>``` you should add:

```
  <kvm>
    <hidden state='on'/>
  </kvm>
```

That's all, you allowed Genshin Impact to working! To double check it, you can also run systeminfo in a PowerShell or command prompt. It should show something like that instead of hypervisor data:

```
Hyper-V Requirements:      VM Monitor Mode Extensions: Yes
                           Virtualization Enabled In Firmware: Yes
                           Second Level Address Translation: Yes
                           Data Execution Prevention Available: Yes
```

## Attaching the gpu and installing gpu driver for guest vm

In virt-manager switch view to "details", then add hardware -> Pci host device -> Select your guest gpu from list. Retry for the in-gpu sound device if you have ones.

Then launch the system. Just go to AMD/Nvidia/Intel site and download installer, then install it. But after reboot, the system can stop respond to mouse commands. It happens because QLX emulated gpu isn't compatible with real ones. You should poweroff the guest and disable QLX (set gpu adapter to none) in the vm details.

After that, connect your guest gpu with monitor and in virt-manager details menu you may attach the mouse, the keyboard and (optionally) external usb sound card via USB-redirect.

## Congratulations!

Then you can just install Genshin Impact or whatever you want to play just like plain Windows desktop.

Enjoy the Genshin Impact brainwash!
