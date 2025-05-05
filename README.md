## 1. Salvestussüsteemi seisundi analüüs ja aruandlus

Loo funktsioon `Get-DiskHealthReport`, mis koostab aruande kõigi füüsiliste ketaste kohta. Funktsioon võtab argumendina ühe parameetri – output faili asukoha. Kui parameetrit ei ole antud, kasutatakse vaikimisi asukohta "$env:SystemRoot**\temp\healthReport.json**".

1. Aruanne peaks sisaldama:
    - Ketta number, friendly name ja maht
    - Health seisund ja tööseisund
    - Püsivara versioon
    - S.M.A.R.T andmed (kui saadaval)
    - Andmete põhjal arvutatakse "Riskitase"
        - HealthStatus -eq “Healthy” ja OperationalStatus -eq “OK” → “Low”
        - HealthStatus -eq "Warning" → “Medium”
        - muu → “High”
2. Vorminda väljund konsoolisõbralikus tabelis ning salvesta JSON formaadis faili
3. Lisa kokkuvõtlik statistika (”Low” riskitasemega ketaste koguarv, tähelepanu vajavate ketaste arv)
4. Lisaülesanne loo funktsioon `Get-DiskHealthReportHTML` mis paneb kokku aruanne HTML formaadis vastavalt mallile.

Näide: [DiskHealthReport.html](DiskHealthReport.html)
    
    - **Vihje**
        
        seisundi saamiseks kasutage `.HealthStatus` ,  `OperationalStatus` ja SMART `Get-StorageReliabilityCounter`.
        
        Vajalikud parameetrid: DeviceId, FriendlyName, Size (aga GB-des), HealthStatus, OperationalStatus, FirmwareVersion.
        

## 2. Automatiseeritud ketta puhastamine ja optimeerimine

Loo skript `Start-DriveCleanup`, mis teostab järgmised toimingud:

- Kontrollib ja eemaldab ajutised failid (Windowsi temp-kataloog, kasutajate temp-kataloogid)
- Tühjendab prügikasti
- Analüüsib kõigi volume fragmenteerituse taset
- Optimeerib (defragmenteerib) need, mille fragmenteerituse tase ületab 10%
- Koostab võrdlusaruanne “enne-pärast”, näidates vabastatud kettaruumi ja vähenenud fragmenteeritust

Parameetrite nimed ja vaikimisi väärtused on järgmised:

```powershell
param(
[switch]$TempCleanup = $true,
    [switch]$EmptyRecycleBin = $true,
    [switch]$OptimizeVolumes = $true,
    [int]$FragmentationThreshold = 10,
    [string]$DriveLetters = "",
    [string]$LogPath = "$env:SystemRoot\temp\DiskCleanup_$(Get-Date -Format 'yyyyMMdd_HHmmss').log"
)
```

1. Skript peab logima kõik toimingud ja sisaldama veatuvastust (error handling)
2. Vea tekkimisel skript peab oma töö jätkama (SilentlyContinue)

Aruanne format (`Verbose` paremeeter ei ole kohustuslik):

```powershell
Transcript started, output file is C:\Temp\DiskCleanup_20250424_100114.log
Starting disk cleanup and optimization script at 04/24/2025 10:01:14
Initial free space on C: 12.74 GB
Cleaning up temporary files...
Temporary files cleanup complete. Initial size: 10.84 KB
Emptying Recycle Bin...
Recycle Bin emptied successfully.
Analyzing disk fragmentation...
Analyzing volume C...
VERBOSE: Invoking analysis on (C:)...

VERBOSE: Analysis:  0% complete...
VERBOSE: Analysis:  63% complete...
VERBOSE: Analysis:  69% complete...
VERBOSE: Analysis:  78% complete...
VERBOSE: Analysis:  89% complete...
VERBOSE: Analysis:  100% complete...
VERBOSE: Analysis:  100% complete.  
VERBOSE: 
Post Defragmentation Report:

VERBOSE: 
	Volume Information:
VERBOSE: 		Volume size                 = 29.43 GB
VERBOSE: 		Cluster size                = 4 KB
VERBOSE: 		Used space                  = 16.68 GB
VERBOSE: 		Free space                  = 12.74 GB
VERBOSE: 
	Fragmentation:
VERBOSE: 		Total fragmented space      = 6%
VERBOSE: 		Average fragments per file  = 1.05
VERBOSE: 		Movable files and folders   = 53518
VERBOSE: 		Unmovable files and folders = 6
VERBOSE: 
	Files:
VERBOSE: 		Fragmented files            = 1003
VERBOSE: 		Total file fragments        = 2900
VERBOSE: 
	Folders:
VERBOSE: 		Total folders               = 3922
VERBOSE: 		Fragmented folders          = 365
VERBOSE: 		Total folder fragments      = 1521
VERBOSE: 
	Free space:
VERBOSE: 		Free space count            = 130
VERBOSE: 		Average free space size     = 154.14 MB
VERBOSE: 		Largest free space size     = 18.03 GB
VERBOSE: 
	Master File Table (MFT):
VERBOSE: 		MFT size                    = 85.50 MB
VERBOSE: 		MFT record count            = 87551
VERBOSE: 		MFT usage                   = 100%
VERBOSE: 		Total MFT fragments         = 4
VERBOSE: 	Note: File fragments larger than 64MB are not included in the fragmentation statistics.
VERBOSE: 
	You do not need to defragment this volume.
Volume C: Defragmentation recommended - 

Cleanup and Optimization Report
------------------------------
Drive C:
  Initial free space: 12.74 GB
  Current free space: 12.74 GB
  Space gained: 0 bytes
  Initial fragmentation status: 
  Current fragmentation status: 

Total space gained: 0 bytes
Disk cleanup and optimization completed at 04/24/2025 10:01:24
Transcript stopped, output file is C:\Temp\DiskCleanup_20250424_100114.log
```

- Vihje
    
    Kasutage `Start-Transcript` logi kirjutamiseks and `Optimize-Volume` defragmenteerimiseks.
