
## Custom Security Attribute in Microsoft Graph Powershell

This is a PowerShell function that creates custom security attributes in Azure Active Directory using the Microsoft Graph API. Custom security attributes are useful because they allow you to associate additional information with users or groups in your organization's directory.

The `New-SBCustomSecurityAttribute` function takes two parameters: `$AttributeSetName` and `$Attributes`. `$AttributeSetName` is the name of the attribute set that the custom security attribute should be associated with. `$Attributes` is an array of strings, where each string is the name of a custom security attribute.

The `Begin` block of the function first imports the necessary PowerShell modules and connects to the Microsoft Graph API. It then checks if an attribute set with the specified name already exists, and if not, creates a new attribute set.

The `Process` block of the function loops through each custom security attribute specified in `$Attributes`. For each attribute, it sanitizes the name (stripping out any characters that aren't alphanumeric or whitespace) and creates a unique ID for the attribute. It then checks if a custom security attribute with that ID already exists, and if not, creates a new custom security attribute with the sanitized name and a description based on the original attribute name.

The `End` block of the function disconnects from the Microsoft Graph API.

There are also two helper functions defined within this function: `New-AttributeSetParams` and `New-CustomAttributeParams`. These functions simply define the parameters that are required when creating a new attribute set or a new custom attribute definition.

Overall, this code allows you to easily create and manage custom security attributes in Azure Active Directory, which can help you better organize and manage your organization's directory.

```powershell


function New-SBCustomSecurityAttribute {
	[CmdletBinding()]
	param(
		[Parameter(Mandatory = $true)]
		[string]$AttributeSetName,
		
		[Parameter(Mandatory = $true, ValueFromPipeline = $true)]
		[string[]]$Attributes
	)

	Begin {
		Write-Verbose "Importing required PowerShell modules"
		Import-Module Microsoft.Graph.Identity.DirectoryManagement -DisableNameChecking
		Import-Module Microsoft.Graph.Applications -DisableNameChecking
		
		Connect-MgGraph
		$setAttribute = Get-MgDirectoryAttributeSet -AttributeSetId $AttributeSetName -ErrorAction SilentlyContinue
		if (!$setAttribute) {
			Write-Verbose "Creating new directory attribute set: $AttributeSetName"
			$params = New-AttributeSetParams -AttributeSetName $AttributeSetName
			$params | New-MgDirectoryAttributeSet -Verbose
		}
	}

	Process {
		foreach ($attribute in $Attributes) {
			$name = $attribute -replace ('[^a-zA-Z\d\s:]', '')
			
			$customSecurityAttributeID = $AttributeSetName + '_' + $name
			$customAttribute = Get-MgDirectoryCustomSecurityAttributeDefinition -CustomSecurityAttributeDefinitionId "$customSecurityAttributeID" -ErrorAction SilentlyContinue
			if (!$customAttribute) {
				Write-Verbose "Creating new custom security attribute definition: $attribute"
				
				$description = $attribute -replace ('[^a-zA-Z\d\s:]', ' ')
				$params = New-CustomAttributeParams -AttributeSetName $AttributeSetName `
												  -Name $name `
												  -Description $description
				$params | New-MgDirectoryCustomSecurityAttributeDefinition -Verbose
			}
			else {
				Write-Verbose "Custom Security Attribute $customSecurityAttributeID already exists"
			}
		}
	}

	End {
		if ($null -ne (Get-MgContext  -ErrorAction SilentlyContinue)) {
			Disconnect-MgGraph | Out-Null
		}
	}
}

function New-AttributeSetParams {
	param($AttributeSetName)

	@{
		id                  = $AttributeSetName
		description         = 'Attributes for IAM Team'
		maxAttributesPerSet = 25
	}
}

function New-CustomAttributeParams {
	param($AttributeSetName, $Name, $Description)

	@{
		AttributeSet            = $AttributeSetName
		Description             = $Description
		IsCollection            = $false
		IsSearchable            = $true
		Name                    = $Name
		Status                  = 'Available'
		Type                    = 'String'
		usePreDefinedValuesOnly = $false
	}
}

```


This is a PowerShell test written in Pester, a popular testing framework for PowerShell. The purpose of this test is to validate that the `New-SBCustomSecurityAttribute` function behaves correctly.

The first line of the test imports the `CreateCustomSecurityAttribute` script file (not shown here), which contains the `New-SBCustomSecurityAttribute` function being tested.

The test suite is defined by the `Describe` block. Within this block, there is a `BeforeAll` block that runs before any tests are executed. This block simply loads the script file containing the function being tested.

Next, there is a `Context` block that defines the context for the tests within it. In this case, the context is "When processing attributes". Within this context, there are two `It` blocks that define individual tests.

The first test checks that a new directory attribute set is created when it doesn't already exist. The test starts by arranging the input variables (`$attributeSetName` and `$attributes`). It then calls the function being tested with these variables using `New-SBCustomSecurityAttribute`. Finally, it asserts that the directory attribute set was actually created by attempting to retrieve it from the Microsoft Graph API and checking that it is not null or empty. If the attribute set couldn't be found, the test fails.

The second test checks that a new custom security attribute definition is created when it doesn't already exist. The test starts by arranging the input variables (`$attributeSetName` and `$attributes`) and generates a unique ID for the custom security attribute definition. It then attempts to retrieve the attribute definition from the Microsoft Graph API, using the generated ID. Finally, it asserts that the attribute definition was actually created by checking that it is not null or empty. If the attribute definition couldn't be found, the test fails.

Overall, this test validates that the `New-SBCustomSecurityAttribute` function behaves correctly under specific test cases. If any of the tests fail, it means that there is a problem with the functionality of the function being tested or the environment in which it is being run.


```powershell
# Import the CreateCustomSecurityAttribute script file



Describe "New-SBCustomSecurityAttribute" {
    BeforeAll {
        . $PSCommandPath.Replace('.Tests.ps1','.ps1')
        
    
    }
    # Define a context for the Process block
    Context "When processing attributes" {

        It "creates a new directory attribute set when it doesn't exist" {
            # TODO: Implement test logic here
            # Arrange
        
            $attributeSetName = "TestAttributeSet2"
            $attributes = @("TestAttribute1", "TestAttribute2")
            #Remove-MgDirectoryAttributeSet -AttributeSetId $attributeSetName -ErrorAction SilentlyContinue

            # Act

            New-SBCustomSecurityAttribute -AttributeSetName $attributeSetName -Attributes $attributes

            # Assert
            Connect-MgGraph -Scopes 'CustomSecAttributeDefinition.Read.All', 'CustomSecAttributeDefinition.ReadWrite.All'
            $result = Get-MgDirectoryAttributeSet -AttributeSetId $attributeSetName
            $result | Should -Not -BeNullOrEmpty -ErrorAction Stop
        }

        It "creates a new custom security attribute definition when it doesn't exist" {
            # TODO: Implement test logic here
            #Arrange

            $attributeSetName = "TestAttributeSet1"
            $attributes = @("TestAttribute1", "TestAttribute2")
        
            #Assert
            $customSecurityAttributeID = $AttributeSetName + '_' + $attributes[0]
            $customAttribute = Get-MgDirectoryCustomSecurityAttributeDefinition -CustomSecurityAttributeDefinitionId "$customSecurityAttributeID" -ErrorAction SilentlyContinue
            $customAttribute | Should -Not -BeNullOrEmpty -ErrorAction Stop	
        }

    

        
    }


}





```

Once you run the Pester Test by running  ```Invoke-Pester``` you can verify the test result.

```xml
<results>
          <test-suite type="TestFixture" name="New-SBCustomSecurityAttribute" executed="True" result="Success" success="True" time="5.8525" asserts="0" description="New-SBCustomSecurityAttribute">
            <results>
              <test-suite type="TestFixture" name="New-SBCustomSecurityAttribute.When processing attributes" executed="True" result="Success" success="True" time="5.796" asserts="0" description="New-SBCustomSecurityAttribute.When processing attributes">
                <results>
                  <test-case description="creates a new directory attribute set when it doesn't exist" name="New-SBCustomSecurityAttribute.When processing attributes.creates a new directory attribute set when it doesn't exist" time="5.5141" asserts="0" success="True" result="Success" executed="True" />
                  <test-case description="creates a new custom security attribute definition when it doesn't exist" name="New-SBCustomSecurityAttribute.When processing attributes.creates a new custom security attribute definition when it doesn't exist" time="0.2039" asserts="0" success="True" result="Success" executed="True" />
                </results>
              </test-suite>
 </results>
```
```

You can also verify the changes in Azure Portal.
![alt text](/assets/img/TestAttributeSet.png)



