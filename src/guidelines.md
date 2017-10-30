# GUIDELINES

The following security guidelines apply to servers, components, accounts, and communications protocols that constitute an ACP instance and the underlying web servers and OS instance an ACP instance resides on. The steps to implement these guidelines vary by hosting environment and provider, but the concepts are the same. 

## File System Access Controls

The following section lists a series of recommended access controls to limit user file system permission on both Windows Web/Application Servers and Platform Repository shares. In turn, the permissions granted to Platform services during install/setup will allow the Platform to operate normally while adding both limited and necessary access for guest application service accounts at deploy time. By following the list of procedures below, you can ensure the removal of permissions for nonessential user access to files and folders.

### Windows Web / App Servers

#### .NET User Interface Workloads

By default, a .NET User Interface (UI) will run as the individual Application Pool Identity user. The Platform will provide the IIS Application Pool user with Full Control access to the application’s SiteData directory. 

#### Running Workloads as Virtual Accounts

By default, the Platform will run guest applications under Windows Virtual Accounts, a Windows Server feature that enhances isolation and manageability of network applications. For each new guest application deployment to the Platform, a new Virtual Account will be created to manage the instance. To run WCF Services, you will have to make sure `NT SERVICE\ALL SERVICES` have the "Log on as a Service" permission. The Platform will only provide Full Control to the workload’s Launchpad folder once the workload is created. 

It is recommended that you run your applications as Virtual Accounts whenever possible to prevent creating and managing domain accounts.

#### Running Workloads as Domain Accounts

Instead of running components as Virtual Accounts, the Platform can be configured to run some or all guest application components under unique domain service accounts. In some cases, this configuration is useful when an application needs to authenticate to an external data center resource using a domain account. Developers can specify guest application component under an unique domain account in directly in the Developer Portal.

If you run a WCF Service or a Windows Service under a domain account, there are no additional permissions that you need to grant these services. The Platform will only provide Full Control to the workload’s Launchpad folder once the workload is created. 

If you want to run an IIS workload (.NET UI) as a service account, you need to provide the following permissions:

1.	“Log on as a batch job” or “Log on as a service”
2.	“Impersonate a client after authentication”

#### Grant Permissions to the Apprenda Admin User

According to both Pre-Installation and Permissions Requirements documentation (see Section 5.0), the Apprenda Admin domain user requires “Allow Log on Locally” permissions to all Windows nodes, and Read/Write Access to the Platform Repository. In situations where the Platform UI Manager will restart (e.g. Platform upgrade or server reboot), the Platform will prevent the Apprenda Admin user from accessing the C:\ApprendaPlatform\SiteData C:\ApprendaPlatform\Container folders, which the user requires. 

In order to prevent this from permission removal from occurring, we recommend creating a specific group for the Apprenda Admin user and setting the required permissions at the group level. This will ensure that the user will have access to the C:\ApprendaPlatform\SiteData C:\ApprendaPlatform\Container folders upon UI Manager restarts. Please refer to additional documentation on Platform service accounts and permissions identified in Section 4.0.

#### Harden the ApprendaPlatform Folder

1.	Disable Inheritance
2.	Convert inherited permissions into explicit permissions on this object
3.	Remove all permissions for users
4.	Add Full Control for Group containing the Domain Admin
5.	Add Full Control for Group containing Apprenda Admin Account

The Platform via the Apprenda Admin Account will grant the following permissions within the Apprenda Platform folder:

1.	“Read & Execute” permissions to IIS AppPool\catchall for C:\ApprendaPlatform\SiteData\Catchall.
2.	Grant “Read & Execute” permissions to IIS AppPool\apprenda and IUSR for C:\ApprendaPlatform\SiteData\Home.

#### APPRENDAPLATFORM\PROCESSLAUNCHER-V<insert platform full version here>

* Add "Read & Execute" Permissions to the Users group

#### Remove Access for the Users Groups in the Following Locations

* C:\installs
* C:\Sysprep
* C:\Temp

#### Mark Read-Only for the Users Group in the Following Locations

* C:\sxs
* C:\XenTools
* C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Temporary ASP.NET Files
* C:\Windows\Microsoft.NET\Framework\v2.0.50727\Temporary ASP.NET Files
* C:\Windows\Microsoft.NET\Framework\v4.0.30319\Temporary ASP.NET Files
* C:\Windows\Microsoft.NET\Framework64\v2.0.50727\Temporary ASP.NET Files

NOTE: In ACP versions 7.1.1 and prior, the Users Group required "Read & Execute" Permissions (on all folders and subfolders) and the IIS_ISURS Group required “Read Attributes” (to the folder only) for C:\ApprendaPlatform\SiteData\developer. These permission requirements are no longer needed for ACP versions 7.2.0 and beyond.

### Repository

#### Root of the Drive Hosting the Repository

1.	Disable Inheritance
2.	Convert inherited permissions into explicit permissions on this object
3.	Remove all permissions for users
4.	Add Full Control for Group containing the Domain Admin
5.	Add Full Control for Group containing Apprenda Admin Account

#### Root of the Platform Share Folder

1.	Right-click properties, Sharing, Advanced Sharing.
2.	Add full control for Groups containing Local Administrators and the Platform Admin Account

### Transport Security

In addition to controlling the file system, system administrators can also lock down nonessential, and often less secure, communications paths for services used by the Platform, guest applications, external data center resources, and clients. In order to permit communications required by the Platform and guest application, and deny nonessential paths, please navigate to Platform networking documentation identified in Section 5.0. Many of the recommended controls in this section originated from the Center for Internet Security (CIS) Benchmarks for IIS 8 https://www.cisecurity.org/cis-benchmarks/. 

#### Secure Application Pools for Sites

IIS 8.0 introduced Application Pool Identities that allows Apprenda (by default) to run guest .NET User Interfaces under unique accounts without the need to create and manage domain accounts (similar to the appeal of Virtual Accounts). 
Upon installation of IIS, the web server creates a default web site and application pool identity, which are not used in any way on the Apprenda Platform. As a result, it is recommended that you remove both the default web site and application pool identity after installing IIS.

#### Disable Less Secure Ciphers

* 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\Ciphers\DES 56/56'
* 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\Ciphers\NULL'
* 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\Ciphers\RC2 128/128'
* 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\Ciphers\RC2 40/128'
* 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\Ciphers\RC2 56/128'
* 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\Ciphers\RC4 40/128'
* 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\Ciphers\RC4 56/128'
* 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\Ciphers\RC4 64/128'
* 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\Ciphers\RC4 128/128'

#### Enable Secure Ciphers

* 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\Ciphers\AES 128/128'
* 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\Ciphers\AES 256/256'
* 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\Ciphers\Triple DES 168/168'

#### Set Cipher Order on Load Manager Servers

Do this on the Apprenda Load Manager servers.
To order cipher suites correctly, ensure the following key is set to the order below: HKLM:\SOFTWARE\Policies\Microsoft\Cryptography\Configuration\SSL\00010002

* TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384_P384
* TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256_P256
* TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384_P256
* TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256_P256
* TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA_P256
* TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA_P256
* TLS_RSA_WITH_AES_256_GCM_SHA384
* TLS_RSA_WITH_AES_128_GCM_SHA256
* TLS_RSA_WITH_AES_256_CBC_SHA256
* TLS_RSA_WITH_AES_128_CBC_SHA256
* TLS_RSA_WITH_AES_256_CBC_SHA
* TLS_RSA_WITH_AES_128_CBC_SHA
* TLS_RSA_WITH_3DES_EDE_CBC_SHA

#### Disable Less Cryptoghraphically Secure Protocols

* Ensure the following key is set to 1: HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 2.0\Server\DisabledByDefault
* Ensure the following key is set to 0: HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 2.0\Server\Enabled
* Ensure the following key is set to 1: HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0\Server\DisabledByDefault
* Ensure the following key is set to 0: HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0\Server\Enabled

#### Enable Cryptographically Secure Protocols

* HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS\Server\Enabled
* HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS\Server\DisabledByDefault
* HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS\Client\Enabled
* HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS\Client\DisabledByDefault

NOTE: The Apprenda Cloud Platform only supports external and internal Platform communication using TLS 1.1 or TLS 1.2 in Platform versions 8.1.0 and beyond. 

