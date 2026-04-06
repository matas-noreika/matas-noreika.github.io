---
layout: post
title:  "Linux Environment With WSL"
date: 2026-03-29 16:33:58 +0100
categories: windows wsl
---

In this blog post we are going to explore a fun option of maintaining the
best of both worlds from Windows and Linux. I discovered
WSL (Windows Subsystem for Linux) around my second year of college.
I wanted to have a Linux development environment without having to give up
all the support of the windows platform. Historically Windows has always had
some support for a Linux like environment to maintain consistency within the
software world.

## How to Set Up

Open up a Powershell terminal in administrator mode and run the following
command:

```Shell
wsl --install -d <DistroName>
```

In the place of `<DistroName>` I chose Debian although if you leave it out by
default Ubuntu is installed (which runs Debian under the hood anyways).

Your PC will need a restart, once that is complete you can move onto the next
step.

## Using WSL

In order to turn on your distributions terminal you can open windows terminal
application and on the Powershell prompt write `wsl`. This might prompt a user
configuration on initial use.

## Additional Tools

I suggest for users that develop on embedded systems to install an additional
tool called `usbipd` which maps the USB ports virtually into the WSL instance.

Use the following command to install `usbipd` using Windows Package Manager
(winget):

```Shell
winget install --interactive --exact dorssel.usbipd-win
```

Once installed you may use the tool to `list`, `bind` and `attach` the desired devices.
_Ensure you run the Powershell in administrator mode you may run the following
command to open administrator from inside a Powershell terminal_:

```Shell
start prowershell -Verb RunAs
```

List Example:

```Shell
usbipd list
```

Output:

```Shell
Connected:
BUSID  VID:PID    DEVICE                                                        STATE
1-7    13d3:54b1  Integrated Camera                                             Shared
1-9    046d:c53f  LIGHTSPEED Receiver, USB Input Device                         Not shared
1-10   8087:0026  Intel(R) Wireless Bluetooth(R)                                Shared

Persisted:
GUID                                  DEVICE
```

Binding Example:

```Shell
usbipd bind -b <BUSID>
```

If you run `list` again upon binding device the STATE should show Shared.

Attach Example:

```Shell
usbipd attach --wsl -b <BUSID>
```

Output:

```Shell
usbipd: info: Using WSL distribution 'Debian' to attach; the device will be available in all WSL 2 distributions.
usbipd: info: Detected networking mode 'nat'.
usbipd: info: Using IP address 172.27.128.1 to reach the host.
```

If you wish to return the device control to windows run the following command:

```Shell
usbipd detach -b <BUSID>
```
