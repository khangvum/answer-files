# Answer Files

An **_automated operating system_** (**_OS_**) **_deployment solution_** utilizing **_answer files_**. This solution streamlines the **_installation_** of **_Windows_** and **_Linux systems_** through unattended files, ensuring **_consistent_**, **_repeatable_**, and **_hands-free_** deployment across physical and virtual environments.

## Features

-   **_Unattended installation_** for **_Windows 10_**, **_Windows 11_**, **_Windows Server 2022_**, and **_Windows Server 2025_**.
-   **_OpenSSH Server_** pre-installed to streamline **_post-deployment configuration_** (_e.g.,_ [khangvum/windows-config](https://github.com/khangvum/windows-config)).
-   **_VMware Tools_** installed on supported **_virtual machines_** (**_VMs_**).
-   **_Remote Desktop Services_** (formerly **_Terminal Services_**) enabled.

## Filesystem Hierarchy

```
├── sources
│   └── $oem$
│       └── $$
│           └── Setup
│               └── Scripts
│                   ├── OpenSSH-Server-x64.bat
│                   ├── OpenSSH-Server-x64.msi
│                   └── VMware-Tools-x64.exe
└── autounattend.xml
```

## Troubleshooting

1.  **Windows 11 Unattended Installation Fails on Version 24H2**

-   Windows 11 24H2 uses a **_[new setup process](https://www.elevenforum.com/t/w11-24h2-and-old-installation-setup.25706/post-476855)_**:

Version             |Process
:------------------:|:-------------------------
**23H2 and earlier**|`X:\setup.exe` is invoked.
**24H2**            |`X:\Sources\setup.exe` is executed, which runs `SetupHost.exe`, which in turn invokes `SetupPrep.exe`.

-   The solution is to **_[explicitly invoke the legacy setup process](https://www.elevenforum.com/t/w11-24h2-and-old-installation-setup.25706/post-476942)_**:

    ```xml
    <settings pass="windowsPE">
        <component name="Microsoft-Windows-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <RunSynchronous>
                <RunSynchronousCommand wcm:action="add">
                    <Description>Switch to legacy Setup</Description>
                    <Order>1</Order>
                    <Path>reg add "HKEY_LOCAL_MACHINE\SYSTEM\Setup" /v CmdLine /t REG_SZ /d "X:\sources\setup.exe" /f</Path>
                </RunSynchronousCommand>
            </RunSynchronous>
        </component>
    </settings>
    ```

2.  **OpenSSH Server Installation Fails on Windows Server 2025**

-   Starting with Windows Server 2025, OpenSSH Server is **_[installed by default](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=gui&pivots=windows-server-2025)_** and is no longer labelled as a preview feature. As a result, the **_firewall rule_** created during installation differs compared to other Windows OS:

OS                      |Firewall Rule
:----------------------:|:--------------------------------
**Other Windows OS**    |OpenSSH SSH Server Preview (sshd)
**Windows Server 2025** |OpenSSH SSH Server (sshd)

-   Update the [OpenSSH-Server-x64.bat](Windows%20Server%202025%20Standard/sources/$oem$/$$/Setup/Scripts/OpenSSH-Server-x64.bat) batch script for Windows Server 2025 to reflect the **_new firewall rule name_**.