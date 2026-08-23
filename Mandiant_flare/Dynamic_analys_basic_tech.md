## Dynamic Analysis

```
Extract meaningful runtime characteristics from an unknown binary by allowing it to execute in a
controlled environment


Malware Sandboxes
• Purpose-built appliances for automated malware analysis
o Examples: Joe Sandbox, Cuckoo, VMRay, Hybrid Analysis
• Executes supported file types in an emulated or virtualized environment
• Simulates Internet connectivity and network services
• Captures runtime behavior
• Usually involves injecting analysis code into process memory
o May also intercept and log API calls
• May auto-generate reports with varying degrees of detail

Sysinternals Monitoring Tools
• Process Explorer (procexp.exe)
o Versatile Task Manager replacement with advanced features


• Process Monitor (procmon.exe)
o Monitors file system, registry, process, and some network events in real time
o Set filters to manage output

Process Explorer
• Color coding
o options => configure colors to change or see details
o Can change color duration to improve readability
• Show lower pane
o Handles or DLLs
• Double click to get process details
o Strings on disk image vs. memory

Process Monitor
• Use filters and highlights to capture and emphasize relevant behavior
• Filter by operation
o Process Create
o WriteFile
o RegSetValue
o SetDispositionInformationFile
• Filter or highlight based on process name
• Exclude common processes or operations
• Try different strategies
• Save filters for future use

Network Monitoring Tools
• FakeNet-NG
o Runs inside the analysis VM or in a separate VM
o Simulates common Internet protocols and services (e.g., DNS, HTTP/S, SMTP)
o Automatic protocol and SSL detection
o Process tracking and filtering
o Highly configurable interception engine
o Generates a .pcap traffic capture for each run
• Wireshark
o De facto tool for analyzing .pcap files

```


```
Launching Binaries

Launching Binaries
• EXEs
o Execute from an administrative command prompt
o Look for possible usage information or debug messages printed to the console
• DLLs
o Examine DLL export table and select an export function to execute

o Command line execution format
▪ >rundll32.exe <DLL_name>[, <DLL_export>]
▪ >rundll32.exe <DLL_name>[, #ORDINAL]
o Example:
▪ >rundll32.exe hello.dll, Install

• Service DLLs
o Modify an existing Windows service entry or create a dummy service
▪ SYSTEM\CurrentControlSet\Services\AppMgmt\Parameters\ServiceDLL

▪ >net start AppMgmt
o Malware Analyst’s Cookbook - install_svc.bat and install_svc.py

Dumping Memory

• Dynamic Analysis can also enhance our Static Analysis capabilities
• What obstacles did we encounter during Basic Static Analysis?
o Encoded strings
o Packing
• Difficult to overcome using Static Analysis
• A common technique is to let the malware do the work, then dump the decoded and/or unpacked data to disk.

Process DUmp

Process Dump extracts PE files from a process in memory and dumps them to disk
• Workflow
o Run a packed sample
o Suspend process
o Dump memory
o Analyze unpacked sample

• Usage:
o <pd32.exe | pd64.exe> -pid <pid>
o <pd32.exe | pd64.exe> -p <process name>

Process Dump Advanced Tricks
• Dump any process as it exits
o pd64.exe -closemon
• Dump any unrecognized module
o First generate a whitelist of running modules:
▪ pd64.exe -db -genquick
• Launch the malware
• Dump all modules not matching the generated whitelist:
o pd64.exe -system


```


```
Usual Workflow

 Connect the network adapter in Host-only mode
 Start Process Monitor and set filters accordingly
 Start Process Explorer
 Start FakeNet-NG and test connectivity
 Start any other tools
 Create a VM snapshot
 Launch binary

Mine -> Flare -> Remnux
 Connect the network adapter in AdapterX mode
 Run Inetsim on Remnux & try to ping a site once to test connectivity
 Start Process Monitor and set filters accordingly
 Start Process Explorer
 Start any other tools
 Create a VM snapshot
 Launch binary


```