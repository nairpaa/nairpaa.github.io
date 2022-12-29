---
title: 'Disable Windows Defender'
date: 2022-05-28 07:50:22 +0700
categories: [Windows, Bypass]
tags: [windows, bypass, antivirus]     # TAG names should always be lowercase
author: nairpaa
---

```powershell
PS> Set-MpPreference -DisableRealtimeMonitoring $true
```

--- 

## Referensi

- https://superuser.com/questions/1687523/enable-disable-real-time-monitoring-in-windows
- https://www.windowscentral.com/how-manage-microsoft-defender-antivirus-powershell-windows-10