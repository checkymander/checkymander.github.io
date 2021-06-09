---
title: "Decrypting VEEAM Passwords"
categories:
  - Red Team
  - Veeam
tags:
  - Instructional
  - Mimikatz
  - DPAPI
  - Passwords
  - Privilege Escalation
---

## What is VEEAM?

VEEAM is an enterprise level backup solution that allows administrators to perform backups and restorations of assets such as Cloud Devices, Virtual Machines, and Physical Hosts. It is meant to be easy to setup, and flexible enough for any environment. While also providing reliable backups and helping support resiliency from ransomware attacks.

## So what?
Backup Solutions provide a lot of valuable information to Red Teamers and Penetration Testers, allowing them access to data they otherwise wouldn't have permission to view, and potentially information that could allow them to expand further throughout the network. Back in April of 2020 Brett DeWall ([@xbadbiddyx](https://twitter.com/xbadbiddyx)) put out a post called [From VEEAM to Domain Administrator](https://www.whiteoaksecurity.com/2020-4-14-from-veeam-to-domain-administrator/) which talks about how VEEAM backups of Domain Controllers can be used to leverage DA access in the environment. But what about cases where the VEEAM solution isn't doing DC backups or the SMB share isn't directly available to you?

In order to facilitate performing these backups, these backup solutions need some type of privileged access over the hosts they're connecting to. VEEAM Backup & Replication makes this easy by providing a Credentials Manager, which maintains a list of credentials records that you can use to connect to components in the backup infrastructure. This credentials manager can store the following items:
- Domain Credentials
- Local Credentials
- Linux Credentials
- SSH Private Keys
- AWS/GCP Access Keys
- Azure Computer Account Credentials
- Azure Storage Account Credentials

Depending on your environment this could be a gold mine of privileged accounts, Cloud, AD, and Linux credentials all in one place! The problem is, VEEAM protects these credentials from being viewed directly through the UI, only allowing the password to be updated.

![image](https://user-images.githubusercontent.com/26147220/121399946-fc125e80-c924-11eb-8562-207e7a331fbc.png)

In this post, I'm going to go over how we can gain access to these underlying credential information in order to leverage further attacks against our target.

## Finding The Credentials

First we need to identify where these credentials are being stored. 

When installing VEEAM Backup & Replication, you'll be asked to select a Microsoft SQL Server. The default value is to install and create a new instance of SQL Server on localhost, with permissions of the database granted to Administrators. Alternatively, the administrator setting up the application can specify an external MSSQL server. This information can be discovered by running the "Configuration Database Connection Settings" application. When you open it you'll see the following screen:

![image](https://user-images.githubusercontent.com/26147220/121400943-2b759b00-c926-11eb-9fe7-f806a0dffc62.png)

Hitting "Next" will attempt to connect to the database to validate connnectivity and perform other checks, so avoid clicking that. This provides us everything we need to connect to the database, the location, the authentication type, and the instance name.

With this information we can use a tool such as SQL Server Management Studio to connect to the databse to view configuration information. Open up SSMSS and enter in the required information.

![image](https://user-images.githubusercontent.com/26147220/121401612-e56d0700-c926-11eb-8c19-33029d4c1c98.png)

There are two ways you can pull the credential information from the database. The first is to navigate to the VeeamBackup database and look for the credentials table. It's easy to find as it's just named "dbo.Credentials". Alternatively we can run the following SQL Query:

`SELECT user_name,password FROM VeeamBackup.dbo.Credentials;`

After running this query, you'll see the following output:

![image](https://user-images.githubusercontent.com/26147220/121402412-db97d380-c927-11eb-9eaf-58b4c9d55757.png)

Awesome! It looks base64 encoded, let's see if we can decode it!

![image](https://user-images.githubusercontent.com/26147220/121402509-f9653880-c927-11eb-90c2-e284e84b88b1.png)

hmm...not quite.

## Decrypting the Credentials

Looking at the [documentation](https://helpcenter.veeam.com/docs/agentforwindows/configurator/encryption.html?ver=50), we can see the user credentials get encrypted before they're stored into the database. Unfortunately, it doesn't show us the type of encryption that is being used to protect these passwords.

After some further googling, I came across the following KB article [KB2327](https://www.veeam.com/kb2327) which states:

```
All sensitive information, such as user credentials and encryption keys are stored in the configuration database are encrypted with machine-specific private key of backup server. 
Accordingly, a newly installed backup server will not be able to decrypt such information if attached to the existing database.
```

I know this one, DPAPI!

![image](https://media.giphy.com/media/6JvWR1rDbseeQ/giphy.gif)

For anyone not already familiar with DPAPI refer to [this](https://www.harmj0y.net/blog/redteaming/offensive-encrypted-data-storage-dpapi-edition/) post by Will Shroeder ([Harmj0y](https://twitter.com/harmj0y))

The quick and dirty is, DPAPI is a set of crypto functions that try to make it easier to handle key management for encrypting and decrypting data. It's commonly used by applications such as Google Chrome to securely store saved website logins on disk, and in this case VEEAM Backup & Replication also uses it to protect credentials in the database.

Now that we know how it's encrypted, we can figure out how to decrypt it. Luckily for us there are two tools that have save us the effort from having to implement the code ourselves. I'll show you how to decrypt this data using both tools.

### Mimikatz

The first is ol' faithful, Mimikatz.

First we need to convert the base64 blob into a binary file that can be used by mimkatz. This can be done pretty easily in C# using the following code:

```
using System;

namespace txt2bin
{
    class Program
    {
        static void Main(string[] args)
        {
            string dpapistring = "blobstringgoeshere";

            byte[] data = Convert.FromBase64String(dpapistring);

            System.IO.File.WriteAllBytes("payload.bin", data);
        }
    }
}
```

Take the outputed payload.bin and place it on disk. If you've already exported the DPAPI keys from the backup server, all of these steps can be performed offline providing the apppropriate additional flags. If not, the commands will need to be run in the context of a local administrator on the VEEAM server itself.

Run the following command to decrypt the DPAPI blob:

`dpapi::blob /in:"C:\path\to\payload.bin" /unprotect `

![image](https://user-images.githubusercontent.com/26147220/121405315-e43dd900-c92a-11eb-9ff2-1ad69c83ceb7.png)

This should be successful, and you will seee the data field with some hex after it. This is the hexadecimal equivalent of the password for the account that we decrypted. Convert this to ASCII and you're good to go!

![image](https://user-images.githubusercontent.com/26147220/121405431-0cc5d300-c92b-11eb-80a7-442be010f068.png)


### SharpDPAPI

SharpDPAPI is a little bit easier to use to decrypt the password as we can provide the base64 blob to it directly. In addition, depending on your c2 of choice, it's pretty much guaranteed that you'll be able to execute .NET executables in memory. To decrypt the password using SharpDPAPI run the following command:

`SharpDPAPI.exe blob /target:"base64blobgoeshere" /unprotect`

You should see output like the following:

![image](https://user-images.githubusercontent.com/26147220/121405810-7a71ff00-c92b-11eb-9731-19737be585a7.png)

In both cases the hex value that's returned is the same value and can be decrypted to our password of `Sup3rS3cr3t!`j

## Prevention

As this is intended functionality and is required for the VEEAM backup server to work, the main protection against this technique is to limit the amount of people who have access to the VEEAM Backup Server. In addition, ensure proper access controls are enforced on the MSSQL databse itself to prevent users from reading the protected data in the first place.

## Conclusion

In conclusion, backup solutions continue to be a gold mine of information for Red Teamers to succeed in assessments. By being able to extract these credentials, and correlating the credentials to the machines in the VEEAM Backup & Replication inventory, we can expand our access in order to better achieve our objectives.
