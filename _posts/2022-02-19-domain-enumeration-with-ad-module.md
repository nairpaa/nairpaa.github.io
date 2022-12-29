---
title: Part 2 - Domain Enumeration with AD Module
date: 2022-02-19 12:00:22 +0700
categories: [Active Directory, Post Compromise Enumeration]
tags: [active directory, windows, admodule, enumeration]     # TAG names should always be lowercase
author:
  name: Nairpaa
  link: https://nairpaa.github.io
---

### Tujuan 

- *Post compromise enumeration*

### Prasyarat

- Memiliki kredensial user domain

### Tools

- [AD Module](https://github.com/samratashok/ADModule)

---

## Enumerasi AD dengan AD Module

- **Get Current Domain:** `Get-ADDomain`
- **Enum Other Domains:** `Get-ADDomain -Identity <Domain>`
- **Get Domain SID:** `Get-DomainSID`
- **Get Domain Controlers:** 
  ```powershell
  Get-ADDomainController
  Get-ADDomainController -Identity <DomainName>
  ```
- **Enumerate Domain Users:** 
  ```powershell
  Get-ADUser -Filter * -Identity <user> -Properties *

  #Get a spesific "string" on a user's attribute
  Get-ADUser -Filter 'Description -like "*wtver*"' -Properties Description | select Name, Description
  ```
- **Enum Domain Computers:** 
  ```powershell
  Get-ADComputer -Filter * -Properties *
  Get-ADGroup -Filter * 
  ```
- **Enum Domain Trust:** 
  ```powershell
  Get-ADTrust -Filter *
  Get-ADTrust -Identity <DomainName>
  ```
- **Enum Forest Trust:** 
  ```powershell
  Get-ADForest
  Get-ADForest -Identity <ForestName>

  #Domains of Forest Enumeration
  (Get-ADForest).Domains
  ```
 - **Enum Local AppLocker Effective Policy:**
  ```powershell
  Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
  ```

---

## Referensi

- [Active Directory Exploitation Cheat Sheet](https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet#domain-enumeration)