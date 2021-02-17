### Pre-requisites
1. An [Azure Account](https://azure.microsoft.com/en-us/free/), there are free and paid options
2. A [SendGrid account](https://signup.sendgrid.com),  there are free trial, free and paid options
3. Code from this post!
4. Approximately X time to implement and test
5. Optional: A custom domain for the contact form html and Azure function Application Program Interface (API)

**Securing your Azure and Sendgrid accounts with Two Factor Authentication (2FA) is recommended. 2FA will not have an effect on how the Azure function or Sendgrid API works.

### Implementation Steps
1. In the Azure portal, create an Azure Function using the steps outlined in [this guide](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-azure-function). Your Resource Group, Function App name and Storage Account will be unique to your Azure account. The Function App name must be globally unique. Monitoring and Tags are optional and do not have an impact on how the Function app works.
![az-function-ps-sendgrid-1.PNG](https://kamsalisbury.github.io/images/az-function-ps-sendgrid-1.PNG "Sendgrid Configuration 1")
![az-function-ps-sendgrid-2.PNG](https://kamsalisbury.github.io/images/az-function-ps-sendgrid-2.PNG "Sendgrid Configuration 2")
![az-function-ps-sendgrid-3.PNG](https://kamsalisbury.github.io/images/az-function-ps-sendgrid-3.PNG "Sendgrid Configuration 3")
2. **Security Steps!** In the Azure portal, Azure Function App Blade, Configuration, General Settings: Change FTP state to FTPS only. In the Azure portal, Azure Function App Blade, Custom domains, toggle HTTPS Only to On. Both of these settings changes ensure any communication with the function app occurs over an encrypted data stream.
3. **Cost Protection Step!** In the Azure portal, Azure Function App Blade, Configuration, Function Runtime Settings: Change Daily Usage Quota to 12903 (Azure grants Function apps 400,000 GB Seconds of monthly execution. Dividing the total grant by 31 calendar days limits the Function app into a cost prevention posture)
4. In the Sendgrid web portal, create a dedicated Sendgrid API Key. Select Email API, Integration Guide, Web API, then Curl. I recommend selecting "Curl" because the curl command line utility provides another method of troubleshooting the SendGrid API.
![az-function-ps-sendgrid-4.PNG](https://kamsalisbury.github.io/images/az-function-ps-sendgrid-4.PNG "Sendgrid Configuration 4")
Provide a unique API Key Name and click Create Key. Copy the resulting API Key for the next step.
![az-function-ps-sendgrid-5.PNG](https://kamsalisbury.github.io/images/az-function-ps-sendgrid-5.PNG "Sendgrid Configuration 5")
5. In the Azure portal, Azure Function App Blade, Configuration, Application Settings: Create a New Application Setting entry in your Function App called SENDGRID_APIKEY that contains your SendGrid API key created in step four. Create a second Application Setting entry in your Function App called SENDGRID_TO that contains the email address that will be receiving emails from the Function app through Sendgrid. Application Settings cannot use a dash in the variable name.
6. In the Azure portal, Azure Function App Blade, Functions, click Add. Select HTTP Trigger, provide a New Function name and select Anonymous for Authorization level. Note that New Function name will appear in the Azure Function URL.
![az-function-ps-sendgrid-6.PNG](https://kamsalisbury.github.io/images/az-function-ps-sendgrid-6.PNG "Sendgrid Configuration 6")
7. **Optional Step!** The default Function App URL will be HTTPS protected, however, a custom domain and matching HTTPS certificate can also be used. In the Azure portal, Azure Function App Blade, Custom Domains, configure a custom domain to be used by the Azure Function by following [this tutorial](https://docs.microsoft.com/en-us/azure/app-service/app-service-web-tutorial-custom-domain). Then, in the Azure portal, Azure Function App Blade, TLS/SSL settings, select Private Key Certificates, then Create App Service Managed Certificate. See [this guide](https://docs.microsoft.com/en-us/azure/app-service/configure-ssl-certificate#create-a-free-certificate-preview) about free and App Service Managed Certificates.
8. In the Azure portal, Azure Function App Blade, Functions, select the new function created in step six. Select Code + Test, then Get Function URL. Copy the URL for use in the HTML form.
![az-function-ps-sendgrid-7.PNG](https://kamsalisbury.github.io/images/az-function-ps-sendgrid-7.PNG "Sendgrid Configuration 7")
9. In the Azure portal, Azure Function App Blade, Functions, select the new function created in step six, replace the template code with the following PowerShell code. Save the updated code.
    ```
    ## Azure Function, HTTP trigger, HTML Contact Form Submit Method GET, PowerShell send email using SendGrid API

    # Default namespace
    using namespace System.Net

    # Input bindings are passed in via param block.
    param($Request, $TriggerMetadata)

    # Write to the Azure Functions log stream.
    Write-Host "PowerShell HTTP trigger function processed a request."

    # These variables can be used to see variables after the function processes them
    # $Request
    # $get = $Request.Query
    # $headers = $Request.Headers
    # PowerShell Display all environment variables
    # $debug = (dir env:)

    # PowerShell execute SendGrid web API call
    $SGurl = "https://api.sendgrid.com/v3/mail/send"
    $SGfrom = $Request.Query.Email
    $SGsubject = $Request.Query.Subject
    $SGbody = $Request.Query.Message
    $SGheaders = @{}
    # Azure Application Settings are available here in the PowerShell Function, using them to add values to an array variable
    $SGheaders.Add("Authorization","Bearer $env:SENDGRID_APIKEY")
    $SGheaders.Add("Content-Type", "application/json")
    # Building the message body in JSON
    $SGjson = [ordered]@{
        personalizations= @(@{to = @(@{email = "$env:SENDGRID_TO"})
        subject = "$SGsubject" })
        from = @{email = "$SGfrom"}
        content = @( @{ type = "text/plain"
        value = "$SGbody" }
        )} | ConvertTo-Json -Depth 10

    Write-Host "PowerShell Azure Function executing HTTPS POST to SendGrid API"
    Invoke-RestMethod -Uri $SGurl -Method Post -Headers $SGheaders -Body $SGjson -Verbose

    # Build the response page
    $submitted = @"
    <html>
    <head>
    </head>
    <body>
    <p>Thank you $SGfrom, I will review the submitted information and respond soon.<br /><br />
    <a href="https://yourdomain.tld">yourdomain.tld</a>
    </p>
    </body>
    </html>
    "@

    # Associate values to output bindings by calling 'Push-OutputBinding'.
    Push-OutputBinding -Name Response -Value (@{ 
        headers = @{"content-type" = "text/html"}
        Body = $submitted
    })
    ```
10. Create a Static Web App using [this guide](https://docs.microsoft.com/en-us/azure/static-web-apps/getting-started?tabs=vanilla-javascript), or edit an existing web page. The key changes to the HTML form tags are; Specify the correct action URL. Specify get method. Each input parameter must have a name attribute. Javascript is not required at all for the form to function.
    ```
    <form action="https://yourfunctionappname.azurewebsites.net/api/yourfunctionname" target="_self" method="get">
    Email: <input name="Email" class="input" type="text" placeholder="you@yourdomain.tld" /><br />
    Subject: <input name="Subject" class="input" type="text" placeholder="Consultation topic" /><br />
    Message: <br />
    <textarea name="Message" rows="10" cols="80" class="input" type="text" placeholder="Additional details"></textarea><br />
    <input type="submit" value="Submit" " />
    </form>
    ```
11. Now test for effect!
![az-function-ps-sendgrid-8.PNG](https://kamsalisbury.github.io/images/az-function-ps-sendgrid-8.PNG "Sendgrid Configuration 8")

A successful test :) Of course, the email specified in the Application Settings variable SENDGRID_TO, should receive the submitted information in email.
![az-function-ps-sendgrid-9.PNG](https://kamsalisbury.github.io/images/az-function-ps-sendgrid-9.PNG "Sendgrid Configuration 9")

12. In the Sendgrid web portal, select the confirmation box to the left of "I've executed the code above." and then "Next: Verify Integration". The Sendgrid web portal will validate if an email was submitted through the Sendgrid API using the Application Settings variable SENDGRID_APIKEY.

![az-function-ps-sendgrid-10.PNG](https://kamsalisbury.github.io/images/az-function-ps-sendgrid-10.PNG "Sendgrid Configuration 10")

### Recap
Azure Functions insulate your code from platform patching and infrastructure changes while providing consistent robust performance. In this step by step guide, an Azure PowerShell Function was implemented to send HTML form submitted data as email through the Sendgrid API. Additionally, a quota was specified to minimize unexpected Azure account charges.
