---
layout: single
title: Administrator - Hack The Box
excerpt: Este es el write-up de Administrator, una máquina Windows de dificultad media centrada en el abuso de Active Directory, donde los permisos ACL inadecuados, la reutilización de credenciales y el Kerberoasting conducen al compromiso total del dominio.
date: 2024-11-13
classes: wide
header:
  teaser: /assets/images/htb-writeup-administrator/administrator-logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - AD
  - kerberoasting
  - ACL
---

Empezar

### Tools/Blogs used

-

## Quick summary

- There's an SQL injection in the generic products inventory page
- Using the SQL injection in MSSQL, we can trigger an SMB connection back to us and get the NTLM hash with responder.py
- The credentials are used to gain access to a restricted PS session through the Web Powershell interface
- The Ubiquiti Unifi Video service has weak file permissions and allow us to upload an arbitrary file and execute it as SYSTEM
- A reverse shell executable is compiled, uploaded and executed to get SYSTEM access

## Detailed steps

### Nmap
