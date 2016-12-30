---
inFeed: true
description: '17 MAY 2016 on defense, technical, workflow, devops, ansible'
dateModified: '2016-12-30T18:17:05.380Z'
datePublished: '2016-12-30T18:17:05.831Z'
title: 'Poor Man''s Windows Security Metrics with Ansible, Powershell, and Bash'
author: []
publisher: {}
via: {}
hasPage: true
sourcePath: >-
  _posts/2016-12-30-poor-mans-windows-security-metrics-with-ansible-powershell.md
starred: false
datePublishedOriginal: '2016-12-30T18:13:03.918Z'
url: poor-mans-windows-security-metrics-with-ansible-powershell/index.html
_type: Article

---
# Poor Man's Windows Security Metrics with Ansible, Powershell, and Bash

17 MAY 2016 on [defense][0], [technical][1], [workflow][2], [devops][3], [ansible][4]

_Work, but don't work too much. Eat, but don't eat too much._

---

I'm doing a thing for work and I needed to install a shit load of software onto a shit load of Windows machines. I automated this process with [Terraform][5] but as everything was installing I would want to periodically check on it.

To accomplish this I installed Ansible on my Macbook

$ brew install ansible

...populated the "hosts" config file...

\[windows\] 1.1.1.1 2.2.2.2 3.3.3.3 \[windows:vars\] ansible\_ssh\_user=administrator ansible\_ssh\_pass=whatever ansible\_ssh\_port=5985 ansible\_connection=winrm

...and wrote this hilariously disgusting one-liner:

$ ansible -i hosts -**m** raw -a "powershell -Command Get-WmiObject -Class Win32\_Product" all | **grep** "Name " | **sort** | uniq -c | **sort** -n | gtac

...which yielded the following beautiful ordered and descending list of software packages installed on each of these machines.

99 Name : aws-cfn-bootstrap 99 Name : EC2ConfigService 99 Name : AWS Tools for Windows 99 Name : AWS PV Drivers 51 Name : Python 2.7.11 (64-bit) 16 Name : calibre 64bit 11 Name : Microsoft Visual C++ 2010 x86 Redistributable - 10.0.40219 11 Name : Microsoft Visual C++ 2010 x64 Redistributable - 10.0.40219 8 Name : Microsoft Visual C++ 2013 x64 Minimum Runtime - 12.0.21005 8 Name : Microsoft Visual C++ 2013 x64 Additional Runtime - 8 Name : CDBurnerXP (64 bit) 7 Name : FileBot 7 Name : Brackets 6 Name : swMSM 5 Name : Microsoft Visual C++ 2008 Redistributable - x86 5 Name : Microsoft Visual C++ 2008 Redistributable - x64 5 Name : Java SE Development Kit 8 **Update** 92 (64-bit) 5 Name : Dropbox **Update** Helper 5 Name : Classic Shell 4 Name : iTunes 3 Name : Far Manager 3 x64 3 Name : Bonjour 3 Name : Apple Mobile Device Support 3 Name : Apple Application Support 2 Name : Strawberry Perl (64-bit) 2 Name : Microsoft Visual C++ 2010 x64 Redistributable - 10.0.30319 2 Name : Google **Update** Helper 2 Name : Evernote v. 6.0.6 2 Name : AWS Command Line Interface 1 Name : grepWin x64 1 Name : Octopus Deploy Tentacle 1 Name : Microsoft ASP.NET Web Pages 2 Runtime 1 Name : Microsoft ASP.NET Web Pages 1 Name : Microsoft ASP.NET MVC 4 Runtime 1 Name : Microsoft ASP.NET MVC 3 1 Name : Jenkins 1.605 1 Name : Jenkins 1.515 1 Name : Java SE Development Kit 8 **Update** 72 (64-bit) 1 Name : Java SE Development Kit 8 **Update** 60 (64-bit) 1 Name : Java SE Development Kit 8 **Update** 5 (64-bit) 1 Name : Java Auto Updater 1 Name : Java 8 **Update** 25 (64-bit) 1 Name : Java 8 **Update** 20 (64-bit) 1 Name : ConEmu 160504.x64 1 Name : ConEmu 160301.x64 1 Name : ConEmu 160222.x64 1 Name : ConEmu 160218.x64 1 Name : ConEmu 160217.x64 1 Name : ConEmu 160204.x64 1 Name : ConEmu 160124.x64 1 Name : ConEmu 151224.x64 1 Name : ConEmu 151210.x64 1 Name : ConEmu 151208.x64 1 Name : ConEmu 151207.x64 1 Name : ConEmu 151129.x64 1 Name : ConEmu 151126.x64 1 Name : ConEmu 151029.x64 1 Name : ConEmu 151025.x64 1 Name : ConEmu 151015.x64 1 Name : ConEmu 150913.x64 1 Name : ConEmu 150821.x64 1 Name : ConEmu 150728.x64 1 Name : ConEmu 150727.x64 1 Name : ConEmu 150504.x64 1 Name : ChocolateyGUI 0.12.4.0 1 Name : ChocolateyGUI 0.12.3.0 1 Name : ChocolateyGUI 0.12.2.0 1 Name : ChocolateyGUI 0.12.1.0 1 Name : ChocolateyGUI 0.12.0.0 1 Name : Chef Client v12.4.1 1 Name : Chef Client v11.16.4 1 Name : Chef Client v11.14.2 1 Name : Chef Client v11.12.4 1 Name : Blender 1 Name : Adobe Flash Player 21 NPAPI 1 Name : ActivePerl 5.22.0 Build 2200 (64-bit) 1 Name : 7-Zip 9.22 (x64 edition)

We can do the same thing with running processes (again, no agent needed).

$ ansible -i hosts -m raw -a "tasklist" all | cut -d' ' -f1 | grep "\\.exe" | sort | uniq -c | sort -n

which yields the following results...

1 AIMP.Setup.exe 1 AdobeAIRInstall.exe 1 CuteWriterInstall.exe 1 Everything.exe 1 adobereaderInstall.exe 1 aimpInstall.exe 1 clover.exe 1 dittoInstall.exe 1 java.exe 1 jenkins.exe 1 jre-8u20-windows-x64.exe 1 jre-8u25-windows-x64.exe 2 Setup.exe 3 AutoHotkeyU32.exe 3 gpg4winInstall.exe 3 mDNSResponder.exe 3 taskeng.exe 4 dirmngr.exe 4 iPodService.exe 4 iTunesHelper.exe 5 ClassicStartMenu.exe 8 python.exe 9 Agile1pService.exe 9 git.installInstall.exe 12 Dropbox.exe 14 adwcleaner.exe 15 AutoHotkey.exe 24 setup.exe 52 choco.exe 99 Ec2Config.exe 99 KorIME.exe 99 LiteAgent.exe 99 LogonUI.exe 99 lsass.exe 99 msdtc.exe 99 services.exe 99 smss.exe 99 tasklist.exe 99 vds.exe 99 wininit.exe 99 winlogon.exe 101 msiexec.exe 133 cmd.exe 147 winrshost.exe 150 conhost.exe 162 powershell.exe 198 WmiPrvSE.exe 198 csrss.exe 1089 svchost.exe

...and finally, even with a handfull of Windows autorun items.

$ ansible -i host -**m** raw -a "powershell -Command 'Get-CimInstance Win32\_StartupCommand | Select-Object Name, command, Location, User | Format-List'" all | **grep** "command \\|Location " | **sort** | uniq -c | **sort** -n | gtac

...which yields

99 command : Ec2WallpaperInfo.url 99 Location : Common Startup 7 Location : HKLM\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run 5 command : "C:\\Program Files\\Classic Shell\\ClassicStartMenu.exe" -autorun 2 command : C:\\Program Files\\Greenshot\\Greenshot.exe 2 command : C:\\Program Files\\Ditto\\Ditto.exe 1 Location : HKU\\S-1-5-21-516659210-2350455428-2211046881-500\\SOFTWARE\\Microsoft\\ 1 Location : HKU\\S-1-5-21-1601683224-898520616-3470010867-500\\SOFTWARE\\Microsoft\\

The reason this is cool is because it doesn't require any sort of agent on the Windows endpoints. Just ensure you have Powershell remoting enabled (WinRM), populate the list of machines in your host list, configure credentials and bam. Easy like Sunday morning.

Enjoy!

--Andrew

[0]: http://morris.sc/tag/defense/
[1]: http://morris.sc/tag/technical/
[2]: http://morris.sc/tag/workflow/
[3]: http://morris.sc/tag/devops/
[4]: http://morris.sc/tag/ansible/
[5]: https://terraform.io/