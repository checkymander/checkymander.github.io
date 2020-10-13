---
title: "Red Teaming Power Users Pt. 1 - Terminals Programs"
last_modified_at: 2016-03-09T16:20:02-05:00
categories:
  - Red Team
  - Hacking
  - Enumeration
tags:
  - PuTTY
  - MTPuTTY
  - MobaXTerm
---

While performing a Red Team, you may occasionally find yourself on a Power Users box, with no obvious ways to escalate privileges. You could start scanning the network to try to identify boxes where your current user context DOES have administrator privilege, then move laterally to that location. But what about if you want to limit the amount of network traffic you're generating in order to avoid detection by the Blue Team? In this series of posts, I'm going to discuss different technologies in use by Power Users and how you can use them for different attacks.

Part 1 is going to discuss common Terminal applications that you could use for Enumeration, Lateral Movement, and Privilege Escalation. Specifically we're going to discuss the following applications:
* PuTTY
* MTPuTTY
* MobaXTerm
* KiTTY

### PuTTY
[PuTTY](https://www.putty.org/) is a pretty standard SSH and telnet client. It's one of the most popular clients out there and will likely be the one you see on your assessments. Like the other applications in this post, it has the ability to save user sessions for easier access. Each saved session can contain multiple configuration options which will be dependant on what your user decides to save. From an enumeration standpoint, we're mainly going to be interested in uncovering the following items:
* Username
* Hostname/Port
* Credential Information (Password/Keys)

The good news for us is, that it's super easy to gain access to all the information we need. PuTTY while being easy to use, stores all of it's information in cleartext in the registry. Each session is stored under a registry key located at `HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\`

<picture of reg>
  
The easiest way to enumerate this information is to use [SessionGopher](https://github.com/Arvanaghi/SessionGopher) by [@Arvanaghi](https://twitter.com/arvanaghi), however, Powershell is quickly becoming more difficult to use in a stealthy way. Most people know I'm big into .NET development so here's some quick code I made to enumerate the information we need:

```
using System;
using Microsoft.Win32;

namespace PuTTYCheck
{
    class Program
    {
        static void Main(string[] args)
        {
            var reg = Registry.CurrentUser.OpenSubKey(@"Software\SimonTatham\PuTTY\Sessions\");
            foreach (string subKeyName in reg.GetSubKeyNames())
            {
                try
                {
                    using (RegistryKey
                        tempKey = reg.OpenSubKey(subKeyName))
                    {
                        Console.WriteLine(tempKey.Name);
                        Console.WriteLine("HostName: {0}", tempKey.GetValue("HostName"));
                        Console.WriteLine("Protocol: {0}", tempKey.GetValue("Protocol"));
                        Console.WriteLine("PortNumber: {0}", tempKey.GetValue("PortNumber"));
                        Console.WriteLine("ProxyPort: {0}", tempKey.GetValue("ProxyPort"));
                        Console.WriteLine("ProxyUsername: {0}", tempKey.GetValue("ProxyUsername"));
                        Console.WriteLine("ProxyPassword: {0}", tempKey.GetValue("ProxyPassword"));
                        Console.WriteLine("ProxyTelnetCommand: {0}", tempKey.GetValue("ProxyTelnetCommand"));
                        Console.WriteLine("UserName: {0}", tempKey.GetValue("UserName"));
                        Console.WriteLine("UserNameFromEnvironment: {0}", tempKey.GetValue("UserNameFromEnvironment"));
                        Console.WriteLine("LocalUserName: {0}", tempKey.GetValue("LocalUserName"));
                        Console.WriteLine("PublicKeyFile: {0}", tempKey.GetValue("PublicKeyFile"));
                        Console.WriteLine("RemoteCommand: {0}", tempKey.GetValue("RemoteCommand"));
                        Console.WriteLine("SSHManualHostKeys: {0}", tempKey.GetValue("SSHManualHostKeys"));
                        Console.WriteLine("===================================================");
                    }
                }
                catch (Exception e)
                {
                    Console.WriteLine(e.Message);
                }
            }
        }
    }
}
```

### MTPuTTY



### MobaXTerm



### KiTTY
