---
title: Concepts - Label metadata in the MIP SDK
description: This article will help you understand the metadata that is generated by the Microsoft Information Protection SDK.
author: tommoser
ms.service: information-protection
ms.topic: conceptual
ms.date: 11/08/2018
ms.author: tommos
---
# Microsoft Information Protection SDK - Metadata

The Microsoft Information Protection SDK generates the set of metadata that should be applied to a file. This metadata is a representation of the label. This document describes the metadata the SDK generates to apply to mail, documents, and other records.

## Labels

Labels in the Microsoft Information Protection SDK are applied to information to describe the sensitivity of that information. Label data is persisted to file or record in a set of key-value pairs that describe the label. The metadata name is built on the following structure:

`DefinedPrefix_ElementType_GlobalIdentifier_AttributeName`

When applied to data labeled with Microsoft Information Protection, the result is:

`MSIP_Label_GUID_Enabled = true`

The GUID is a unique identifier for each label in an organization.

## Microsoft Information Protection SDK Metadata

The MIP SDK applies the following set of metadata.

| Attribute | Type or Value                 | Description                                                                                                                                                                                                                                        | Mandatory |
|-----------|-------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------|
| **Enabled**   | True or False                 | This attribute indicates whether the classification represented by this set of key-value pairs is enabled for the data item. DLP products typically validate the existence of this key to identify the classification label. | Yes       |
| **SiteId**    | GUID                          | Azure Active Directory Tenant ID                                                                                                                                                                                                                   | Yes       |
| **ActionId**  | GUID                          | ActionID is changed each time a label is set. Audit logs will include both old and new actionID to allow chaining of labeling activity to the data item.                                                                                 | Yes       |
| **Method**    | Standard, Privileged, or Auto        | Set via mip::AssignmentMethod                                                                                                                                                                                                                 | No        |
| **SetDate**   | Extended ISO 8601 Date Format | The timestamp when the label was set.                                                                                                                                                                                                              | No        |
| **Name**      | string                        | Label unique name within the tenant. It doesn't necessarily correspond to display name.                                                                                                                                                              | No      |
| **ContentBits** | integer | Bitmask that describes the types of content marking that should be applied to a file. CONTENT_HEADER = 0X1, CONTENT_FOOTER = 0X2, WATERMARK = 0X4
 | No |

When applied to a file, the result is similar to the table below.

| Key                                                         | Value                                |
|-------------------------------------------------------------|--------------------------------------|
| MSIP_Label_2096f6a2-d2f7-48be-b329-b73aaa526e5d_Enabled     | true                                 |
| MSIP_Label_2096f6a2-d2f7-48be-b329-b73aaa526e5d_SetDate     | 2018-11-08T21:13:16-0800             |
| MSIP_Label_2096f6a2-d2f7-48be-b329-b73aaa526e5d_Method      | Privileged                           |
| MSIP_Label_2096f6a2-d2f7-48be-b329-b73aaa526e5d_Name        | Confidential                         |
| MSIP_Label_2096f6a2-d2f7-48be-b329-b73aaa526e5d_SiteId      | cb46c030-1825-4e81-a295-151c039dbf02 |
| MSIP_Label_2096f6a2-d2f7-48be-b329-b73aaa526e5d_ContentBits | 2                                    |
| MSIP_Label_2096f6a2-d2f7-48be-b329-b73aaa526e5d_ActionId    | 88124cf5-1340-457d-90e1-0000a9427c99 |

## Extending Metadata with Custom Attributes

Custom metadata may be appended via File and Policy API. Custom attributes must maintain the base `MSIP_Label_GUID` prefix. 

For example, an application written by Contoso Corporation must apply metadata indicating which system generated a labeled file. The application can create a new label, prefixed with `MSIP_Label_GUID`. The software vendor name and custom attribute are appended to the prefix to generate the custom metadata.

```
MSIP_Label_f048e7b8-f3aa-4857-bf32-a317f4bc3f29_ContosoCorp_GeneratedBy = HRReportingSystem
```

> [!Note]
> To maintain compatibility across common applications, the maximum length for each a key and a value is 255 characters.

## Versioning

Over time, attributes will be introduced, modified, or retired. It's expected that applications will continue to handle these old or retired attributes, as replacing the value across an enterprise may take years.

When replacing an attribute with a newer version, a version suffix should be added to the attribute:

`MSIP_Label_GUID_EnabledV2 = True | False | Condition`

## Email

Metadata applied to email maintains a key/value pair format similar to that of documents. The primary difference is that all attributes are serialized in to a single email header called **MSIP_Labels**. The key/value pairs are delimited by a semicolon and a whitespace, and placed in the new header.

Using the sample metadata above:

```
MSIP_Labels: MSIP_Label_2096f6a2-d2f7-48be-b329-b73aaa526e5d_Enabled=true; MSIP_Label_2096f6a2-d2f7-48be-b329-b73aaa526e5d_SetDate=2018-11-08T21:13:16-0800; MSIP_Label_2096f6a2-d2f7-48be-b329-b73aaa526e5d_Method=Privileged; MSIP_Label_2096f6a2-d2f7-48be-b329-b73aaa526e5d_Name=Confidential; MSIP_Label_2096f6a2-d2f7-48be-b329-b73aaa526e5d_SiteId=cb46c030-1825-4e81-a295-151c039dbf02; MSIP_Label_2096f6a2-d2f7-48be-b329-b73aaa526e5d_ContentBits=2; MSIP_Label_2096f6a2-d2f7-48be-b329-b73aaa526e5d_ActionId=88124cf5-1340-457d-90e1-0000a9427c99
```