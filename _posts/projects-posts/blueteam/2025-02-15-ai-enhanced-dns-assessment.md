---
layout: post
title: AI Enhanced DNS Assessment
gh-repo: 4D5A/ai_tools
gh-badge: [follow, star]
categories: [projects]
tags: [Systems-Administration-Tools, infosec]
after-content: [disclaimer-notice.html]
---

This PowerShell script pulls a domain's MX, SPF, DMARC, DKIM, and TXT records, hands them to an LLM (in this example, OpenAI's API) along with a prompt asking for security recommendations, and writes the result out as an HTML report. It's a quick first pass for spotting missing or misconfigured email-authentication records. For the manual, step-by-step version of what this script is checking for, see the [SPF]({{ site.baseurl }}{% post_url blog-posts/2022-04-23-configuring-sender-policy-framework-in-exchange-online %}), [DKIM]({{ site.baseurl }}{% post_url blog-posts/2022-04-23-how-to-configure-dkim-in-microsoft-365 %}), and [DMARC]({{ site.baseurl }}{% post_url blog-posts/2022-05-23-implementing-dmarc-for-enhanced-email-security-a-guide-for-systems-administrators %}) posts.

## How it works

1. `Get-DnsRecords` wraps `Resolve-DnsName` to fetch MX and TXT records for the target domain, then filters the TXT records for the ones that look like SPF (`v=spf1`), DMARC (`v=DMARC1`), or DKIM entries.
2. `Query-OpenAI` packages those records into a JSON blob and sends it to the Chat Completions API with a system prompt asking specifically for email security and DNS posture recommendations.
3. `Generate-HtmlReport` renders the recommendations into a simple HTML page.

## Usage

```powershell
.\GenerateDnsReport.ps1 -Domain "example.com"
```

This produces `example.com-DnsReport.html` in the current directory.

**Before running this**, replace the placeholder API key in `Query-OpenAI` with your own OpenAI API key. Better yet, pull it from an environment variable or a secret store instead of hardcoding it in the script.

```powershell
<#
.SYNOPSIS
Generates an HTML report with DNS records (MX, SPF, DMARC, DKIM, and TXT) and recommendations to improve security for a given domain.

.PARAMETER Domain
The domain name for which the DNS records will be fetched.

.EXAMPLE
.\GenerateDnsReport.ps1 -Domain "example.com"
#>

param (
    [Parameter(Mandatory = $true)]
    [string]$Domain
)

function Get-DnsRecords {
    param (
        [string]$Domain,
        [string]$RecordType
    )
    try {
        $records = Resolve-DnsName -Name $Domain -Type $RecordType
        return $records
    } catch {
        Write-Error "Failed to get $RecordType records for $Domain"
        return @()
    }
}

function Query-OpenAI {
    param (
        [string]$Prompt
    )
    $apiKey = "Your-OpenAI-API-Key"
    $apiUrl = "https://api.openai.com/v1/chat/completions"

    $body = @{
        model = "gpt-4"
        messages = @(@{
            role = "system"; content = "You are an AI assistant that provides cybersecurity recommendations based on DNS records."
        }, @{
            role = "user"; content = $Prompt
        })
        max_tokens = 1024
        temperature = 0.7
    } | ConvertTo-Json -Depth 10

    $headers = @{
        "Authorization" = "Bearer $apiKey"
        "Content-Type"  = "application/json"
    }

    try {
        $response = Invoke-RestMethod -Uri $apiUrl -Method Post -Headers $headers -Body $body
        return $response.choices[0].message.content.Trim()
    } catch {
        Write-Error "Failed to query OpenAI API: $_"
        return $null
    }
}

function Generate-HtmlReport {
    param (
        [string]$Domain,
        [array]$MXRecords,
        [array]$SPFRecords,
        [array]$DMARCRecords,
        [array]$DKIMRecords,
        [array]$TXTRecords,
        [string]$Recommendations
    )

    $html = @"
<html>
<head>
    <title>DNS Report for $Domain</title>
    <style>
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid black; padding: 8px; text-align: left; }
        th { background-color: #f2f2f2; }
    </style>
</head>
<body>
    <h1>DNS Report for $Domain</h1>
    <h2>DNS Records</h2>
    <pre>$Recommendations</pre>
</body>
</html>
"@

    return $html
}

# Fetch DNS records
$MXRecords = Get-DnsRecords -Domain $Domain -RecordType "MX"
$SPFRecords = Get-DnsRecords -Domain $Domain -RecordType "TXT" | Where-Object { $_.Strings -match "v=spf1" }
$DMARCRecords = Get-DnsRecords -Domain $Domain -RecordType "TXT" | Where-Object { $_.Strings -match "v=DMARC1" }
$DKIMRecords = Get-DnsRecords -Domain $Domain -RecordType "TXT" | Where-Object { $_.Strings -match "DKIM" }
$TXTRecords = Get-DnsRecords -Domain $Domain -RecordType "TXT"

# Prepare prompt for AI
$dnsData = @{
    MX = $MXRecords | ForEach-Object { @{ Preference = $_.Preference; Exchange = $_.Exchange } }
    SPF = $SPFRecords | ForEach-Object { @{ SPFRecord = $_.Strings } }
    DMARC = $DMARCRecords | ForEach-Object { @{ DMARCRecord = $_.Strings } }
    DKIM = $DKIMRecords | ForEach-Object { @{ DKIMRecord = $_.Strings } }
    TXT = $TXTRecords | ForEach-Object { @{ TXTRecord = $_.Strings } }
} | ConvertTo-Json -Depth 10

$prompt = @"
Based on the following DNS records for the domain '$Domain', provide:
1. Recommended changes to improve email security and reliability.
2. Suggestions for enhancing the domain's overall security posture.


DNS Records:
$dnsData
"@

# Query OpenAI API
Write-Output "Querying OpenAI for recommendations..."
$recommendations = Query-OpenAI -Prompt $prompt

if ($recommendations) {
    # Generate HTML report
    $htmlReport = Generate-HtmlReport -Domain $Domain -MXRecords $MXRecords -SPFRecords $SPFRecords -DMARCRecords $DMARCRecords -DKIMRecords $DKIMRecords -TXTRecords $TXTRecords -Recommendations $recommendations

    # Save the HTML report
    $reportPath = "$Domain-DnsReport.html"
    $htmlReport | Out-File -FilePath $reportPath -Encoding UTF8

    Write-Output "HTML report generated: $reportPath"
} else {
    Write-Error "Failed to retrieve recommendations from OpenAI API."
}
```

**A note on the AI output:** treat the LLM's recommendations as a starting point, not gospel. Verify anything it suggests against the actual DNS records and your own understanding of SPF/DKIM/DMARC before acting on it.
