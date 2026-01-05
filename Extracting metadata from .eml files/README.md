# Extracting Metadata from .eml Files (Power Automate + SharePoint)

## Overview
This Power Automate workflow automatically extracts **sender metadata** from `.eml` email files uploaded to a **SharePoint document library** and populates a SharePoint metadata column (e.g. **From**) with the sender’s name or email.

The flow is triggered every time a new file is uploaded and processes only `.eml` files.  
It reads the raw email content, parses the **From** header, cleans the extracted value, and updates the SharePoint file properties accordingly.

This ensures that uploaded email files are **searchable, filterable, and consistently tagged** without any manual intervention.

## Key Features
- Automatic trigger on file upload to SharePoint.  
- Conditional processing limited to `.eml` files only.  
- Extraction and parsing of raw email headers.  
- Cleans and normalizes sender information.  
- Updates SharePoint metadata dynamically.  
- Fully automated with no user input required.

---

## Flow Structure
<img width="522" height="860" alt="Flow Extraction metadata From eml files" src="https://github.com/user-attachments/assets/36bc69a5-d147-4735-a42c-90246b1ca228" />

### 1. **When a file is created (properties only)**
- **Trigger:** SharePoint
- **Site Address:** *Your SharePoint site*
- **Library Name:** *Your document library*

*Starts the workflow whenever a new file is uploaded to the library.*

### 2. **Initialize Variable – Sender Name**
- **Name:** `vSenderName`
- **Type:** `String`

*Stores the final cleaned sender name or email before updating SharePoint.*

### 3. **Condition – Is it an .eml file**
Checks whether the uploaded file is an email file:

```text
triggerBody()?['{FilenameWithExtension}'] ends with '.eml'
```

- **If FALSE**: No action (the flow exits).
- **If TRUE**: Continue processing the file.

### 4. **Get File Content (SharePoint)**
- **Site Address**: *Your SharePoint site*
- **File Identifier:**

```text
triggerBody()?['{Identifier}']
```
*Retrieves the raw content of the uploaded .eml file.*

### 5. **Compose – Convert EML Content to Text**
- **Action name**: `EMLText`
- **Expression**:
```text
base64ToString(outputs('Get_file_content')?['body']['$content'])
```
*Converts the Base64-encoded email file into readable text.*

### 6. **Compose – Extract From Line**
- **Action name**: `FromLine`
- **Expression**:

```text
first(
  split(
    first(
      skip(
        split(outputs('EMLText'), 'From: '),
        1
      )
    ),
    decodeUriComponent('%0A')
  )
)
```

*Extracts the first line following the `From`: header in the email.*

### 7. **Compose – Trim Quotes**
- **Action name**: `FromTrim`
- **Expression**:

```text
trim(replace(outputs('FromLine'), '"', ''))
```

*Removes tabs, extra spaces, and normalizes the sender string.*

### 9. **Condition – Does Sender Contain Angle Brackets**
Checks if the sender value includes an email address in angle brackets:
```text
outputs('FromClean') contains '<'
```

**If TRUE**:
- Compose – Extract Name (Angle Brackets)
```text
trim(first(split(outputs('FromClean'), '<')))
```
- Set variable – vSenderName
  - Value: extracted name

**If FALSE**:
- Set variable – vSenderName
  - Value: `outputs('FromClean')`
 
### 10. **Update File Properties (SharePoint)**
- **Site Address**: *Your SharePoint site*
- **Library Name**: *Your document library*
- **ID**: SharePoint file ID
- **Advanced Parameters**:
  - From:
  ```text
  variables('vSenderName')
  ```

*Writes the extracted sender name into the SharePoint metadata column.*

## Example Result
- **Uploaded File**: `Email_Submission.eml`
- **Extracted Metadata**:

| Column         | Value                                                                                             |
| --- | --- |
| **From:**      | `John Smith`                                                                                      |

### Notes
- This flow only processes `.eml` files — other file types are ignored.
- Column names and variables can be customized to match your SharePoint setup.
- Ensure the SharePoint library has a From (or equivalent) column available.
- Connection references must be reconfigured when importing the flow.
