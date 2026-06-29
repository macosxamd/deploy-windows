Windows NT 5.x $OEM$ Deployment Guide

This guide provides a professional, step-by-step explanation of the $OEM$ folder usage in Windows 2000/XP/2003 (NT 5.x) unattended deployments, including key files like CMDLINES.TXT and the OemPnPDriversPath setting.

1. Overview of $OEM$ Folder Purpose

The $OEM$ folder is a special directory used during unattended Windows setup to:

Copy additional files to the Windows directory, system32, or root of the system drive.

Inject text-mode drivers (mass storage, NICs) so Setup can detect disks and network adapters.

Run custom scripts during GUI-mode setup via CMDLINES.TXT.

Optionally copy files to other drives using drive-letter subfolders.

All actions are controlled by the answer file (WINNT.SIF or UNATTEND.TXT) with the key:

[Unattended]
OemPreinstall = Yes

Without OemPreinstall=Yes, the $OEM$ folder is ignored.

2. Core $OEM$ Folder Structure

Typical structure in a flat share or CD-based distribution:

\i386
$OEM$
$$
$1
TEXTMODE
CMDLINES.TXT

$OEM$ (root): Base OEM folder containing all OEM files and scripts.

$OEM$$$: Files copied to %WINDIR% (e.g., C:\Windows).

$OEM$$$\System32: Files copied to %WINDIR%\System32.

$OEM$$1: Files copied to the root of the system drive (usually C:).

$OEM$\DriveLetter (e.g., C, D): Files copied to other drive roots.

$OEM$\TEXTMODE: Text-mode drivers used during the blue-screen phase of Setup.

$OEM$\CMDLINES.TXT: INF-style script file with commands run during GUI-mode setup.

3. Step-by-Step Deployment Options and Folder Placement

3.1 CD-Based Deployment

On a Windows XP/2003 installation CD, the $OEM$ folder must be placed inside the architecture folder, either I386 (for x86) or AMD64 (for 64-bit). For example:

CDROOT\
I386\
$OEM$\
$$\
$1\
TEXTMODE\
CMDLINES.TXT

or

CDROOT\
AMD64\
$OEM$\
$$\
$1\
TEXTMODE\
CMDLINES.TXT

Important: The $OEM$ folder must be inside the I386 or AMD64 folder, not at the root of the CD. Setup looks for it there automatically.

3.2 Flat Network Share Deployment

When deploying from a network share, the typical structure is:

\\SERVER\XP\
I386\
$OEM$\
$$\
$1\
TEXTMODE\
CMDLINES.TXT
UNATTEND.TXT

Here, $OEM$ sits at the same level as the I386 folder.

You can also redirect the OEM files location using OemFilesPath in the answer file:

[Unattended]
OemPreinstall = Yes
OemFilesPath = "\\SERVER\\OEMFILES"

This points Setup to \\SERVER\\OEMFILES$OEM$ instead of the default location.

3.3 RIS Deployment

Remote Installation Services (RIS) stores OS images on the server. The $OEM$ folder is placed inside the image folder, alongside I386:

\\RemoteInstall\
Setup\
English\
Images\
XP_PRO\
I386\
$OEM$\
$$\
$1\
TEXTMODE\
CMDLINES.TXT
ristndrd.sif

Setup uses the $OEM$ folder here similarly to flat share deployment.

3.4 Summary Table of $OEM$ Folder Placement

Deployment Type

$OEM$ Folder Location

Notes

CD-based install

Inside I386 or AMD64 folder

Must be inside architecture folder on CD

Flat network share

At the same level as I386 folder

Can use OemFilesPath to redirect

RIS image

Inside the image folder, next to I386

Similar to flat share, inside image folder

4. Understanding CMDLINES.TXT

CMDLINES.TXT is an INF-style text file located directly inside the $OEM$ folder. It controls commands that run automatically during the GUI-mode phase of Windows Setup at the T-12 point (near the end of GUI-mode setup, before the first user logon).

4.1 Purpose and Usage

Run scripts, batch files, or commands to customize the system before the user logs in.

Common tasks: importing registry tweaks, creating users/groups, copying files, running setup scripts.

4.2 Location

$OEM$\CMDLINES.TXT

4.3 Format

The file uses a simple INF-style section called [Commands] listing commands to run, each enclosed in quotes:

[Commands]
"REGEDIT /S regtweaks.reg"
"cscript.exe //B //NoLogo %SystemDrive%\Install\setup.vbs"
"%SystemDrive%\Install\postsetup.cmd"

4.4 Key Points

Commands run sequentially at T-12 under the SYSTEM account.

Environment variables like %SystemDrive% and %WINDIR% are valid.

Requires OemPreinstall=Yes in the answer file.

For commands at first user logon instead, use GuiRunOnce in the answer file.

5. Explanation of OemPnPDriversPath

OemPnPDriversPath is a key in the answer file (WINNT.SIF or UNATTEND.TXT) that specifies additional folders containing Plug and Play drivers to be installed silently during setup.

These folders are typically inside the $OEM$ tree (e.g., $OEM$\Install\drivers).

Setup scans these folders and installs drivers automatically without user intervention.

Useful for adding drivers not included in the default Windows installation.

Example snippet in answer file:

[Unattended]
OemPreinstall = Yes
OemPnPDriversPath = "Install\\drivers"

6. Summary Example

Here is a concrete example of a typical folder layout and key files:

\I386
$OEM$
$$\
System32\
mytool.exe
$1\
Install\
apps.cmd
drivers\
TEXTMODE\
iastor.sys
iastor.inf
iastor.cat
CMDLINES.TXT
UNATTEND.TXT

Example CMDLINES.TXT:

[Commands]
"%SystemDrive%\Install\apps.cmd"
"REGEDIT /S %SystemDrive%\Install\regtweaks.reg"

Example UNATTEND.TXT:

[Unattended]
OemPreinstall = Yes
OemPnPDriversPath = "Install\\drivers"
TargetPath = \WINDOWS
UnattendMode = FullUnattended
FileSystem = NTFS

[GuiRunOnce]
"%SystemDrive%\Install\apps.cmd"

[MassStorageDrivers]
"Intel 82801ER SATA RAID RAID Controller" = OEM

[OEMBootFiles]
iastor.sys
iastor.inf
iastor.cat

This setup ensures:

Text-mode Setup loads mass-storage drivers from $OEM$\TEXTMODE.

Files are copied to the appropriate Windows and system locations.

Commands in CMDLINES.TXT run at T-12.

Additional scripts run at first user logon via GuiRunOnce.

References

Some Detailed Explanation Of $OEM$ Folder - MSFN

Distribution Shares and Configuration Sets Overview - Microsoft Docs

Designing Unattended Installations - Microsoft Download
