---
layout: post
title: "Using Maester and HRProvisioningTests to Unit Test Your HR-Driven Provisioning"
subtitle: "Create Simplified Test Results for Non-Technical Stakeholders"
cover-img: assets/img/HRTest10.png
thumbnail-img: assets/img/HRTest10.png
share-img: assets/img/HRTest10.png
tags: [EntraID, IAM, Pester, HR, PowerShell]
---

## Using Maester and HRProvisioningTests to Unit Test Your HR DrivenProvisioning

## Table of Contents

1. [Introduction](##introduction)
2. [Understanding Unit Testing](##understanding-unit-testing)
   - [Why Unit Test HR-Driven Provisioning?](##why-unit-test-hr-driven-provisioning)
3. [Introducing Maester and HRProvisioningTests](##introducing-maester-and-hrprovisioningtests)
   - [Maester](##maester)
   - [HRProvisioningTests](##hrprovisioningtests)
4. [Steps to Unit Test HR-DrivenProvisioning](##steps-to-unit-test-hrdrivenprovisioning)
   - [1. Setting Up the Testing Environment](##1-setting-up-the-testing-environment)
   - [2. Generating Test Cases](##2-generating-test-cases)
   - [3. Executing Tests](##3-executing-tests)
5. [Refactoring the Test Script](##refactoring-the-test-script)
   - [Adding Maester Functionality](##adding-maester-functionality)
   - [Updating TestResult Configuration](##updating-testresult-configuration)
   - [Modifying Test Cases for Maester Visualization](##modifying-test-cases-for-maester-visualization)
6. [Full Script: Invoke-HRTests.ps1](##full-script-invoke-hrtestsps1)
7. [Conclusion](##conclusion)


## Introduction

In IAM integration, ensuring the accuracy and reliability of [HR-driven provisioning systems](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/what-is-hr-driven-provisioning) is crucial. Gaining the confidence of all stakeholders is a continuous effort, with unit testing being crucial. This guide explains how to use Maester and HRProvisioningTests for unit testing HR-driven provisioning, and presents results for non-technical stakeholders.

## Understanding Unit Testing

Unit testing involves testing individual software components in isolation to ensure they perform as expected. This method helps catch bugs early, reducing the cost of fixing them and ensuring a high-quality final product.

#### Why Unit Test HR-Driven Provisioning?

Unit testing HR-driven provisioning is essential because it ensures that digital identities are created accurately and reliably, based on the information from the HR system Microsoft states:

> "HR-driven provisioning creates digital identities based on the HR system. The HR system becomes the authority for these identities and often starts many provisioning processes. For instance, when a new employee is added to the HR system, it triggers the creation of a user account in Active Directory, which Microsoft Entra Connect then provisions to Microsoft Entra ID."

By unit testing HR-driven provisioning, you can:

* Identify and fix bugs early in the development process.
* Ensure that each component of the provisioning system performs as expected.
* Maintain the security and integrity of your Microsoft 365 environment.
* Gain the confidence of stakeholders by demonstrating the reliability and accuracy of the provisioning process.

Incorporating unit tests helps reduce the cost of fixing issues later on and contributes to delivering a high-quality final product.

In HR-driven integrations with Entra ID or Active Directory, complex attribute mappings can be considered as individual units to test.



## Introducing Maester and HRProvisioningTests Powershell Module

#### Maester

According to one of the creators of Maester, [**Thomas Naunheim**](https://www.cloud-architekt.net/), Maester is a PowerShell-based test automation framework designed to help you monitor and maintain the security configuration of your Microsoft 365 environment. It provides a user-friendly interface and powerful features to create comprehensive test cases that cover a wide range of scenarios.

#### HRProvisioningTests

HRProvisioningTests is a PowerShell module that enhances Pester Tests by generating pre-built Data driven Test Suite and utilities for HR-driven provisioning systems. Martin Rubrik, the creator, has thoroughly explained its functionality in his blog: [unit testing your HR driven provisioning rules | flesh shapes the day.](https://martin.rublik.eu/2024/05/23/pester-and-HR-provisioning.html##extracting-the-attribute-mappings-from-the-provisioning-schema)

## Steps to Unit Test HR-Driven Provisioning



#### 1. Setting Up the Testing Environment

Begin by setting up your development environment with Installing Maester, Pester and HRProvisioningTests Module.

```powershell
Install-Module HRProvisioningTests -Force
Install-Module Pester -SkipPublisherCheck -Force
Install-Module Maester
```

#### 2. Generating Test Cases

Create a test suite by connecting to MgGraph, specifying the HR Provisioning app's display name, and defining an output directory:

```powershell
Connect-MgGraph Synchronization.Read.All
New-HRProvisioningRulesTestSuite -TestSuiteDirectory C:\TEMP\HR2ADUnitTests -HRApplicationDisplayName 'HR to Active Directory User Provisioning'
```

The New-HRProvisioningRulesTestSuite generates a test suite with a specific directory structure.

![Diagram of SAP HR integrations.](/assets/img/HRTest1.png)

![A screenshot of a computer screen](/assets/img/HRTest2.png)

![A screenshot of a computer Description automatically generated](/assets/img/HRTest3.png)

![A screenshot of a computer Description automatically generated](/assets/img/HRTest4.png)

Modify the JSON files in the `Data` subdirectories to create test cases. Add or delete JSON files as needed. If a test case is not required or can be tested with Pester unit tests, delete the entire directory and data subdirectory. Here is an example modification for the `givenName` case.

![A screen shot of a computer code Description automatically generated](/assets/img/HRTest4.png)

We also have a Test Script: Invoke-HRTests.ps1 that created as part of the Test Suit to trigger pester tests.

```powershell
$pesterConfig = New-PesterConfiguration @{
  Run = @{ Container = New-PesterContainer -Path "$PSScriptRoot\Tests" }
  Output = @{ Verbosity = 'Detailed' }
}

Invoke-Pester -Configuration $pesterConfig
Write-Host "Pester report saved to: $outFile"
```
#### 3. Executing Tests Refactoring the Test Script

The **Invoke-HRTests.ps1** script needs to be modified to include the Maester functionality. One of the major benefits of Maester is the HTML Test report it produces, which facilitates easy access to Test Insights. Enhance the script to include Maester's HTML report generation capabilities:

I took the inspiration from the [Invoke-Maester.ps1](https://github.com/maester365/maester/blob/d67de01cd7286e4207a9fa6fdcef5b646517247c/powershell/public/Invoke-Maester.ps1) the Maester Module to refactor the Invoke-HRTests.ps1 file.

First we set the path for Test Results HTML Output path.


```powershell
$outHTMLFile = "$PSScriptRoot\test-results\$((Get-Date).ToString('yyyy-MM-yy_hh-mm-ss'))-TestResults.html"
```

Remove old Test Results if present. As We would like to keep our Repository clean.
This is optional, comment it out if you feel to keep historical test results.

```powershell
$pesterResults = Invoke-Pester -Configuration $pesterConfig
$maesterResults = ConvertTo-MtMaesterResult $pesterResults
$output = Get-MtHtmlReport -MaesterResults $maesterResults
$output | Out-File -FilePath $outHTMLFile -Encoding UTF8
Write-Host "🔥 Maester test report generated at $outHTMLFile"
```
Generated **Invoke-HRTests.ps1** has a default Pester Configuration. For integrating with Pester I make small changes to it. I need to add Passthru Run configuration.
```powershell
Run = @{
    Container = New-PesterContainer -Path "$PSScriptRoot/Tests"
    PassThru = true # Added PassThru to convert the PesterResults to a variable
}
```


I have changed the TestResult Output Format to Junit. But this is optional. I prefer Junit for my CI/CD build.

```powershell
TestResult = @{
			OutputFormat  = 'JUnitXml' ##Changed to JunitXML
			TestSuiteName = "$($config.HRApplicationDisplayName) Tests"
			Enabled       = $true
			OutputPath    = "$outFile" ##Set the Path to the test-results folder
		}
```

Now it is time to invoke the Pester Tests. Here we need to explicitly define the function **ConvertTo-MtMaesterResult** as this is an internal function in Maester module.
Great thing about Maester module is that source code is available online and we can stand on the shoulder of the Giants and reuse the same code.

I only have to make a small modification. I set the Result Detail to Null as we do not have access to internal module variables and we do not need any details.

```powershell
##ResultDetail    = $__MtSession.TestResultDetail[$test.ExpandedName] ## I need to Remove this internal variable
ResultDetail    = $null
```


```powershell
$pesterResults = Invoke-Pester -Configuration $pesterConfig ## Store the Pester Results in a variable
Write-Host "🔥  Pester report saved to: $outFile"
$maesterResults = ConvertTo-MtMaesterResult $PesterResults  ## Convert the Pester Results to Maester Results
$output = Get-MtHtmlReport -MaesterResults $maesterResults 
$output | Out-File -FilePath $outHTMLFile -Encoding UTF8
Write-Host "🔥 Maester test report generated at $outHTMLFile" 
Invoke-Item $outHTMLFile | Out-Null ##Optional
Write-Host "`nTests Passed ✅: $($pesterResults.PassedCount), " -NoNewline -ForegroundColor Green
Write-Host "Failed ❌: $($pesterResults.FailedCount), " -NoNewline -ForegroundColor Red
Write-Host "Skipped ⚫: $($pesterResults.SkippedCount)`n" -ForegroundColor DarkGray
```

Once we are done refactoring the **Invoke-HRTests.ps1**  code we need one small adjustment on Test Cases present for each attribute within the directory.
For Maester visualization to work we need to add Tags for each test cases.

Add tags to test cases for better visualization. For example:

```powershell
Describe 'givenName' -Tag "HR", "givenName" {
  # Test logic here
}
```

Now it is time to Invoke the test.

![A screenshot of a computerDescription automatically generated](/assets/img/HRTest5.png)


Here is the complete **Invoke-HRTests.ps1** file.

```powershell
##Region Maester Internal Helper Functions

<##
.SYNOPSIS
  Converts Pester results to the Maester test results format which includes additional information.
##>

function ConvertTo-MtMaesterResult {
	[CmdletBinding()]
	param(
		## The Pester test results returned from Invoke-Pester -PassThru
		[Parameter(Mandatory = $true)]
		[psobject] $PesterResults
	)

	function GetTenantName() {
		if (Test-MtConnection Graph) {
			$org = Invoke-MtGraphRequest -RelativeUri 'organization'
			return $org.DisplayName
		}
		elseif (Test-MtConnection Teams) {
			$tenant = Get-CsTenant
			return $tenant.DisplayName
		}
		else {
			return 'TenantName (not connected to Graph)'
		}
	}

	function GetTenantId() {
		if (Test-MtConnection Graph) {
			$mgContext = Get-MgContext
			return $mgContext.TenantId
		}
		elseif (Test-MtConnection Teams) {
			$tenant = Get-CsTenant
			return $tenant.TenantId
		}
		else {
			return 'TenantId (not connected to Graph)'
		}
	}

	function GetAccount() {
		if (Test-MtConnection Graph) {
			$mgContext = Get-MgContext
			return $mgContext.Account
			##} elseif (Test-MtConnection Teams) {
			##    $tenant = Get-CsTenant ##ToValidate: N/A
			##    return $tenant.DisplayName
		}
		else {
			return 'Account (not connected to Graph)'
		}
	}

	function GetTestsSorted() {
		## Show passed and failed tests first by name then show not run tests
		$activeTests = $PesterResults.Tests | Where-Object { $_.Result -eq 'Passed' -or $_.Result -eq 'Failed' } | Sort-Object -Property Name
		$inactiveTests = $PesterResults.Tests | Where-Object { $_.Result -ne 'Passed' -and $_.Result -ne 'Failed' } | Sort-Object -Property Name

		## Convert to array and add, if not when only one object is returned it doesn't create an array with all items.
		return @($activeTests) + @($inactiveTests)
	}

	function GetFormattedDate($date) {
		if (!$IsCoreCLR) {
			## Prevent 5.1 date format to json issue
			return $date.ToString('o')
		}
		else {
			return $date
		}
	}

	function GetMaesterLatestVersion() {
		if (Get-Command 'Find-Module' -ErrorAction SilentlyContinue) {
			return (Find-Module -Name Maester).Version
		}

		return 'Unknown'
	}

	##if(Test-MtConnection Graph) { ##ToValidate: Issue with -SkipGraphConnect
	##    $mgContext = Get-MgContext
	##}

	##$tenantId = $mgContext.TenantId ?? "Tenant ID (not connected to Graph)"
	$tenantId = GetTenantId
	$tenantName = GetTenantName
	##$account = $mgContext.Account ?? "Account (not connected to Graph)"
	$account = GetAccount

	$currentVersion = ((Get-Module -Name Maester).Version | Select-Object -Last 1).ToString()
	$latestVersion = GetMaesterLatestVersion

	$mtTests = @()
	$sortedTests = GetTestsSorted

	foreach ($test in $sortedTests) {

		$name = $test.ExpandedName
		$helpUrl = ''

		$start = $name.IndexOf('See https')
		## Get the Help Url from the message and the ID
		if ($start -gt 0) {
			$helpUrl = $name.Substring($start + 4).Trim() ##Strip away the "See https://maester.dev" part
			$name = $name.Substring(0, $start).Trim() ##Strip away the "See https://maester.dev" part
		}
		$mtTestInfo = [PSCustomObject]@{
			Name            = $name
			HelpUrl         = $helpUrl
			Tag             = ($test.Block.Tag + $test.Tag | Select-Object -Unique)
			Result          = $test.Result
			ScriptBlock     = $test.ScriptBlock.ToString()
			ScriptBlockFile = $test.ScriptBlock.File
			ErrorRecord     = $test.ErrorRecord
			Block           = $test.Block.ExpandedName
			##ResultDetail    = $__MtSession.TestResultDetail[$test.ExpandedName] ## I need to Remove this internal variable
			ResultDetail    = $null
		}
		$mtTests += $mtTestInfo
	}

	$mtBlocks = @()
	foreach ($container in $PesterResults.Containers) {

		foreach ($block in $container.Blocks) {
			$mtBlockInfo = $mtBlocks | Where-Object { $_.Name -eq $block.Name }
			if ($null -eq $mtBlockInfo) {
				$mtBlockInfo = [PSCustomObject]@{
					Name         = $block.Name
					Result       = $block.Result
					FailedCount  = $block.FailedCount
					PassedCount  = $block.PassedCount
					SkippedCount = $block.SkippedCount
					NotRunCount  = $block.NotRunCount
					TotalCount   = $block.TotalCount
					Tag          = $block.Tag
				}
				$mtBlocks += $mtBlockInfo
			}
			else {
				$mtBlockInfo.FailedCount += $block.FailedCount
				$mtBlockInfo.PassedCount += $block.PassedCount
				$mtBlockInfo.SkippedCount += $block.SkippedCount
				$mtBlockInfo.NotRunCount += $block.NotRunCount
				$mtBlockInfo.TotalCount += $block.TotalCount
			}
		}
	}

	$mtTestResults = [PSCustomObject]@{
		Result         = $PesterResults.Result
		FailedCount    = $PesterResults.FailedCount
		PassedCount    = $PesterResults.PassedCount
		SkippedCount   = $PesterResults.SkippedCount
		TotalCount     = $PesterResults.TotalCount
		ExecutedAt     = GetFormattedDate($PesterResults.ExecutedAt)
		TenantId       = $tenantId
		TenantName     = $tenantName
		Account        = $account
		CurrentVersion = $currentVersion
		LatestVersion  = $latestVersion
		Tests          = $mtTests
		Blocks         = $mtBlocks
	}

	return $mtTestResults
}





##Main Region
try {
	$context = Get-MgContext

	if (-not $context) {
		Write-Warning "You are not connected to MGGraph. Please run: 'Connect-MGGraph -Scopes Synchronization.ReadWrite.All'"
		return
	}
	else {
		if (-not ('Synchronization.ReadWrite.All' -in $context.Scopes)) {
			Write-Warning "Missing Synchronization.Read.All context. Please run: 'Connect-MGGraph -Scopes Synchronization.ReadWrite.All'"
			return    
		}
	}

	##Set Maester Activity and Progress
	$maesterResults = $null    
    
	
	$config = Get-Content -Raw "$PSScriptRoot\Config\Config.json" | ConvertFrom-Json
	
	
	$outFile = "$PSScriptRoot\test-results\$((Get-Date).tostring('yyyy-MM-yy_hh-mm-ss'))-TestResults.xml"
	##Set HTML Output Path

	$outHTMLFile = "$PSScriptRoot\test-results\$((Get-Date).tostring('yyyy-MM-yy_hh-mm-ss'))-TestResults.html"

	##Remove Old Test Results
	Get-ChildItem -Path "$PSScriptRoot\test-results" | 
		ForEach-Object { Remove-Item -Path $_.FullName -Force }

	$pesterConfig = New-PesterConfiguration @{
		Run        = @{
			Container = New-PesterContainer -Path "$PSScriptRoot\Tests" 
			PassThru  = $true ##Added PassThru to Convert the PesterResults to a variable
		}
		Output     = @{
			Verbosity           = 'None'
			StackTraceVerbosity = 'Filtered'
			
		}
		TestResult = @{
			OutputFormat  = 'JUnitXml' ##Changed to JunitXML
			TestSuiteName = "$($config.HRApplicationDisplayName) Tests"
			Enabled       = $true
			OutputPath    = "$outFile" ##Set the Path to the test-results folder
		}
	}

	$pesterResults = Invoke-Pester -Configuration $pesterConfig ## Store the Pester Results in a variable
	Write-Host "🔥  Pester report saved to: $outFile"

	$maesterResults = ConvertTo-MtMaesterResult $PesterResults  ## Convert the Pester Results to Maester Results
	
	$output = Get-MtHtmlReport -MaesterResults $maesterResults 
	$output | Out-File -FilePath $outHTMLFile -Encoding UTF8
	Write-Host "🔥 Maester test report generated at $outHTMLFile" 

	Invoke-Item $outHTMLFile | Out-Null

	Write-Host "`nTests Passed ✅: $($pesterResults.PassedCount), " -NoNewline -ForegroundColor Green
	Write-Host "Failed ❌: $($pesterResults.FailedCount), " -NoNewline -ForegroundColor Red
	Write-Host "Skipped ⚫: $($pesterResults.SkippedCount)`n" -ForegroundColor DarkGray

}
catch {
	throw $_
}



```

And Pester unit test file for givenName.

**givenName.tests.ps1**

```powershell
##givenName.tests.ps1
Describe 'givenName' -Tag "HR","givenName" {
## load the json config files
$testConfig=@()
ls "$PSScriptRoot\Data\\*.json" | foreach {
$configObject = Get-Content -Raw $\_.FullName | ConvertFrom-Json
$ht=@{}
$configObject.psobject.properties | foreach {$ht.Add($\_.Name,$\_.Value)}
$tenantConfig=ls "$PSScriptRoot\..\..\Config\Config.json" | get-content -raw | ConvertFrom-Json
$ht.Add("SynchronizationTemplateId",$tenantConfig.SynchronizationTemplateId)
$ht.Add("ServicePrincipalId",$tenantConfig.ServicePrincipalId)
$testConfig+=$ht
}
## for each test structure: test Parsing, Evaluation and Expected result
It "When: '<Description>', it returns: '<ExpectedResult>'" -ForEach $testConfig {
   $propertiesHT = @()
   foreach($attr in $InputAttributes.psobject.properties){
$propertiesHT+=@{'key'=$attr.Name; 'value'=$attr.Value}
}
$params=@{
expression = $Expression
targetAttributeDefinition = $null
testInputObject = @{
definition = $null
properties = $propertiesHT
}
}

$retval = Invoke-MgParseServicePrincipalSynchronizationTemplateSchemaExpression -ServicePrincipalId $ServicePrincipalId -BodyParameter $params -SynchronizationTemplateId $SynchronizationTemplateId
$retval.ParsingSucceeded | Should -Be $true -Because "PARSING must succeed to determine EvaluationResult."
$retval.EvaluationSucceeded | Should -Be $true -Because "EVALUATION must succeed to determine EvaluationResult."
$retval.EvaluationResult | Should -Be $ExpectedResult
}
}
```

#### 4. Presenting Results to Non-Technical Stakeholders

Maester provides detailed reports that highlight which tests failed and why, making it easier to pinpoint and address issues. To communicate the results of unit tests to non-technical stakeholders, we can share the generated html file and stakeholders can open the Test Result in the preferred browser.

![A screenshot of a computerDescription automatically generated](/assets/img/HRTest6.png)

![A screenshot of a computerDescription automatically generated](/assets/img/HRTest7.png)

Maester provides detailed reports that highlight which tests failed and why, making it easier to pinpoint and address issues. The generated HTML file can be shared with non-technical stakeholders, who can then open the test results in their preferred browser.

Once the file is opened, stakeholders can utilize the filtering options available, allowing them to view only the failed test results. This focused view helps in understanding specific issues without getting overwhelmed by the entire dataset.

Additionally, by clicking on the Info Button associated with each test, stakeholders can access detailed explanations about the test case, including the objectives, the parameters tested, and the reasons for failure.

Communicating these results in a simplified and business-focused manner ensures that non-technical stakeholders can appreciate the value of these tests and understand their impact on the organization. This approach bridges the gap between technical and non-technical audiences, fostering a shared understanding of the importance of unit testing in maintaining the integrity of HR-driven provisioning systems.



![A screenshot of a computerDescription automatically generated](/assets/img/HRTest8.png)

They can filter the test results to only show failed test results to dig deeper. Click on the Info Button to see the details about the test cases.

![A screenshot of a computerDescription automatically generated](/assets/img/HRTest9.png)


![A screenshot of a computerDescription automatically generated](/assets/img/HRTest10.png)

## Conclusion

Unit testing HR-driven provisioning using `Maester` and `HRProvisioningTests` ensures reliability and efficiency. By following the steps outlined above, IAM Engineers can create robust test cases that validate their provisioning logic. Moreover, presenting the results in a simplified and business-focused manner helps non-technical stakeholders appreciate the value of these tests and understand their impact on the organization.