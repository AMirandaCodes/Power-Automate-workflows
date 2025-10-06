# Automatic Reminders – Power Automate + SharePoint

## Overview
This Power Automate workflow automatically sends **email reminders** for documents, certifications, or inspections stored in a **SharePoint list**, based on their individual due dates.  

Each SharePoint record contains:
- The **due date**
- The **responsible person (Representative)**
- An optional **CC recipient**
- Configurable columns for **days before first and second reminders**

The flow runs daily at **9:00 AM**, checks which items are approaching expiry, and emails the relevant users accordingly.

## Key Features
- Daily automatic trigger at a specified time.  
- Fully configurable reminder intervals per record (two reminders per item).  
- Dynamic email recipients and CCs based on SharePoint data.  
- No-code configurability: all parameters (recipients, reminder days, due dates) can be edited directly in the SharePoint list.  
- Email subject and body dynamically include item title and due date.  

---

## Flow Structure

### 1. **Recurrence**
- **Trigger:** `Recurrence`
- **Settings:**  
  - Interval: `1`  
  - Frequency: `Day`  
  - Start time: `09:00` (UTC or local time as preferred)
  
  *Ensures the workflow runs every morning at 9 AM.*

### 2. **Get items**
- **Site Address**: *Your SharePoint site*
- **List Name**: *Your SharePoint list*
- **Filter Query:** `field_4 ne null`

### 2. **Apply to Each**
- **Input:** `body/value` from the SharePoint “Get items” action (not shown here but assumed to be before this step).  

*Iterates through every item in the SharePoint list.*

### 3. **Compose – Extract Fields**
Extracts relevant SharePoint columns for easier reference in later steps:
- **Due date:** `field_4`  
- **Representative:** `Representative` (Person column)  
- **CC (optional):** `Cc Email`  
- **Reminder days:** `DaysbeforeFirstReminder`, `DaysbeforeSecondReminder`

### 4. **Compose – Calculate Reminder Date (1)**
- **Action name:** `Calculate Reminder Date`  
- **Expression:**
  ```text
  formatDateTime(
      subtractFromTime(
          item()?['field_4'],
          int(item()?['DaysbeforeFirstReminder']),
          'Day'
      ),
      'yyyy-MM-dd'
  )

*Calculates the date when the first reminder should be sent.*

### 5. **Compose – Calculate Reminder Date (2)**
- **Action name:** `Calculate Reminder Date 2`  
- **Expression:**
  ```text
  formatDateTime(
    subtractFromTime(
        item()?['field_4'],
        int(item()?['DaysbeforeSecondReminder']),
        'Day'
    ),
    'yyyy-MM-dd'
  )
  ```
*Calculates the date when the second reminder should be sent.*

### 6. **Condition – Check Reminder Dates**
Evaluates if today matches either of the reminder dates:
  ```text
  Outputs('Calculate_Reminder_Date') is equal to utcNow('yyyy-MM-dd')
  OR
  Outputs('Calculate_Reminder_Date_2') is equal to utcNow('yyyy-MM-dd')
  ```
- If FALSE: No action (skip to next item).
- If TRUE: Proceed to send the reminder email.

### 7. **Send Email (V2) – Outlook**
When the condition is met, the flow sends a customized email:
| Field        | Value                                                                                             |
| ------------ | ------------------------------------------------------------------------------------------------- |
| **To:**      | `[Representative]`                                                               |
| **Subject:** | `Reminder: [Title] is due on [Due Date]`                                                          |
| **Body:**    | `Hi [Representative], the [Title] is due on [Due Date].`                                          |
| **CC:**      | `if(empty(items('Apply_to_each')?['Cc']?['Email']), '', items('Apply_to_each')?['Cc']?['Email'])` |

#### Example SharePoint List Structure
| Title                   | Due Date   | Representative | Cc                                                  | DaysbeforeFirstReminder | DaysbeforeSecondReminder |
| ----------------------- | ---------- | -------------- | --------------------------------------------------- | ----------------------- | ------------------------ |
| Fire Safety Certificate | 2025-10-30 | John Doe       | [jane.doe@example.com](mailto:jane.doe@example.com) | 90                      | 30                       |
| Equipment Inspection    | 2025-11-15 | Alice Smith    |                                                     | 60                      | 15                       |

#### Example Email Output
**Subject**
> Reminder: Fire Safety Certificate is due on 30/10/2026

**Body:**
> Hi John Doe,
> 
> The Fire Safety Certificate is due on 30/10/2025.
>
> Kind regards,
> 
> Automated Reminder System

#### Notes
- All SharePoint column names and variables can be customised — update the expressions accordingly.
- Ensure the date formats (yyyy-MM-dd) match your region’s configuration.
- Connection references (SharePoint, Outlook) will need to be reconfigured upon import.
