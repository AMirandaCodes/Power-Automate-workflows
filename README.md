# Power Automate Workflows

## Overview
This repository contains a collection of **Microsoft Power Automate workflows** (flows) I developed to automate and improve various processes in a manufacturing and engineering environment.  
Each workflow was created using Power Automate’s **visual interface**, with custom expressions and small code snippets where necessary.

## Workflows

### 1. Automatic Reminders
**Goal:** Automate email reminders for documents and certifications approaching expiry.

**Description:**
I was tasked with finding a system that would automatically email users when certain documents or inspections were close to expiring.  
To accomplish this, I:
- Created a **SharePoint list** containing all items with due dates and configurable reminder intervals.
- Designed a **Power Automate flow** that triggers daily at 9 AM.
- Added logic to check whether today matches the reminder windows (e.g., 90 or 30 days before the due date).
- Sent automatic emails to responsible users, with the option to CC other recipients.

**Key features:**
- Customizable reminder intervals (two separate reminders per document).  
- Dynamic recipients and CCs based on SharePoint columns.  
- Fully automated and configurable without modifying the flow.
  
### 2. Extracting Metadata from .eml Files (SharePoint)
**Goal:** Automatically populate the “From” column in SharePoint when new `.eml` files are uploaded.

**Description:**
This flow triggers whenever a new `.eml` file is uploaded to a specific SharePoint library. It extracts sender information (name or email) and updates the “From” column automatically.

**Key features:**
- Triggered on file upload.  
- Parses `.eml` metadata automatically.  
- Maintains consistent and searchable metadata in SharePoint.


### 3. Extracting Metadata from .msg Files (SharePoint)
**Goal:** Extend metadata extraction capabilities to `.msg` files.

**Description:**
Similar to the `.eml` version, this flow processes `.msg` files uploaded to SharePoint. Since Power Automate cannot natively read `.msg` metadata, this version integrates the **Encodian** connector to extract sender data before updating SharePoint columns.

**Key features:**
- Triggered on file upload.  
- Integrates with the **Encodian** connector for `.msg` parsing.  
- Consistent output and metadata structure across `.eml` and `.msg` formats.

## Notes
- Power Automate workflows are visual-based; exporting them produces `.zip` or `.json` files.  
- These exports can be imported back into any Power Automate environment via “Import a flow”.  
- All sensitive data (emails, connection references, etc.) has been anonymised before publishing.
