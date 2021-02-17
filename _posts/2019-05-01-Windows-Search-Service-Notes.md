The Windows Search Service is not enabled by default on Windows Server 2012r2, 2016, 2019. Reference: [https://support.microsoft.com/en-us/help/3204979/windows-search-is-disabled-by-default-in-windows-server-2016](https://support.microsoft.com/en-us/help/3204979/windows-search-is-disabled-by-default-in-windows-server-2016)

## Impact
As file shares are created in an Azure hosted multi-terabyte scenario, Windows clients must index share content over the WAN to be able to search by filename or inside indexable files. Even before several Gigabytes of content is shared, the indexing process becomes slow, consumes bandwidth and is not shared with other windows clients. This end user experience is most pronounced in these Azure cloud via WAN scenarios. However, given fast server disks and fast gigabit or WAN networking, small workgroups may not notice the absence of the search service.

## Solution
Install Windows Search service and configure indexing so that windows clients can query the server's search index. The result is faster, shared consistent shared content indexing. For example, a windows client performing a search of 100GB of mixed MS Word, PDF and other shared data files for the search phrase "this content" takes many, many minutes to complete. After Windows Search Service is enabled and configured, the same search of the same data completes in approximately ten seconds or less.

## Tips
1. Windows Search Service uses the system drive to store index files by default. In Windows Search Service there is an option to move the index files location to a different drive, which decreases contention for the system drive, reduces system drive fragmentation and lowers system drive space issue risks.
2. Windows Search Service can be added via the Windows Server GUI, add Windows Feature, Search Service.
3. Windows Search Service can be added via PowerShell, `Add-WindowsFeature Search-Service`
4. Configure Windows Search Service via the Windows Server GUI, Start Button, type `Indexing Options`, add the shared folders to the list of indexed folders.
5. Move the indexed files folder containing the database via the Windows Server GUI, Start Button, type `Indexing Options`, Advanced button, Index location.
6. Have a lot of data (Less than 1TB) to index? Use a dedicated disk for the Index Files location.
7. Windows Search Service log entries are available for review via Event Viewer, Application Log.
8. Windows Search Service log entries are available for review via PowerShell `Get-Eventlog -LogName Application -Newest 10 -Source 'Windows Search Service'`
9. Tested setting `HKLM\SOFTWARE\Microsoft\Windows Search\Gathering Manager\DisableBackoff` to "1", VM gained 20% CPU on average for a dedicated file server. Don't use this setting on servers hosting apps, SQL, etc. (You should know better).
10. Have many TB of data to index? ***Don't use Windows Search Service***. I tried everything and MS says Windows Search Service is not supported in Enterprise scenarios. However, they do not define limits or criteria for the scenarios. I can confirm that multi-TB data share scenarios will ultimately fail. Even after "tuning" the server and using all tips shown above, the index database file will grow far past 10GB (the SQL Server Express engine limit) and eventually corrupt. The Windows Search Service will automatically begin rebuilding the index but during the many days/weeks of rebuild, clients will receive incomplete search results without any warning that the results are incomplete. The MS supported scenario for multi-TB data shares is SharePoint search.
