---
title: Understanding Application Control event IDs (Windows 10)
description: Learn what different Windows Defender Application Control event IDs signify.
keywords: security, malware
ms.assetid: 8d6e0474-c475-411b-b095-1c61adb2bdbb
ms.prod: m365-security
ms.mktglfcycl: deploy
ms.sitesec: library
ms.pagetype: security
ms.localizationpriority: medium
audience: ITPro
ms.collection: M365-security-compliance
author: jsuther1974
ms.reviewer: isbrahm
ms.author: dansimp
manager: dansimp
ms.date: 3/17/2020
ms.technology: mde
---

# Understanding Application Control events

A Windows Defender Application Control (WDAC) policy logs events locally in Windows Event Viewer in either enforced or audit mode. These events are generated under two locations:

 - Event IDs beginning with 30 appear in Applications and Services logs – Microsoft – Windows – CodeIntegrity – Operational

 - Event IDs beginning with 80 appear in Applications and Services logs – Microsoft – Windows – AppLocker – MSI and Script

## Microsoft Windows CodeIntegrity Operational log event IDs

| Event ID | Explanation |
|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 3076 | Audit executable/dll file |
| 3077 | Block executable/dll file |
| 3089 | Signing information event correlated with either a 3076 or 3077 event. One 3089 event is generated for each signature of a file. Contains the total number of signatures on a file and an index as to which signature it is.<br>Unsigned files will generate a single 3089 event with TotalSignatureCount 0. Correlated in the "System" portion of the event data under "Correlation ActivityID". |
| 3099 | Indicates that a policy has been loaded |

## Microsoft Windows Applocker MSI and Script log event IDs

| Event ID | Explanation |
|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 8028 | Audit script/MSI file generated by Windows LockDown Policy (WLDP) being called by the scripthosts themselves. Note: there is no WDAC enforcement on 3rd party scripthosts. |
| 8029 | Block script/MSI file |
| 8038 | Signing information event correlated with either a 8028 or 8029 event. One 8038 event is generated for each signature of a script file. Contains the total number of signatures on a script file and an index as to which signature it is. Unsigned script files will generate a single 8038 event with TotalSignatureCount 0. Correlated in the "System" portion of the event data under "Correlation ActivityID". | |

## Optional Intelligent Security Graph (ISG) or Managed Installer (MI) diagnostic events

If either the ISG or MI is enabled in a WDAC policy, you can optionally choose to enable 3090, 3091, and 3092 events to provide additional diagnostic information. 

| Event ID | Explanation |
|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 3090 | Allow executable/dll file |
| 3091 | Audit executable/dll file |
| 3092 | Block executable/dll file |

3090, 3091, and 3092 events are generated based on the status code of whether a binary passed the policy, regardless of what reputation it was given or whether it was allowed by a designated MI. The SmartLocker template which appears in the event should indicate why the binary passed/failed. Only one event is generated per binary pass/fail. If both ISG and MI are disabled, 3090, 3091, and 3092 events will not be generated.

### SmartLocker template

Below are the fields which help to diagnose what a 3090, 3091, or 3092 event indicates.

| Name | Explanation |
|-------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| StatusCode | STATUS_SUCCESS indicates a binary passed the active WDAC policies. If so, a 3090 event is generated. If not, a 3091 event is generated if the blocking policy is in audit mode, and a 3092 event is generated if the policy is in enforce mode. |
| ManagedInstallerEnabled | Policy trusts a MI |
| PassesManagedInstaller | File originated from a trusted MI |
| SmartlockerEnabled | Policy trusts the ISG |
| PassesSmartlocker | File had positive reputation |
| AuditEnabled | True if the policy is in audit mode, otherwise it is in enforce mode |

### Enabling ISG and MI diagnostic events

In order to enable 3091 audit events and 3092 block events, you must create a TestFlags regkey with a value of 0x100. You can do so using the following PowerShell command:

```powershell
reg add hklm\system\currentcontrolset\control\ci -v TestFlags -t REG_DWORD -d 0x100
```
    
In order to enable 3090 allow events as well as 3091 and 3092 events, you must instead create a TestFlags regkey with a value of 0x300. You can do so using the following PowerShell command:

```powershell
reg add hklm\system\currentcontrolset\control\ci -v TestFlags -t REG_DWORD -d 0x300
```
