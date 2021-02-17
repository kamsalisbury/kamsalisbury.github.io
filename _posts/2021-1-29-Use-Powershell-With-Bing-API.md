[Bing provides an API for automating URL submissions](https://www.bing.com/webmasters/url-submission-api#Documentation) among [other things](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bing.webmaster.api.interfaces.iwebmasterapi.submiturlbatch?view=bing-webmaster-dotnet#Microsoft_Bing_Webmaster_Api_Interfaces_IWebmasterApi_SubmitUrlBatch_System_String_System_Collections_Generic_List_System_String__). Just one of many tasks in grooming Search Engine Optimization is to submit new content URLs to Bing's web crawler.

To meet this need I created a powershell script to automate the url submission instead of using the web site interface [Example-Submit-URL-Bing-API.ps1](https://github.com/kamsalisbury/ps/blob/examples/Example-Submit-URL-Bing-API.ps1) was created. Wrapping the code from that script into a git process powershell script can look like this.

```
# First, process the git repository push
$CommitMessage = Read-Host -Prompt 'What is the one line sumamry of this git push?'
git add *
git commit -m "$CommitMessage"
git push

# Then submit the aplicable URL
$BingAPIkey = .\BingWebmasterAPI.key # Remember, saving your keys in clear tesxt is always bad, so secure them https://kamsalisbury.github.io/Do-Not-Store-Keys-In-PowerShell-Scripts/
$Domain = Read-Host -Prompt 'Input the main domain url ex. https://domain.tld'
$URI = Read-Host -Prompt 'Input a URL to submit ex. https://domain.tld/page.html'
$BingAPIendpoint = 'https://ssl.bing.com/webmaster/api.svc/json/SubmitUrl?apikey=' + $BingAPIkey
$Body = @{
    siteUrl = "$Domain"
    url = "$URI"
}

Invoke-RestMethod -Method Post -Uri "$BingAPIendpoint" -Body (ConvertTo-Json $Body) -ContentType 'application/json'
```
