# Known Issues and Limitations

Breaking Change: Certificates used to encrypt/decrypt passwords in DSC configurations may not work after installing WMF 5.0 RTM
--------------------------------------------------------------------------------------------------------------------------------

In WMF 4.0 and WMF 5.0 Preview releases, DSC would not allow passwords in the configuration to be of length more than 121 characters. DSC was forcing to use short passwords even if lengthy and strong password was desired. This breaking change allows passwords to be of arbitrary length in the DSC configuration.

Resolution: Re-create the certificate with Data Encipherment or Key Encipherment Key usage, and Document Encryption Enhanced Key usage (1.3.6.1.4.1.311.80.1). Technet article <https://technet.microsoft.com/en-us/library/dn807171.aspx> has more information.


LCM can go into an unstable state while using Get-DscConfiguration in DebugMode
-------------------------------------------------------------------------------

If LCM is in DebugMode, pressing CTRL+C to stop the processing of Get-DscConfiguration can cause LCM to go into an unstable state such that majority of DSC cmdlets won’t work.

Resolution: Don’t press CTRL+C while debugging Get-DscConfiguration cmdlet.


If LCM is in DebugMode, Stop-DscConfiguration may hang while trying to stop an operation started by Get-DscConfiguration
------------------------------------------------------------------------------------------------------------------------

Resolution: Finish the debugging of the operation started by Get-DscConfiguration as outlined in section ‘[DSC RESOURCE SCRIPT DEBUGGING](#dsc-resource-script-debugging)’.


Connecting to an old remote Exchange endpoint causes a crash
------------------------------------------------------------

The old Exchange endpoint redirects to a new endpoint. There is a bug in the redirection logic that results in a crash.

Resolution: Connect directly to the new endpoint.


PowerShell Shortcuts are broken when used for the first time
------------------------------------------------------------

Resolution: Perform one of the following actions:

1.  Right click on the PowerShell shortcut. Select “Windows PowerShell” to launch in a non-elevated mode.
2.  Right click on the PowerShell shortcut. Right click on “Windows PowerShell” and select “Run As Administrator” to launch in an elevated mode.

Once you have performed either of the above actions, the PowerShell shortcuts will work. These actions only need to be performed only once.


Invoke-DscResource operations cannot be retrieved by Get-DscConfigurationStatus cmdlet
--------------------------------------------------------------------------------------

After using Invoke-DscResource cmdlet to direct invoke any resource’s methods, the records of such operation cannot be retrieved through Get-DscConfigurationStatus at a later time.

Resolution: None.


Various Partial Configuration documents for same node cannot have identical resource names
------------------------------------------------------------------------------------------

For several partial configurations that are deployed onto a single node, identical names of resources cause run time error.

Resolution: Use different names for even same resources in different partial configurations.


Start-DscConfiguration –UseExisting does not work with -Credential
------------------------------------------------------------------

When using Start-DscConfiguration with –UseExisting parameter set, the –Credential parameter is ignored. DSC uses default process identity to proceed the operation. This causes error when a different credential is needed to proceed on remote node.

Resolution: Use CIM session for remote DSC operations

PS &gt; $session = New-CimSession -ComputerName $node -Credential $credential
PS &gt; Start-DscConfiguration -UseExisting -CimSession $session


Running Set-DscLocalconfigurationmanager to set Meta Configuration against WMF 4.0 or WMF 5.0 Production Preview builds will not work
---------------------------------------------------------------------------------------------------------------------------------------

There is a backward compatibility issue when running Set-DscLocalConfiguration against previous WMF’s. You will see this error complaining about new added cmdlet parameter –Force not available on the target machine.

PS C:\\&gt; Set-DscLocalConfigurationManager -Path . -Verbose -ComputerName WIN-3B576EM3669
VERBOSE: Performing the operation "Start-DscConfiguration: SendMetaConfigurationApply" on target "MSFT\_DSCLocalConfigurationManager".
VERBOSE: Perform operation 'Invoke CimMethod' with following parameters, ''methodName' = SendMetaConfigurationApply,'className' = MSFT\_DSCLocalConfigurationManager,'namespaceName' = root/Microsoft/Windows/DesiredStateConfiguration'.
The WinRM client cannot process the request. The object contains an unrecognized argument: "Force". Verify that the spelling of the argument name is correct.
+ CategoryInfo : MetadataError: (root/Microsoft/...gurationManager:String) \[\], CimException
+ FullyQualifiedErrorId : HRESULT 0x803381e1
+ PSComputerName : WIN-3B576EM3669
VERBOSE: Operation 'Invoke CimMethod' complete.
VERBOSE: Set-DscLocalConfigurationManager finished in 0.121 seconds.

Resolution: Follow these steps to use Invoke-CimMethod to call underlying CIM method directly to set meta configuration.
PS C:\\&gt; $computerName = "WIN-3B576EM3669"
PS C:\\&gt; $mofPath = "C:\\$computerName.meta.mof"
PS C:\\&gt; $configurationData = \[Byte\[\]\]\[System.IO.File\]::ReadAllBytes($mofPath)
PS C:\\&gt; $totalSize = \[System.BitConverter\]::GetBytes($configurationData.Length + 4 )
PS C:\\&gt; $configurationData = $totalSize + $configurationData
PS C:\\&gt; Invoke-CimMethod -Namespace root/microsoft/windows/desiredstateconfiguration -class MSFT\_DSCLocalConfigurationManager -name SendMetaConfigurationApply -arguments @{ConfigurationData = \[Byte\[\]\]$configurationData} -Verbose -ComputerName $computerName
VERBOSE: Performing the operation "Invoke-CimMethod: SendMetaConfigurationApply" on target "MSFT\_DSCLocalConfigurationManager".
VERBOSE: Perform operation 'Invoke CimMethod' with following parameters, ''methodName' = SendMetaConfigurationApply,'className' = MSFT\_DSCLocalConfigurationManager,'namespaceName' = root/microsoft/windows/desiredstateconfiguration'.
VERBOSE: An LCM method call arrived from computer WIN-3B576EM3669 with user sid S-1-5-21-2127521184-1604012920-1887927527-5557045.
VERBOSE: \[WIN-3B576EM3669\]: LCM: \[ Start Set \]
VERBOSE: \[WIN-3B576EM3669\]: LCM: \[ Start Resource \] \[MSFT\_DSCMetaConfiguration\]
VERBOSE: \[WIN-3B576EM3669\]: LCM: \[ Start Set \] \[MSFT\_DSCMetaConfiguration\]
VERBOSE: \[WIN-3B576EM3669\]: LCM: \[ End Set \] \[MSFT\_DSCMetaConfiguration\] in 0.4060 seconds.
VERBOSE: \[WIN-3B576EM3669\]: LCM: \[ End Resource \] \[MSFT\_DSCMetaConfiguration\]
VERBOSE: \[WIN-3B576EM3669\]: LCM: \[ End Set \] in 0.4807 seconds.
VERBOSE: Operation 'Invoke CimMethod' complete.


Variables & Functions defined in $script scope in DSC Class-Based Resource are not preserved across multiple calls to a DSC Resource 
-------------------------------------------------------------------------------------------------------------------------------------

Multiple consecutive calls to Start-DSCConfiguartion will fail if configuration is using any class based resource which has variables or functions defined in $script scope.

Resolution: Define all variables and function in DSC Resource class itself. No script scope variables/functions.


PsDscRunAsCredential is not supported for DSC Composite Resources
----------------------------------------------------------------

Resolution: Use Credential property if available. Example ServiceSet and WindowsFeatureSet


WindowsOptionalFeature is not available in Windows 7
-------------------------------------------------

The WindowsOptionalFeature DSC resource is not available in Windows 7. This resource requires the DISM module, and DISM cmdlets that are available starting in Windows 8 and newer releases of the Windows operating system.


Software Inventory Logging feature is erroneously stopped after WMF 5 installation on Windows Server 2012 R2
-------------------------------------------------------------------------------------------------------------

When installing WMF 5.0 on a Windows Server 2012 R2 Server that is already running SIL, the Software Inventory Logging feature is erroneously stopped after install.

Resolution: Run the Start-SilLogging cmdlet once after the WMF install, as the installation process will errantly stop the Software Inventory Logging feature.
