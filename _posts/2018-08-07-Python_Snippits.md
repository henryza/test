---
layout: post
category: powershell
title: powershell Snips
tagline: by Henry
tags:
  - powershell
  -snips
published: true
---

# Powershell Snippits
Some of my PowerShell snippits that come in useful

## Get Script working directory in (PowerShell)
```PowerShell
#Get Script Execution Directory
[System.IO.Path]::GetDirectoryName($myInvocation.MyCommand.Definition)
#OR
$PSScriptRoot
#OR
$scriptPath = split-path -parent $MyInvocation.MyCommand.Definition
```

## How to tail in PowerShell
``` PowerShell
Get-Content (read-host "Enter Path to logfile") -wait
```

## Get all local users in Admin with PowerShell
``` PowerShell
Get-CimInstance Win32_GroupUser | where {$_.GroupComponent -like "*admini*"} |select -ExpandProperty PartComponent
```

## Example Powershell Regex
``` PowerShell
$a="a,b,c,d,e,f,g"
([regex]::Matches($a, "<=,/" )).count
```

## Powershell version
``` PowerShell
PSVersionTable.PSVersion
```

## How to sign your Powershell scripts
This link gives a breakdown of how to get this done [CodeSigning](https://blogs.technet.microsoft.com/heyscriptingguy/2010/06/17/hey-scripting-guy-how-can-i-sign-windows-powershell-scripts-with-an-enterprise-windows-pki-part-2-of-2/)
``` powershell
#Once you have your cert, use this to sign your script
Set-AuthenticodeSignature SCRIPT @(Get-ChildItem cert:\CurrentUser\My -codesign)[0]
```

### Add Powershell modules from custom path
``` PowerShell
Add-Content -Path $Profile -Value 'Import-Module "C:\Program Files (x86)\SomeProduct\PowerShell\ProductModule.psd1"' -Force
```

## BAse64 in PowerShell
``` PowerShell
$test = -join ((65..122) | Get-Random -Count 10000 | % {[char]$_})
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes("$test"))
```


### Powershell get last shutdown environments
``` PowerShell
$arrShutdown =  Get-EventLog -LogName System -EntryType Information -InstanceId 2147484722 | sort-object $_.Time -descending
    foreach ($item in $arrShutdown)
    {
        $UserName = $item.UserName
        $Message = $item.Message
        $Time = $item.TimeGenerated
        write-host "UserName `t:$username `nTime `t`t:$time `nMessage `t:$Message `n`n"

    }
```

## Last Reboot Time
``` powershell
Get-WmiObject win32_operatingsystem | select @{LABEL="LastBootUpTime";EXPRESSION={$_.ConverttoDateTime($_.lastbootuptime)}}  
```

## How to list all mode paths
``` PowerShell
(Get-Module -ListAvailable *).path
```

## Event Logs (PowerShell)
``` powershell
Get-EventLog -LogName Application -After (Get-Date).AddDays(-2) -EntryType "Error" | export-csv .\ApplicationLog-Errors.csv
Get-EventLog -LogName Application -After (Get-Date).AddDays(-2) -EntryType "Warning" | export-csv .\ApplicationLog-Warnings.csv
Get-EventLog -LogName System -After (Get-Date).AddDays(-2) -EntryType "Error" | export-csv .\SystemLog-Errors.csv
Get-EventLog -LogName System -After (Get-Date).AddDays(-2) -EntryType "Warning" | export-csv .\SystemLog-Warnings.csv
```

## Hashes with PowerShell
``` powershell
Get-FileHash -Algorithm SHA1
Get-FileHash -Algorithm SHA256
Get-FileHash -Algorithm SHA384
Get-FileHash -Algorithm SHA512
Get-FileHash -Algorithm MACTripleDES
Get-FileHash -Algorithm MD5
Get-FileHash -Algorithm RIPEMD160
```

## Secure Strings with PowerShell
``` PowerShell
$SecurePassword = (Get-Credential).password | Convertfrom-SecureString

$SecurePassword |out-file c:\temp\SecurePassword.txt

$SecurePassword=c:\temp\SecurePassword.txt | Convertto-SecureString
$BSTR= [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($SecurePassword)

$Password=[System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)
```

## AES in PowerShell
[AES Encryption](https://gist.github.com/ctigeek/2a56648b923d198a6e60)


## Auto Elevate Powershell Script
``` PowerShell
#Elevate to Admin
if (!([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")) { Start-Process powershell.exe "-NoProfile -ExecutionPolicy Bypass -File `"$PSCommandPath`"" -Verb RunAs; exit }
#Your code here
```

## Powershell get IPV4 address
``` PowerShell
#Get all Valid IPV4
Get-CimInstance Win32_NetworkAdapterConfiguration | where{$_.ipenabled -like "True"}| Select -ExpandProperty IPAddress | where{$_ -like "*.*.*"}
#Select first valid V4
Get-CimInstance Win32_NetworkAdapterConfiguration | where{$_.ipenabled -like "True"}| Select -ExpandProperty IPAddress | where{$_ -like "*.*.*"} | Select -first 1

#Quick method
$localIpAddress=((ipconfig | findstr [0-9].\.)[0]).Split()[-1]

#Longer Method
$ipaddress=([System.Net.DNS]::GetHostAddresses('PasteMachineNameHere')|Where-Object {$_.AddressFamily -eq "InterNetwork"}   |  select-object IPAddressToString)[0].IPAddressToString

#Prone to DNS Failure
Test-Connection -ComputerName localhost -Count 1  | Select IPV4Addres
```

## Epoc Time to Human Readable in PowerShell
``` PowerShell
#Get EPOCH time ( Unix Time)
#The Unix epoch (or Unix time or POSIX time or Unix timestamp) is the number of seconds that have elapsed since January 1, 1970 (midnight UTC/GMT), not counting leap seconds (in ISO 8601: 1970-01-01T00:00:00Z).
[int][double]::Parse((Get-Date (get-date).touniversaltime() -UFormat %s))


#Convert from Epoch to human time
$origin = New-Object -Type DateTime -ArgumentList 1970, 1, 1, 0, 0, 0, 0
$whatIWant = $origin.AddSeconds($unixTime)
#left(10)
$e = $e.Substring(0,10)
```

## Powershell The Date Thing (PowerShell)
``` PowerShell
$currentDate = Get-date
$yesterdaysDate = (Get-Date).AddDays(-1)
$tomorrosDAte = (Get-Date).AddDays(1)
$hours = (Get-Date).addHours(-1)
$minute = (Get-Date).addMinutes(-1)
$minute = (Get-Date).addYears(-1)

#DATES BREAKDWON
$today = get-Date
$today.DayOfWeek
$today.month
$today.day
$today.year

#FORMATTING
get-date -format "dd-MM-yyyy HH:mm:ss"


Get-Date
(Get-Date).AddDays(1)
(Get-Date).AddDays(1)
(Get-Date).AddHours(1)
(Get-Date).AddMilliseconds(1)
(Get-Date).AddMinutes(1)
(Get-Date).AddMonths(1)
(Get-Date).AddSeconds(1)
(Get-Date).AddTicks(1)
(Get-Date).AddYears(1)



(get-date).day
(get-date).dayofweek
(get-date).dayofyear
(get-date).hour
(get-date).millisecond
(get-date).minute
(get-date).month
(get-date).second
(get-date).timeofday
(get-date).year


new-object system.globalization.datetimeformatinfo
get-date -format

```


|Specifier|				Format              						|Sample Output|
|---------|---------------------------------------|-------------|
|d					|ShortDatePattern						           |8/30/2007|
|D					|LongDatePattern							         |Thursday, August 30, 2007|
|f					|Full date and time (long date and short time)			|Thursday, August 30, 20|
|F					|FullDateTimePattern (long date and long time)			|Thursday, August 30, 2007 11:19:59 AM|
|G					  |General (short date and long time)		|8/30/2007 11:20:24 AM|
|m, M			  |MonthDayPattern							        |August 30|
|g					|General (short date and short time)	|8/30/2007 11:20 AM|
|o					|Round-trip date/time pattern		      |2007-08-30T11:18:49.0312500-07:00	RFC1123Pattern Thu, 30 Aug 2007 11:21:36 GMT|
|s					|SortableDateTimePattern (based on ISO 8601) using local time	|2007-08-30T11:20:36|
|t					|ShortTimePattern						          |11:20 AM|
|T					|LongTimePattern							        |11:20:42 AM|
|u					|UniversalSortableDateTimePattern format for universal time	2007-08-30 11:21:50Z
|U					|Full date and time (long date and long time) using universal 	|Thursday, August 30, 2007 6:21:52 PM|
|y, Y				|YearMonthPattern						          |August, 2007|

|Specifier		|Description|
|-------------|--------------------------------------------------------------------|
|d. %d			|The day of the month. Single-digit days will not have a leading zero. Specify "%d" if the format pattern is not combined with other format patterns.|
|dd			    |The day of the month. Single-digit days will have a leading zero.|
|ddd			  |The abbreviated name of the day of the week.|
|dddd			  |The full name of the day of the week, as defined in DayNames.|
|h, %h			|The hour in a 12-hour clock. Single-digit hours will not have a leading zero. Specify "%h" if the format pattern is not combined with other format patterns.|
|hh			    |The hour in a 12-hour clock. Single-digit hours will have a leading zero.|
|H, %H			|The hour in a 24-hour clock. Single-digit hours will not have a leading zero. Specify "%H" if the format pattern is not combined with other format patterns.|
|HH			    |The hour in a 24-hour clock. Single-digit hours will have a leading zero.|
|m, %m			|The minute. Single-digit minutes will not have a leading zero. Specify "%m" if the format pattern is not combined with other format patterns.|
|mm			    |The minute. Single-digit minutes will have a leading zero.|
|M, %M			|The numeric month. Single-digit months will not have a leading zero. Specify "%M" if the format pattern is not combined with other format patterns.|
|MM			    |The numeric month. Single-digit months will have a leading zero.|
|MMM			  |The abbreviated name of the month, as defined in AbbreviatedMonthNames.|
|MMMM			  |The full name of the month, as defined in MonthNames.|
|s, %s			|The second. Single-digit seconds will not have a leading zero. Specify "%s" if the format pattern is not combined with other format patterns.|
|ss			    |The second. Single-digit seconds will have a leading zero.|
|t, %t			|The first character in the AM/PM designator defined in AMDesignator or PMDesignator, if any. Specify "%t" if the format pattern is not combined with other format patterns.|
|tt			    |The AM/PM designator defined in AMDesignator or PMDesignator, if any.||
|y, %y			|The year without the century. If the year without the century is less than 10, the year is displayed with no leading zero. Specify "%y" if the format pattern is not combined with other format patterns.|
|yy			    |The year without the century. If the year without the century is less than 10, the year is displayed with a leading zero.|
|yyy			 |The year in three digits. If the year is less than 100, the year is displayed with a leading zero.|
|yyyy		   |The year in four or five digits (depending on the calendar used), including the century. Will pad with leading zeroes to get four digits. Thai Buddhist and Korean calendars both have five digit years; users selecting the "yyyy" pattern will see |

## The String Thing (PowerShell)
```PowerShell
[string]::Compare($a, $b, $True)

$a.StartsWith("Script")
$a.EndsWith("Script")

$a.ToLower()
$a.ToUpper()
$d.ToLower()

$a.trim()
$a.Replace("Scriptign", "Scripting")
$e.Substring(3,3)
$e.ToCharArray()

$Text -split  ','

$Text.IndexOf('y')
$a.Contains("ript")
```


## Template PowerShell Params
``` PowerShell
Param([
	parameter(
		Mandatory=$true,
		Position=0,
		ValueFromPipelineByPropertyName=$true,
		ValueFromPipeline=$true,
		HelpMessage="Enter one or more User names separated by commas.")
	]
	[ValidateNotNullOrEmpty()]
	[alias("UN","User")]
	[String[]]
	$UserName,

	[parameter(
		Mandatory=$true,
		Position=1,
		ValueFromPipelineByPropertyName=$true,
		ValueFromPipeline=$true,
		HelpMessage="Enter one or more Computer names separated by commas.")
	]
	[ValidateNotNullOrEmpty()]
	[alias("CN","Computer")]
	[String[]]
	$ComputerName

	[switch]$DebugValue
)
```



## Powershell Headers / comments note
```PowerShell
<#
.Synopsis
	Checks services and identifies all services that need to be running.
	Attempts to restart if they are not running. Reports on success and failures
.DESCRIPTION
   This script runs through all services and fixes services that should be running.
   Du to space limitations it has currently only got faulty services enabled for logging.
   Upon trying to restart the service 3 times and failing it will report the service down.
   Down will report NOK, up OK and remediated Fixed.
   Due to the number fo services Faulty services by default are enabled in one file.
   Running services that are non faulty are in another report but disabled
   This report has a cleanup task that runs every 30 days
.Parameter
.Inputs
	-Computername
.Outputs
	This file will generate an output file of ".\$ComputerName-ServicesFAULTY-$Date.txt"
	and  ".\$ComputerName-ServicesOK-$Date.txt"
.Example
    Diskspace -computername
.Notes
	This report will error out if unable to connect to the server with a try/catch/finally statement
	Created by 	: Henry Stock
	Version 	: V1.0.0.0a
	Dated		: March 2016
	Authorised	: Henry Stock
	OS			: Windows
	Clients		: All
	PS Version	: All
.Link
#>
```


## Debug Options
``` powershell
$VerbosePreference =

<#
Continue 			- Show the message on the console
SilentlyContinue 	- do not show the message. [this is the Default action]
Stop 				- show the message and then halt
Inquire 			- prompt the user
<#

<#
Write-Debug 		- Write a debug message to the host display
Write-Error 		- Write an object to the error pipeline.
Write-Host 			- Display objects through the host user interface
Write-Output 		- Write an object to the pipeline
Write-Progress 		- Display a progress bar
Write-Warning 		- Write a warning message
#>
````


## Active directory password last reset 60 days and others
``` PowerShell
Import-Module *Active*

#Windows 7, Enabled, Last 35Days
get-adcomputer -filter * -properties * |  select DNSHostName,OperatingSystem,Enabled,@{N='pwdLastSet'; E={[DateTime]::FromFileTime($_.pwdLastSet)}}|  where{$_.pwdLastSet -ge (get-date).adddays(-60) -and $_.OperatingSystem -like "*7*" -and $_.enabled -like "*True*"} |  export-csv ADComputers.csv


#Servers, Enabled, Last 35Days
get-adcomputer -filter * -properties * | select DNSHostName,OperatingSystem,Enabled,@{N='pwdLastSet'; E={[DateTime]::FromFileTime($_.pwdLastSet)}}| where{$_.pwdLastSet -ge (get-date).adddays(-60) -and $_.OperatingSystem -like "*Server*" -and $_.enabled -like "*True*"} |  export-csv ADServers.csv

Get-ADuser -Filter {UserPrinciplieName -like "**"}  -prop * | Select isDeleted,enabled,CN,displayName,description, @{n="lastLogonDate";e={[datetime]::FromFileTime($_.l)}},{n="lastLogonDate";e={[datetime]::FromFileTime
```


## Powershell get disk Diskspace
``` powershell
$arrDisk = Get-WmiObject -class Win32_LogicalDisk
    foreach($Item in $arrDisk)
    {
        $Name = $Item.DeviceID
        $Caption = $item.VolumeName
        $PercentFree = [math]::round( ( ($item.FreeSpace /$item.Size)*100),2)
        $PercentUsed =  [math]::round( ((  $item.Size - $item.FreeSpace )/$item.Size)*100,2)
        $FreeSpace = [math]::round($item.FreeSpace/1gb,2)
        $UsedSpace =  [math]::round(($item.Size/1gb - $item.FreeSpace/1gb ),2)     
        $TotalSize =  [math]::round($item.Size/1gb,2)

        write-host "Drive `t:$Name `nLabel `t:$Caption `n%Free `t:$PercentFree% `n%Used `t:$PercentUsed% `nFree `t:$FreeSpace GB `nUsed `t:$UsedSpace GB `nTotal `t:$TotalSize GB `n"

    }

```
