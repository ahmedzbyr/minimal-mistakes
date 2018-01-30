---
title: Bootstrap Windows - knife-windows - Chef
category: ['Windows', 'Chef', 'Chef-Client']
tags: ['windows', 'chef', 'chef-client']
---
knife-windows plugin adds additional functionality to the Chef Knife CLI tool for configuring / interacting with nodes running Microsoft Windows.

* Bootstrap of nodes via the [Windows Remote Management (WinRM)](http://msdn.microsoft.com/en-us/library/aa384426\(v=VS.85\).aspx) or SSH protocols
* Remote command execution using the WinRM protocol
* Utilities to configure WinRM SSL endpoints on managed nodes

Few Good Helpful Links.

* [Knife Windows `git-src-location`](https://github.com/chef/knife-windows)
* [Awesome Blog Post by `Matt Wrock` - For More Trouble Shooting Information](http://www.hurryupandwait.io/blog/understanding-and-troubleshooting-winrm-connection-and-authentication-a-thrill-seekers-guide-to-adventure)

## Installing `gems`

Installing necessary gem for the windows deployments.

~~~ ruby
chef gem install winrm
chef gem install knife-windows
~~~

## Setting up

- Run `Enable-PSRemoting` (Use a `Administrator` PowerShell)
- Open the firewall with: `netsh advfirewall firewall add rule name="WinRM-HTTP" dir=in localport=5985 protocol=TCP action=allow` [On a Test Machine Disable Firewall till you test chef bootstrap]

Run these commands:

~~~ ruby
winrm set winrm/config/client/auth '@{Basic="true"}'
winrm set winrm/config/service/auth '@{Basic="true"}'
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
~~~

NOTE : Above settings are for Testing only.`NOT FOR PRODUCTION`.

## Test Connectivity From Chef-Workstation

use command below

~~~ ruby
telnet 172.22.2.222 5985
~~~

Output would be similar to this.
We are able to reach `5985` port from our workstation.

~~~ ruby
┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/hepsiburada/chef-repo]
└─▪ telnet 172.22.2.222 5985
Trying 172.22.2.222...
Connected to 172.22.2.222.
Escape character is '^]'.
^CConnection closed by foreign host..
~~~

Also test the command below.

~~~ ruby
┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/mychefserver/chef-repo]
└─▪ knife wsman test 172.22.2.222 -m
Connected successfully to 172.22.2.222 at http://172.22.2.222:5985/wsman.
~~~

### Try 1. [Failed]

Using `Administrator` account.

~~~ ruby
┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/mychefserver/chef-repo/cookbooks/init-setup]
└─▪ knife bootstrap windows winrm 172.16.2.35 --winrm-user Administrator --winrm-password 'Zz_12345@123' --node-name nagios_test_windows_client --run-list 'recipe[chef-client]'
Node nagios_test_windows_client exists, overwrite it? (Y/N) Y
Client nagios_test_windows_client exists, overwrite it? (Y/N) Y
Creating new client for nagios_test_windows_client
Creating new node for nagios_test_windows_client

Waiting for remote response before bootstrap.ERROR: Failed to authenticate to 172.16.2.35 as Administrator
Response: WinRM::WinRMAuthorizationError
Hint: Make sure to prefix domain usernames with the correct domain name.
Hint: Local user names should be prefixed with computer name or IP address.
EXAMPLE: my_domain\user_namer
~~~

### Try 2. [Failed]

Using `CHEF-WINDOWS-01\chef` Account with `-winrm-ssl-verify-mode verify_none` option.

~~~ ruby
┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/mychefserver/chef-repo]
└─▪ knife bootstrap windows winrm 172.22.2.222 --winrm-user 'CHEF-WINDOWS-01\chef' --winrm-password 'Nagios2234' --node-name nagiosxi_test_windows_client --winrm-ssl-verify-mode verify_none -y --run-list 'recipe[chef-client]'
Creating new client for nagiosxi_test_windows_client
Creating new node for nagiosxi_test_windows_client

Waiting for remote response before bootstrap.ERROR: Failed to authenticate to 172.22.2.222 as CHEF-WINDOWS-01\chef
Response: WinRM::WinRMAuthorizationError
Hint: Make sure to prefix domain usernames with the correct domain name.
Hint: Local user names should be prefixed with computer name or IP address.
EXAMPLE: my_domain\user_namer
~~~

### Try 3. [Failed]

Using `CHEF-WINDOWS-01\chef` using `--winrm-authentication-protocol basic` option.

~~~ ruby
┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/mychefserver/chef-repo]
└─▪ knife bootstrap windows winrm 172.22.2.222 --winrm-user 'CHEF-WINDOWS-01\chef' --winrm-password 'Nagios2234' --node-name nagiosxi_test_windows_client --winrm-authentication-protocol basic -VV -y --run-list 'recipe[chef-client]'
INFO: Using configuration from /home/ahmed/work/mychefserver/chef-repo/.chef/knife.rb
DEBUG: Looking for key winrm_authentication_protocol and found value basic
DEBUG: Looking for key winrm_transport and found value plaintext
ERROR: Validatorless bootstrap over unsecure winrm channels could expose your key to network sniffing
~~~

### Try 4. [Failed]

Using `Administrator` after changing password on the windows machine.

~~~ ruby
┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/mychefserver/chef-repo]
└─▪ knife bootstrap windows winrm 172.16.2.35 --winrm-user Administrator --winrm-password 'Nagios2234' --node-name nagios_test_windows_client --run-list 'recipe[chef-client]' -V -y
INFO: Using configuration from /home/ahmed/work/mychefserver/chef-repo/.chef/knife.rb
INFO: HTTP Request Returned 404 Object Not Found: error
INFO: HTTP Request Returned 404 Object Not Found: error
Creating new client for nagios_test_windows_client
Creating new node for nagios_test_windows_client
INFO: HTTP Request Returned 404 Object Not Found: error

Waiting for remote response before bootstrap.ERROR: Failed to authenticate to 172.16.2.35 as Administrator
Response: WinRM::WinRMAuthorizationError
Hint: Make sure to prefix domain usernames with the correct domain name.
Hint: Local user names should be prefixed with computer name or IP address.
EXAMPLE: my_domain\user_namer
~~~


### Try 5. [SUCCESS]

Adding `--winrm-ssl-verify-mode verify_none` option.
Little more Verbose so that we get to know what is happing under the hood.

~~~ ruby

┌─[ahmed][zubair-HP-ProBook][±][master ✓][~/work/mychefserver/chef-repo]
└─▪ knife bootstrap windows winrm 172.22.2.222 --winrm-user 'Administrator' --winrm-password 'Nagios2234' --node-name nagiosxi_test_windows_client --winrm-ssl-verify-mode verify_none -V -y --run-list 'recipe[chef-client]'
INFO: Using configuration from /home/ahmed/work/mychefserver/chef-repo/.chef/knife.rb
INFO: HTTP Request Returned 404 Object Not Found: error
INFO: HTTP Request Returned 404 Object Not Found: error
Creating new client for nagiosxi_test_windows_client
Creating new node for nagiosxi_test_windows_client
INFO: HTTP Request Returned 404 Object Not Found: error

Waiting for remote response before bootstrap.172.22.2.222 .
172.22.2.222 Response received.
Remote node responded after 0.01 minutes.
172.22.2.222 AMD64
Bootstrapping Chef on 172.22.2.222
172.22.2.222 Rendering "C:\Users\ADMINI~1\AppData\Local\Temp\bootstrap-18039-1472729874.bat" chunk 1
172.22.2.222 Rendering "C:\Users\ADMINI~1\AppData\Local\Temp\bootstrap-18039-1472729874.bat" chunk 2
172.22.2.222 Rendering "C:\Users\ADMINI~1\AppData\Local\Temp\bootstrap-18039-1472729874.bat" chunk 3
172.22.2.222 Rendering "C:\Users\ADMINI~1\AppData\Local\Temp\bootstrap-18039-1472729874.bat" chunk 4
172.22.2.222 Rendering "C:\Users\ADMINI~1\AppData\Local\Temp\bootstrap-18039-1472729874.bat" chunk 5
172.22.2.222 Rendering "C:\Users\ADMINI~1\AppData\Local\Temp\bootstrap-18039-1472729874.bat" chunk 6
172.22.2.222 Rendering "C:\Users\ADMINI~1\AppData\Local\Temp\bootstrap-18039-1472729874.bat" chunk 7
172.22.2.222 Rendering "C:\Users\ADMINI~1\AppData\Local\Temp\bootstrap-18039-1472729874.bat" chunk 8
172.22.2.222 Checking for existing directory "C:\chef"...
172.22.2.222 Existing directory not found, creating.
172.22.2.222
172.22.2.222 C:\Users\Administrator>(
172.22.2.222 echo.url = WScript.Arguments.Named("url")
172.22.2.222  echo.path = WScript.Arguments.Named("path")
172.22.2.222  echo.proxy = null
172.22.2.222  echo.'* Vaguely attempt to handle file:// scheme urls by url unescaping and switching all
172.22.2.222  echo.'* / into .  Also assume that file:/// is a local absolute path and that file://<foo>
172.22.2.222  echo.'* is possibly a network file path.
172.22.2.222  echo.If InStr(url, "file://") = 1 Then
172.22.2.222  echo.url = Unescape(url)
172.22.2.222  echo.If InStr(url, "file:///") = 1 Then
172.22.2.222  echo.sourcePath = Mid(url, Len("file:///") + 1)
172.22.2.222  echo.Else
172.22.2.222  echo.sourcePath = Mid(url, Len("file:") + 1)
172.22.2.222  echo.End If
172.22.2.222  echo.sourcePath = Replace(sourcePath, "/", "\")
172.22.2.222  echo.
172.22.2.222  echo.Set objFSO = CreateObject("Scripting.FileSystemObject")
172.22.2.222  echo.If objFSO.Fileexists(path) Then objFSO.DeleteFile path
172.22.2.222  echo.objFSO.CopyFile sourcePath, path, true
172.22.2.222  echo.Set objFSO = Nothing
172.22.2.222  echo.
172.22.2.222  echo.Else
172.22.2.222  echo.Set objXMLHTTP = CreateObject("MSXML2.ServerXMLHTTP")
172.22.2.222  echo.Set wshShell = CreateObject( "WScript.Shell" )
172.22.2.222  echo.Set objUserVariables = wshShell.Environment("USER")
172.22.2.222  echo.
172.22.2.222  echo.rem http proxy is optional
172.22.2.222  echo.rem attempt to read from HTTP_PROXY env var first
172.22.2.222  echo.On Error Resume Next
172.22.2.222  echo.
172.22.2.222  echo.If NOT (objUserVariables("HTTP_PROXY") = "") Then
172.22.2.222  echo.proxy = objUserVariables("HTTP_PROXY")
172.22.2.222  echo.
172.22.2.222  echo.rem fall back to named arg
172.22.2.222  echo.ElseIf NOT (WScript.Arguments.Named("proxy") = "") Then
172.22.2.222  echo.proxy = WScript.Arguments.Named("proxy")
172.22.2.222  echo.End If
172.22.2.222  echo.
172.22.2.222  echo.If NOT isNull(proxy) Then
172.22.2.222  echo.rem setProxy method is only available on ServerXMLHTTP 6.0+
172.22.2.222  echo.Set objXMLHTTP = CreateObject("MSXML2.ServerXMLHTTP.6.0")
172.22.2.222  echo.objXMLHTTP.setProxy 2, proxy
172.22.2.222  echo.End If
172.22.2.222  echo.
172.22.2.222  echo.On Error Goto 0
172.22.2.222  echo.
172.22.2.222  echo.objXMLHTTP.open "GET", url, false
172.22.2.222  echo.objXMLHTTP.send()
172.22.2.222  echo.If objXMLHTTP.Status = 200 Then
172.22.2.222  echo.Set objADOStream = CreateObject("ADODB.Stream")
172.22.2.222  echo.objADOStream.Open
172.22.2.222  echo.objADOStream.Type = 1
172.22.2.222  echo.objADOStream.Write objXMLHTTP.ResponseBody
172.22.2.222  echo.objADOStream.Position = 0
172.22.2.222  echo.Set objFSO = Createobject("Scripting.FileSystemObject")
172.22.2.222  echo.If objFSO.Fileexists(path) Then objFSO.DeleteFile path
172.22.2.222  echo.Set objFSO = Nothing
172.22.2.222  echo.objADOStream.SaveToFile path
172.22.2.222  echo.objADOStream.Close
172.22.2.222  echo.Set objADOStream = Nothing
172.22.2.222  echo.End If
172.22.2.222  echo.Set objXMLHTTP = Nothing
172.22.2.222  echo.End If
172.22.2.222 ) 1>C:\chef\wget.vbs
172.22.2.222
172.22.2.222 C:\Users\Administrator>(
172.22.2.222 echo.param(
172.22.2.222  echo.   [String] $remoteUrl,
172.22.2.222  echo.   [String] $localPath
172.22.2.222  echo.)
172.22.2.222  echo.
172.22.2.222  echo.$ProxyUrl = $env:http_proxy;
172.22.2.222  echo.$webClient = new-object System.Net.WebClient;
172.22.2.222  echo.
172.22.2.222  echo.if ($ProxyUrl -ne '') {
172.22.2.222  echo.  $WebProxy = New-Object System.Net.WebProxy($ProxyUrl,$true)
172.22.2.222  echo.  $WebClient.Proxy = $WebProxy
172.22.2.222  echo.}
172.22.2.222  echo.
172.22.2.222  echo.$webClient.DownloadFile($remoteUrl, $localPath);
172.22.2.222 ) 1>C:\chef\wget.ps1
172.22.2.222
172.22.2.222 C:\Users\Administrator>(
172.22.2.222
172.22.2.222
172.22.2.222
172.22.2.222 )
172.22.2.222 Detected Windows Version 6.3 Build 9600
172.22.2.222
172.22.2.222 C:\Users\Administrator>goto Version6.3
172.22.2.222
172.22.2.222 C:\Users\Administrator>goto Version6.2
172.22.2.222
172.22.2.222 C:\Users\Administrator>goto architecture_select
172.22.2.222
172.22.2.222 C:\Users\Administrator>goto install
172.22.2.222 Checking for existing downloaded package at "C:\Users\ADMINI~1\AppData\Local\Temp\chef-client-latest.msi"
172.22.2.222 No existing downloaded packages to delete.
172.22.2.222 Attempting to download client package using PowerShell if available...
172.22.2.222 powershell.exe -ExecutionPolicy Unrestricted -NoProfile -NonInteractive -File  C:\chef\wget.ps1 "https://www.chef.io/chef/download?p=windows&pv=2012&m=x86_64&DownloadContext=PowerShell&v=12" "C:\Users\ADMINI~1\AppData\Local\Temp\chef-client-latest.msi"
172.22.2.222 Exception calling "DownloadFile" with "2" argument(s): "The underlying
172.22.2.222 connection was closed: An unexpected error occurred on a send."
172.22.2.222 At C:\chef\wget.ps1:14 char:1
172.22.2.222 + $webClient.DownloadFile($remoteUrl, $localPath);
172.22.2.222 + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
172.22.2.222     + CategoryInfo          : NotSpecified: (:) [], MethodInvocationException
172.22.2.222     + FullyQualifiedErrorId : WebException
172.22.2.222
172.22.2.222 Failed download: download completed, but downloaded file not found
172.22.2.222 Warning: Failed to download "https://www.chef.io/chef/download?p=windows&pv=2012&m=x86_64&DownloadContext=PowerShell&v=12" to "C:\Users\ADMINI~1\AppData\Local\Temp\chef-client-latest.msi"
172.22.2.222 Warning: Retrying download with cscript ...
172.22.2.222 Download via cscript succeeded.
172.22.2.222 Installing downloaded client package...
172.22.2.222
172.22.2.222 C:\Users\Administrator>msiexec /qn /log "C:\Users\ADMINI~1\AppData\Local\Temp\chef-client-msi22296.log" /i "C:\Users\ADMINI~1\AppData\Local\Temp\chef-client-latest.msi"
172.22.2.222 Successfully installed Chef Client package.
172.22.2.222 Installation completed successfully
172.22.2.222 Writing validation key...
172.22.2.222 Validation key written.
172.22.2.222
172.22.2.222 C:\Users\Administrator>mkdir C:\chef\trusted_certs
172.22.2.222
172.22.2.222 C:\Users\Administrator>(
172.22.2.222 echo.-----BEGIN CERTIFICATE-----
172.22.2.222  echo.MIID7jCCAtagAwIBAgIBADANBgkqhkiG9w0BAQsFADBcMQswCQYDVQQGEwJVUzEQ
172.22.2.222  echo.MA4GA1UECgwHWW91Q29ycDETMBEGA1UECwwKT3BlcmF0aW9uczEmMCQGA1UEAwwd
172.22.2.222  echo.Y2hlZm1ncnNlcnZlci5oZXBzaWJ1cmFkYS5jb20wHhcNMTYwODE5MDc0NjQyWhcN
172.22.2.222  echo.MjYwODE3MDc0NjQyWjBcMQswCQYDVQQGEwJVUzEQMA4GA1UECgwHWW91Q29ycDET
172.22.2.222  echo.MBEGA1UECwwKT3BlcmF0aW9uczEmMCQGA1UEAwwdY2hlZm1ncnNlcnZlci5oZXBz
172.22.2.222  echo.aWJ1cmFkYS5jb20wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDnyCBM
172.22.2.222  echo.nJ5xigDpZGLcOERJ2h3W9DVd4vW1c/xnlWKwe1RuIJxjgN4Wd+uUDrfotarPLOFw
172.22.2.222  echo.I9lAQRlBmNCILLxeAZfUUU8JFB2iiLeKky521qi1eIKLUAefhZMNt5OjjgdWegOP
172.22.2.222  echo.lJ0l+ugb14eXXvIhaeA4wcOF4FjWwwCqY9/wzifBSTVEVTHirAxmIyT4OaBXwpZD
172.22.2.222  echo.r35YuQoOvI+0NsDLKf3i/OKn7IzgC+bOfbN+tb6ZcLLz59tE3/tys/EFOcWaGlsq
172.22.2.222  echo.DSoPr4R7VZicllbOUawx3V2cn3Utbji6583h0IuDQRaymtw4DdpscwCwPpBCLEpr
172.22.2.222  echo.1PRChMFTKCE52TJvAgMBAAGjgbowgbcwDwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4E
172.22.2.222  echo.FgQUokp/XZ9IsbTX0dKgdYyF5sT08aQwgYQGA1UdIwR9MHuAFKJKf12fSLG019HS
172.22.2.222  echo.oHWMhebE9PGkoWCkXjBcMQswCQYDVQQGEwJVUzEQMA4GA1UECgwHWW91Q29ycDET
172.22.2.222  echo.MBEGA1UECwwKT3BlcmF0aW9uczEmMCQGA1UEAwwdY2hlZm1ncnNlcnZlci5oZXBz
172.22.2.222  echo.aWJ1cmFkYS5jb22CAQAwDQYJKoZIhvcNAQELBQADggEBAFN5NAYwHKaxPpFprrfe
172.22.2.222  echo.yGYgZCZY+Pq6hl+Qi/JhRZuiNwXNZ+vB1MJFQOaJnA3XFBDjrrlEEGEePMBC+Oup
172.22.2.222  echo.qhrxp3hRC+NbxsiouZTqnX5Sew0ZOmTSh9AD02iBPC61r0Sbkm1RTtpyIh48KyA+
172.22.2.222  echo.7ZzxvwKsGoZ2aOcJBsaWgdDC4dpxcg8pL7Z4M0bz5vk8unlosLSnoG0EEkv6yr5m
172.22.2.222  echo.r2s7hJPB+D3TqGzXiNhpW27L1eplv9in5P1ezDtjddYWYXUX7KGCuefMoWt/4LV3
172.22.2.222  echo.PFDSr3dBVhHBrQCBTaFA4catTtKd26M7jiU5QkuAg1XAYdYqBfvwEsdHCZabAYHe
172.22.2.222  echo.Xr8=
172.22.2.222  echo.-----END CERTIFICATE-----
172.22.2.222 ) 1>C:\chef/trusted_certs/chefmgrserver_mychefserver_com.crt
172.22.2.222
172.22.2.222 C:\Users\Administrator>(
172.22.2.222 echo.log_level        :info
172.22.2.222  echo.log_location     STDOUT
172.22.2.222  echo.
172.22.2.222  echo.chef_server_url  "https://chefmgrserver.mychefserver.com/organizations/mychefserver"
172.22.2.222  echo.validation_client_name "chef-validator"
172.22.2.222  echo.
172.22.2.222  echo.file_cache_path   "c:/chef/cache"
172.22.2.222  echo.file_backup_path  "c:/chef/backup"
172.22.2.222  echo.cache_options     ({:path => "c:/chef/cache/checksums", :skip_expires => true})
172.22.2.222  echo.
172.22.2.222  echo.node_name "nagiosxi_test_windows_client"
172.22.2.222  echo.trusted_certs_dir "c:/chef/trusted_certs"
172.22.2.222 ) 1>C:\chef\client.rb
172.22.2.222
172.22.2.222 C:\Users\Administrator>(echo.{"run_list":["recipe[chef-client]"]}) 1>C:\chef\first-boot.json
172.22.2.222 Starting chef to bootstrap the node...
172.22.2.222
172.22.2.222 C:\Users\Administrator>SET "PATH=C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\ruby\bin;C:\opscode\chef\bin;C:\opscode\chef\embedded\bin"
172.22.2.222
172.22.2.222 C:\Users\Administrator>chef-client -c c:/chef/client.rb -j c:/chef/first-boot.json
172.22.2.222 [2016-09-01T14:40:22+03:00] INFO: *** Chef 12.13.37 ***
172.22.2.222 [2016-09-01T14:40:22+03:00] INFO: Platform: x64-mingw32
172.22.2.222 [2016-09-01T14:40:22+03:00] INFO: Chef-client pid: 1100
172.22.2.222 [2016-09-01T14:40:54+03:00] INFO: Setting the run_list to ["recipe[chef-client]"] from CLI options
172.22.2.222 [2016-09-01T14:40:54+03:00] INFO: Run List is [recipe[chef-client]]
172.22.2.222 [2016-09-01T14:40:54+03:00] INFO: Run List expands to [chef-client]
172.22.2.222 [2016-09-01T14:40:54+03:00] INFO: Starting Chef Run for nagiosxi_test_windows_client
172.22.2.222 [2016-09-01T14:40:54+03:00] INFO: Running start handlers
172.22.2.222 [2016-09-01T14:40:54+03:00] INFO: Start handlers complete.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: HTTP Request Returned 404 Not Found:
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Loading cookbooks [chef-client@5.0.0, cron@1.7.6, logrotate@2.1.0, compat_resource@12.13.37, windows@1.44.3, chef_handler@1.4.0]
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/recipes/default.rb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/recipes/windows_service.rb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/recipes/runit_service.rb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/recipes/service.rb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/recipes/_unit_test_cloning_resource.rb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/recipes/task.rb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/recipes/smf_service.rb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/recipes/init_service.rb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/recipes/systemd_service.rb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/recipes/launchd_service.rb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/recipes/config.rb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/recipes/src_service.rb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/attributes/default.rb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/recipes/upstart_service.rb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/recipes/cron.rb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/templates/windows/client.service.rb.erb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/recipes/delete_validation.rb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/libraries/helpers.rb in the cache.
172.22.2.222 [2016-09-01T14:40:55+03:00] INFO: Storing updated cookbooks/chef-client/recipes/bsd_service.rb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/arch/chef/chef-client.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/freebsd/chef-client.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/freebsd/chef.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/default/solaris/chef-client.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/default/solaris/manifest.xml.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/default/systemd/chef-client.service.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/default/sv-chef-client-log-run.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/default/client.rb.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/default/debian/default/chef-client.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/default/solaris/manifest-5.11.xml.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/default/debian/init/chef-client.conf.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/default/fedora/sysconfig/chef-client.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/default/debian/init.d/chef-client.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/default/redhat/init.d/chef-client.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/default/redhat/sysconfig/chef-client.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/default/suse/init.d/chef-client.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/default/com.chef.chef-client.plist.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/default/suse/sysconfig/chef-client.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/MAINTAINERS.md in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/templates/default/sv-chef-client-run.erb in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/metadata.json in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/CHANGELOG.md in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/.foodcritic in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/chef-client/README.md in the cache.
172.22.2.222 [2016-09-01T14:40:56+03:00] INFO: Storing updated cookbooks/cron/resources/d.rb in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] ERROR: SSL Validation failure connecting to host: chefmgrserver.mychefserver.com - SSL_read: cert already in hash table
172.22.2.222 [2016-09-01T14:40:57+03:00] ERROR: SSL Error connecting to https://chefmgrserver.mychefserver.com/bookshelf/organization-7e234ea0a3f3e40142d5952ddd238639/checksum-851a37baf4e39196fc664e393cdf2e11?AWSAccessKeyId=8567b11695dde087ee813051a045465e5d2fe704&Expires=1472758855&Signature=FXcKjZ8DyuKcjWpycKCgjaghuU0%3D, retry 1/5
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/cron/providers/d.rb in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/cron/libraries/matchers.rb in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] ERROR: SSL Validation failure connecting to host: chefmgrserver.mychefserver.com - SSL_read: cert already in hash table
172.22.2.222 [2016-09-01T14:40:57+03:00] ERROR: SSL Error connecting to https://chefmgrserver.mychefserver.com/bookshelf/organization-7e234ea0a3f3e40142d5952ddd238639/checksum-f3aef0297c7465e0aeb03116dd16396f?AWSAccessKeyId=8567b11695dde087ee813051a045465e5d2fe704&Expires=1472758855&Signature=trf5jKE%2BipcoJn65yz2jL7Ov3wY%3D, retry 1/5
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/cron/recipes/default.rb in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/cron/attributes/default.rb in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/cron/templates/default/cron_manage.erb in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/cron/templates/default/cron.d.erb in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] ERROR: SSL Validation failure connecting to host: chefmgrserver.mychefserver.com - SSL_read: cert already in hash table
172.22.2.222 [2016-09-01T14:40:57+03:00] ERROR: SSL Error connecting to https://chefmgrserver.mychefserver.com/bookshelf/organization-7e234ea0a3f3e40142d5952ddd238639/checksum-b5b11d46ad336ab4acbe2e488ed8152c?AWSAccessKeyId=8567b11695dde087ee813051a045465e5d2fe704&Expires=1472758855&Signature=fWmrFoBGBmr1/C1JWsvstw4VyfQ%3D, retry 1/5
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/cron/MAINTAINERS.md in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/cron/CHANGELOG.md in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/cron/.foodcritic in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/cron/metadata.json in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/cron/CONTRIBUTING.md in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/logrotate/recipes/default.rb in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/cron/README.md in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/logrotate/resources/app.rb in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/logrotate/recipes/global.rb in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/logrotate/attributes/default.rb in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/logrotate/libraries/matchers.rb in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/logrotate/libraries/logrotate_config.rb in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/logrotate/templates/default/logrotate.erb in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/logrotate/templates/default/logrotate-global.erb in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/logrotate/LICENSE in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/logrotate/Berksfile.lock in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/logrotate/metadata.rb in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/logrotate/.travis.yml in the cache.
172.22.2.222 [2016-09-01T14:40:57+03:00] INFO: Storing updated cookbooks/logrotate/CHANGELOG.md in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/logrotate/.kitchen.yml in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/logrotate/metadata.json in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/logrotate/Makefile in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/logrotate/README.md in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/logrotate/Berksfile in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/logrotate/Gemfile in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/logrotate/CONTRIBUTING.md in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/logrotate/.rubocop.yml in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/logrotate/Gemfile.lock in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/compat_resource/libraries/autoload.rb in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/before/metadata.rb in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/cloning/providers/resource.rb in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/before/recipes/default.rb in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/cloning/metadata.rb in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/hybrid/libraries/normal_hwrp.rb in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/cloning/recipes/default.rb in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/cloning/resources/resource.rb in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/hybrid/providers/resource.rb in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/hybrid/resources/resource.rb in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/hybrid/metadata.rb in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/normal/providers/resource.rb in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/normal/libraries/normal_hwrp.rb in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/normal/resources/resource.rb in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/normal/metadata.rb in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/normal/recipes/declare_resource.rb in the cache.
172.22.2.222 [2016-09-01T14:40:58+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/notifications/metadata.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/notifications/recipes/default.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/test/metadata.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/notifications/resources/resource.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/test/recipes/default.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/test/recipes/test.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/future/libraries/future_custom_resource.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/future/libraries/super_properties.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/config.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/future/metadata.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/future/resources/resource.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/future/resources/super_resource.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/cookbooks/future/recipes/declare_resource.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/Gemfile in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/nodes/ettores-mbp.lan.json in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/data/Gemfile.lock in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/cookbook_spec.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/spec/spec_helper.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/mixin/properties.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/compat_resource/version.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/compat_resource/gemspec.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/provider/noop.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/compat_resource.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/provider/apt_repository.rb in the cache.
172.22.2.222 [2016-09-01T14:40:59+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/mixin/notifying_block.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/provider/apt_update.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/constants.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/mixin/params_validate.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/mixin/powershell_out.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/mixin/lazy_module_include.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/resource.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/mixin/properties.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/run_context.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/provider.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/delayed_evaluator.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/property.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/resource_builder.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/runner.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/recipe.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/resource/apt_repository.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/dsl/declare_resource.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/resource/apt_update.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/dsl/core.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/resource/action_class.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/dsl/platform_introspection.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/dsl/universal.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/monkeypatches/chef/mixin/params_validate.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/monkeypatches/chef/resource.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef/chef/dsl/recipe.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/monkeypatches/chef/run_context.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/monkeypatches/chef/property.rb in the cache.
172.22.2.222 [2016-09-01T14:41:00+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/monkeypatches/chef/log.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/monkeypatches/chef/provider.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/monkeypatches/chef/resource_builder.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/monkeypatches/chef/recipe.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/monkeypatches/chef/resource_collection.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/monkeypatches/chef/runner.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/monkeypatches/chef/resource_collection/resource_set.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] ERROR: SSL Validation failure connecting to host: chefmgrserver.mychefserver.com - SSL_read: cert already in hash table
172.22.2.222 [2016-09-01T14:41:01+03:00] ERROR: SSL Error connecting to https://chefmgrserver.mychefserver.com/bookshelf/organization-7e234ea0a3f3e40142d5952ddd238639/checksum-96227071b1f592e172998e1af8a48058?AWSAccessKeyId=8567b11695dde087ee813051a045465e5d2fe704&Expires=1472758855&Signature=ifqeEv15NFsU7MhM6DVYTkwXStQ%3D, retry 1/5
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/monkeypatches/chef/recipe_hook.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/monkeypatches/chef/resource_collection/resource_list.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] ERROR: SSL Validation failure connecting to host: chefmgrserver.mychefserver.com - SSL_read: cert already in hash table
172.22.2.222 [2016-09-01T14:41:01+03:00] ERROR: SSL Error connecting to https://chefmgrserver.mychefserver.com/bookshelf/organization-7e234ea0a3f3e40142d5952ddd238639/checksum-3b200c3cbc9200da7dd99a93c03bb68c?AWSAccessKeyId=8567b11695dde087ee813051a045465e5d2fe704&Expires=1472758855&Signature=OQCSQdYlLyh2I3In5U4efRjWHpE%3D, retry 1/5
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/monkeypatches/chef.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/resource.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/monkeypatches.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/property.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/copied_from_chef.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/resource/lwrp_base.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/recipe.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/CHANGELOG.md in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/metadata.json in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/windows/resources/auto_run.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/README.md in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/compat_resource/CONTRIBUTING.md in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/windows/resources/registry.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/windows/resources/shortcut.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/windows/resources/feature.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/windows/resources/printer.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/windows/resources/certificate_binding.rb in the cache.
172.22.2.222 [2016-09-01T14:41:01+03:00] INFO: Storing updated cookbooks/windows/resources/task.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/resources/path.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/resources/certificate.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/resources/font.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/chef-client/CONTRIBUTING.md in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/resources/pagefile.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/resources/zipfile.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/resources/http_acl.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/cron/definitions/manage.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/resources/printer_port.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/resources/reboot.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/resources/batch.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/providers/auto_run.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/cron/templates/default/crontab.erb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/providers/certificate_binding.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/providers/printer.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/providers/shortcut.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/providers/registry.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/providers/task.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/providers/path.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/providers/certificate.rb in the cache.
172.22.2.222 [2016-09-01T14:41:02+03:00] INFO: Storing updated cookbooks/windows/providers/font.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/providers/pagefile.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/providers/http_acl.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/providers/zipfile.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/providers/printer_port.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/providers/feature_powershell.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/providers/feature_servermanagercmd.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/providers/feature_dism.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/providers/reboot.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/providers/batch.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/recipes/default.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/recipes/reboot_handler.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/libraries/windows_architecture_helper.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/libraries/matchers.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/libraries/version.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/libraries/powershell_helper.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/libraries/windows_helper.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/libraries/windows_package.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/libraries/powershell_out.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/libraries/wmi_helper.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/libraries/feature_base.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/libraries/registry_helper.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/libraries/windows_privileged.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/attributes/default.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/files/default/handlers/windows_reboot_handler.rb in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/MAINTAINERS.md in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/CHANGELOG.md in the cache.
172.22.2.222 [2016-09-01T14:41:03+03:00] INFO: Storing updated cookbooks/windows/metadata.json in the cache.
172.22.2.222 [2016-09-01T14:41:04+03:00] INFO: Storing updated cookbooks/windows/.foodcritic in the cache.
172.22.2.222 [2016-09-01T14:41:04+03:00] ERROR: SSL Validation failure connecting to host: chefmgrserver.mychefserver.com - SSL_read: cert already in hash table
172.22.2.222 [2016-09-01T14:41:04+03:00] INFO: Storing updated cookbooks/windows/CONTRIBUTING.md in the cache.
172.22.2.222 [2016-09-01T14:41:04+03:00] ERROR: SSL Error connecting to https://chefmgrserver.mychefserver.com/bookshelf/organization-7e234ea0a3f3e40142d5952ddd238639/checksum-444263d2e0ccdad9ea083e944bd7064b?AWSAccessKeyId=8567b11695dde087ee813051a045465e5d2fe704&Expires=1472758855&Signature=4fOcxj%2BRzJf55%2BoJuYt5dmaDHbk%3D, retry 1/5
172.22.2.222 [2016-09-01T14:41:04+03:00] INFO: Storing updated cookbooks/chef_handler/resources/default.rb in the cache.
172.22.2.222 [2016-09-01T14:41:04+03:00] INFO: Storing updated cookbooks/chef_handler/providers/default.rb in the cache.
172.22.2.222 [2016-09-01T14:41:04+03:00] INFO: Storing updated cookbooks/chef_handler/recipes/default.rb in the cache.
172.22.2.222 [2016-09-01T14:41:04+03:00] INFO: Storing updated cookbooks/chef_handler/recipes/json_file.rb in the cache.
172.22.2.222 [2016-09-01T14:41:04+03:00] INFO: Storing updated cookbooks/chef_handler/libraries/helpers.rb in the cache.
172.22.2.222 [2016-09-01T14:41:04+03:00] INFO: Storing updated cookbooks/chef_handler/attributes/default.rb in the cache.
172.22.2.222 [2016-09-01T14:41:04+03:00] INFO: Storing updated cookbooks/chef_handler/files/default/handlers/README in the cache.
172.22.2.222 [2016-09-01T14:41:04+03:00] INFO: Storing updated cookbooks/chef_handler/MAINTAINERS.md in the cache.
172.22.2.222 [2016-09-01T14:41:04+03:00] INFO: Storing updated cookbooks/chef_handler/libraries/matchers.rb in the cache.
172.22.2.222 [2016-09-01T14:41:04+03:00] INFO: Storing updated cookbooks/chef_handler/CHANGELOG.md in the cache.
172.22.2.222 [2016-09-01T14:41:04+03:00] INFO: Storing updated cookbooks/chef_handler/metadata.json in the cache.
172.22.2.222 [2016-09-01T14:41:04+03:00] INFO: Storing updated cookbooks/chef_handler/README.md in the cache.
172.22.2.222 [2016-09-01T14:41:04+03:00] INFO: Storing updated cookbooks/chef_handler/CONTRIBUTING.md in the cache.
172.22.2.222 [2016-09-01T14:41:06+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/monkeypatches/chef/resource/lwrp_base.rb in the cache.
172.22.2.222 [2016-09-01T14:41:06+03:00] INFO: Storing updated cookbooks/compat_resource/files/lib/chef_compat/monkeypatches/chef/exceptions.rb in the cache.
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: Storing updated cookbooks/windows/README.md in the cache.
172.22.2.222 C:/opscode/chef/embedded/lib/ruby/2.1.0/x64-mingw32/dl.so: warning: already initialized constant DL::RUBY_FREE
172.22.2.222
172.22.2.222 C:/opscode/chef/embedded/lib/ruby/gems/2.1.0/gems/net-ssh-3.2.0/lib/net/ssh/authentication/pageant.rb:16: warning: previous definition of RUBY_FREE was here
172.22.2.222 DL is deprecated, please use Fiddle
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: Processing directory[C:/chef/run] action create (chef-client::windows_service line 52)
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: directory[C:/chef/run] created directory C:/chef/run
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: directory[C:/chef/run] owner changed to S-1-5-21-3038300891-2412044433-3823315598-500
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: directory[C:/chef/run] group changed to S-1-5-32-544
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: Processing directory[C:/chef/cache] action create (chef-client::windows_service line 52)
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: directory[C:/chef/cache] owner changed to S-1-5-21-3038300891-2412044433-3823315598-500
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: directory[C:/chef/cache] group changed to S-1-5-32-544
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: Processing directory[C:/chef/backup] action create (chef-client::windows_service line 52)
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: directory[C:/chef/backup] created directory C:/chef/backup
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: directory[C:/chef/backup] owner changed to S-1-5-21-3038300891-2412044433-3823315598-500
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: directory[C:/chef/backup] group changed to S-1-5-32-544
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: Processing directory[C:/chef/log] action create (chef-client::windows_service line 52)
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: directory[C:/chef/log] created directory C:/chef/log
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: directory[C:/chef/log] owner changed to S-1-5-21-3038300891-2412044433-3823315598-500
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: directory[C:/chef/log] group changed to S-1-5-32-544
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: directory[C:/chef/log] permissions changed to [CHEF-WINDOWS-01\Administrator/flags:0/mask:e0010000, BUILTIN\Administrators/flags:0/mask:a0000000, Everyone/flags:0/mask:a0000000]
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: Processing directory[C:/chef] action create (chef-client::windows_service line 52)
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: directory[C:/chef] owner changed to S-1-5-21-3038300891-2412044433-3823315598-500
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: directory[C:/chef] group changed to S-1-5-32-544
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: Processing template[C:/chef/client.service.rb] action create (chef-client::windows_service line 38)
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: template[C:/chef/client.service.rb] created file C:/chef/client.service.rb
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: template[C:/chef/client.service.rb] updated file contents C:/chef/client.service.rb
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: template[C:/chef/client.service.rb] owner changed to S-1-5-21-3038300891-2412044433-3823315598-500
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: template[C:/chef/client.service.rb] group changed to S-1-5-32-544
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: template[C:/chef/client.service.rb] permissions changed to [CHEF-WINDOWS-01\Administrator/flags:0/mask:c0010000, BUILTIN\Administrators/flags:0/mask:80000000, Everyone/flags:0/mask:80000000]
172.22.2.222 [2016-09-01T14:41:09+03:00] INFO: Processing execute[register-chef-service] action run (chef-client::windows_service line 45)
172.22.2.222 [2016-09-01T14:41:14+03:00] INFO: execute[register-chef-service] ran successfully
172.22.2.222 [2016-09-01T14:41:14+03:00] INFO: Processing windows_service[chef-client] action enable (chef-client::windows_service line 50)
172.22.2.222 [2016-09-01T14:41:14+03:00] INFO: Processing windows_service[chef-client] action start (chef-client::windows_service line 50)
172.22.2.222 [2016-09-01T14:41:14+03:00] INFO: windows_service[chef-client] configured with {:service_name=>"chef-client"}
172.22.2.222 [2016-09-01T14:41:22+03:00] INFO: windows_service[chef-client] started
172.22.2.222 [2016-09-01T14:41:22+03:00] INFO: Chef Run complete in 27.546223 seconds
172.22.2.222 [2016-09-01T14:41:22+03:00] INFO: Running report handlers
172.22.2.222 [2016-09-01T14:41:22+03:00] INFO: Report handlers complete
~~~

Now are ready to play with windows cookbooks.
