---
layout: post
title: Unlocking the Mysteries of the Entra PowerShell Connector, A Beginner’s Guide
subtitle:  Unlocking the Mysteries of Entra PowerShell Connector, A Beginner's Guide to Entra ID Powershell Connector
cover-img: assets/img/download.png
thumbnail-img: assets/img/download.png
share-img: assets/img/download.png
tags: [ Powershell MA,  EntraID , Powershell , Powershell Connector, IGA, ECMA  ]

---

# Unlocking the Mysteries of the Entra PowerShell Connector, A Beginner’s Guide

## Table of Contents

1. [Common Module Script Functions](#common-module-script-functions)
   - [Functions related to schema object.](#functions-related-to-schema-object)
   - [Structure of the Schema Script:](#structure-of-the-schema-script)
2. [Import Script of Powershell Connector](#import-script-of-powershell-connector)
3. [Export script of Powershell Connector](#export-script-of-powershell-connector)
4. [Conclusion and Self-Note](#conclusion-and-self-note)
5. [Reference](#reference)

![Powershell Connector](/assets/img/download.png)


In my recent article on [Powershell Connector for Entra ID](link), I aimed to explain the PowerShell MA straightforwardly. However, I understand that colleagues and IAM Professionals with limited MIM development experience may still find it challenging to grasp. With that in mind, I attempt to simplify the understanding of the PowerShell MA for such individuals.

To simplify the understanding of the PowerShell MA, I suggest taking a simpler approach. By exploring the functions provided in the [common module script](https://github.com/microsoft/MIMPowerShellConnectors/blob/master/src/ECMA2HostCSV/Scripts/CommonModule.psm1) that accompanies the PowerShell MA, we can gain a better understanding of its functionalities. This will help us grasp the basic structure of a simple PowerShell MA and ECMAs in general.

## **Common Module Script Functions**

The functions present in the Common Module file can broadly be divided into three simple categories.

### **Functions related to schema object**

 1. `ConvertFrom-SchemaXml` This function converts a connector schema defined in an XML file to a corresponding **Microsoft.MetadirectoryServices.Schema** object. Which forms the connector space for the underlying sync engine of the **EntraID Powershell Connector.**
 1. `Get-xADSyncPSConnectorSetting` This function is used to retrieve the settings that we have defined for our connector. When we add the settings using Connector UI, values are added to the Configuration Object `Microsoft.MetadirectoryServices.ConfigParameter`collection.

### **Structure of the Schema Script**

The Schema Script is utilized to construct the Object Type schema for the ECMA2 Host. For Consistency, use the same set of attributes that we add in the EntraID for the On-Prem provisioning app's Provisioning section of the user object.

Next, create an instance of the Schema class using the `ConvertFrom-SchemaXML` function. The Schema class represents the schema for a connected directory. Alternatively, you can use the Create method of the Schema class to directly obtain an instance of the schema object.

Schema class has a property called Types which is a KeyedCollection (hashtable) of SchemaType. The next step is to create a Schema Type (different object types e.g. user, identity, group, shared mailbox, room mailbox etc. in the source target system) using the following PowerShell command.

```powershell
$schema = [Microsoft.MetadirectoryServices.Schema]::Create()

```

We can use the switch LockAnchorAttributeDefinition to lock the Anchor Attribute so that we cannot modify the anchor from the MA Properties tab.

 Schema Type Represents an object type definition within the schema.

Which consists of the following properties:

1. AnchorAttributes
2. Attributes
3. Locked
4. Name
5. PossibleDNComponentsForProvisioning.

We can use the Create method of the SchemaType class to directly obtain an instance of the schema type object.

```powershell

$schemaType = [Microsoft.MetadirectoryServices.SchemaType]::Create($t.Name,$lockAnchorDefinition)

```

The third step is to populate the schema type with Attributes based on the attributes present in the `Schema.xml` file.
Allowed Attribute Operation in Schema.XML can have the following values: 'ImportOnly', 'ExportOnly', 'ImportExport'.

```powershell
 foreach ($a in $t.Attributes.SchemaAttribute)
      {
        if ($a.IsAnchor -eq 1)
        {
          $schemaType.Attributes.Add([Microsoft.MetadirectoryServices.SchemaAttribute]::CreateAnchorAttribute($a.Name,$a.DataType,$a.AllowedAttributeOperation))
        }
        elseif ($a.IsMultiValued -eq 1)
        {
          $schemaType.Attributes.Add([Microsoft.MetadirectoryServices.SchemaAttribute]::CreateMultiValuedAttribute($a.Name,$a.DataType,$a.AllowedAttributeOperation))
        }
        else
        {
          $schemaType.Attributes.Add([Microsoft.MetadirectoryServices.SchemaAttribute]::CreateSingleValuedAttribute($a.Name,$a.DataType,$a.AllowedAttributeOperation))
        }
      }

```

Schema attributes can be of the following types 'Binary', 'Boolean', 'Integer', 'Reference' and 'String'. It can be either multi-valued or single-valued.
The Anchor switch adds the attribute as an anchor of the schema type.

After adding all the attributes to the schemaType object we can add the schemaType to the 'Types' property of the schema created in the previous step.

```powershell
       $Schema.Types.Add($SchemaType) 
       
```

### **Sample Schema**

```xml
<Schema>
  <Types>
    <SchemaType>
      <Name>Person</Name>
      <LockAnchorDefinition>1</LockAnchorDefinition>
      <Attributes>
        <SchemaAttribute>
          <Name>login</Name>
          <DataType>String</DataType>
          <IsAnchor>1</IsAnchor>
          <IsMultiValued>0</IsMultiValued>
          <AllowedAttributeOperation>ImportExport</AllowedAttributeOperation>
        </SchemaAttribute>
      </Attributes>
   </SchemaType>
</Types>
</Schema>
     
```

## **Import Script of Powershell Connector**

The Import Script acts as a bridge, fetching data from connected systems and storing it within the ECMA Connector Host's in-memory cache. Behind the scenes, this script retrieves information from various external sources like CSV files, SQL databases, or applications with REST API endpoints. Ultimately, it populates the **CSEntries** property of `Microsoft.MetadirectoryServices.
GetImportEntriesResults` object with this retrieved data.

In our previous article to build a connector for the Okta Application, I wrote a small function to import users from the Okta Instance: `Import-OktaProvisionedUsers`.

We need to first initialize the objects we are going to return after processing the import script.

```powershell
$importResults = New-Object -TypeName 'Microsoft.MetadirectoryServices.GetImportEntriesResults'

$csEntries = New-Object -TypeName 'System.Collections.Generic.List[Microsoft.MetadirectoryServices.CSEntryChange]'
```

We will need to iterate over the imported users to Populate the `CSEntries` List.

```powershell
$recordsToImport = Import-OktaProvisionedUsers
foreach ($record in $recordsToImport)
{
   $csEntry = New-xADSyncPSConnectorCSEntryChange -ObjectType $entrySchema.Name -ModificationType Add
   foreach ($column in $columnsToImport)
    {

$csEntry | Add-xADSyncPSConnectorCSAttribute -ModificationType Add -Name $columnName -Value ([Collections.IList]($record.$columnName.Split(";")))
    }

}
```

> **Note:** he function `Add-xADSyncPSConnectorCSAttribute` takes the CSEntriesChange object as a pipeline attribute and a value that is added to the CSEntries change attribute. This is also part of the `common module.psm1` script.

Once all the records are added to the $csEntries list,`ImportEntriesResults` object is exported for the underline ECMA2 Host to process.

```powershell
$importResults.CSEntries = $csEntries

$importResults.MoreToImport = $false

Write-Output $importResults
```

Values stored in the in-memory cache are used by the Entra ID Provisioning Engine to determine provisioning action on the object.

## **Export script of Powershell Connector**

When the Entra ID Provisioning service initiates a provisioning operation, the **ECMA2 Connector Host** takes over and processes the user objects. During this process, the Connector Host creates a collection of objects called CSEntries. Each CSEntry object, of type `Microsoft.MetaDirectoryServices.CSEntryChange` carries information about a specific modification type –  either adding, replacing, or deleting the user object.

The appropriate action on the target system (like Okta) depends on the modification type within the CSEntry object.

In our example of the Okta Connector, we've created three dedicated functions to handle these user modifications in Okta:

1. `Add-OktaProvisionedUser`: Creates a new user in Okta.
2. `Update-OktaUser`: Updates existing user details in Okta.
3. `Delete-OktaUser`: Removes a user from Okta.

Build Powershell Functions suitable for your use cases to meet your **Joiner-Mover-Leaver** scenarios.

Based on the objectModification Type, we can call the appropriate function to propagate changes in the External System.

```powershell

if ($entry.ObjectModificationType -eq 'Add') {
        Write-DebugLog -Message "Adding user to OKTA with Email $($baseObject.Email) and DisplayName $($baseObject.DisplayName) and UserName $($baseObject.UserName) and AzureObjectID $($baseObject.AzureObjectID)"
        Add-OktaProvisionedUser -baseObject $baseObject
      }
if ( $entry.ObjectModificationType -eq 'Delete') {
   $anchorAttributeName = $entry.AnchorAttributes[0].Name;
    $anchorAttributeValue = $entry.AnchorAttributes[0].Value.ToString();
    Write-DebugLog -Message "Delete the object with attribute '$($anchorAttributeName)' equals '$($anchorAttributeValue)'"
    Delete-OktaUser -userId $anchorAttributeValue
}
if ($entry.ObjectModificationType -eq 'Replace') {
      Update-OktaUser -attributeName $attributeName -attributeValue $entry.AttributeChanges[$attributeName].ValueChanges[0].Value -userId $anchorAttributeValue


}
```

After each action based on the response from the external system, processing results are added to the `CSEntryChangeResults` collection.

Once all the CSentries objects are processed, the Export Script creates an Object of `PutExportEntriesResults` type for the **ECMA2 Host** to process.

```powershell
$closedType = [type]"Microsoft.MetadirectoryServices.PutExportEntriesResults"

return [Activator]::CreateInstance($closedType, $csentryChangeResults)

```

## **Conclusion and Self-Note**

In this article, I aimed to break down the Entra PowerShell Connector for those, like myself, who might be new to MIM development.  By focusing on the common module script's functions, I wanted to shed light on the connector's inner workings, the general structure of the PowerShell connector, and the basic **ECMA Architecture**.

Here's what I found most helpful:

- Demystifying schema configuration with the ConvertFrom-SchemaXml function and the Schema class.
- Defining schema types with properties like anchor attributes, Attributes, and Names.
Learning how to populate schema types with attributes using the Create method of the SchemaAttribute class.
- Understanding the role of the Import Script in grabbing data and storing it within the ECMA Connector Host's cache.
- Seeing how the Export Script processes user objects based on their modification type (Add, Replace, Delete) with functions like Add-OktaProvisionedUser, Update-OktaUser, and Delete-OktaUser.

I hope This simplified understanding can act as a springboard to delve deeper into building and customizing Entra PowerShell Connectors in the future.

## Reference

- [Okta Provisioning Rest API Connector](https://github.com/SuryenduB/MIMPowerShellConnectors/blob/master/src/OKTACore/Configuration%20-%20PowershellRestAPI(Okta).xml).
