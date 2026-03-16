# Automate Emailing File Attachments from Microsoft Forms using Power Automate

This guide provides a comprehensive, step-by-step walkthrough for creating a Power Automate flow that automates a common business process: when a user submits a **personal Microsoft Form** with a file upload, the flow retrieves the file from **OneDrive for Business** and sends it as an email attachment via **Outlook**.

This tutorial includes a crucial workaround for a common issue where Power Automate requires a JSON schema before the flow can be tested to generate that schema.

## Prerequisites

- A Microsoft 365 subscription with access to Power Automate, Microsoft Forms, OneDrive for Business, and Outlook.
- A **personal** Microsoft Form (not a Group Form).
- The form must contain a **File Upload** question.

---

## The Guide

The process is divided into two main parts. First, we will perform a temporary setup to capture the data structure of the file upload. Second, we will use that data structure to build the final, permanent flow.

### Part 1: Initial Setup & The Data Sampling Workaround

#### Step 1: Create the Trigger
1.  In Power Automate, create a new **Automated cloud flow**.
2.  For the trigger, search for and select **`Microsoft Forms`**.
3.  Choose the action **`When a new response is submitted`**.
4.  In the **Form Id** field, select your form from the dropdown list.

#### Step 2: Get Response Details
1.  Click **"+ New step"**.
2.  **Connector:** `Microsoft Forms`
3.  **Action:** `Get response details`
4.  **Form Id:** Select the same form again.
5.  **Response Id:** From the dynamic content window, select **`Response Id`** from the trigger step.

#### Step 3: The Workaround - Use `Compose` to Capture a Sample
This temporary step is essential because we need to see what the file data looks like before we can properly parse it in a later step.
1.  Click **"+ New step"**.
2.  **Connector:** `Data Operation`
3.  **Action:** `Compose`
4.  **Inputs:** In the dynamic content window, under the "Get response details" header, select the question corresponding to your **file upload**.

#### Step 4: Perform a Test Run to Capture the Sample Data
1.  Click **"Save"** in the top-right corner.
2.  After saving, click **"Test"**. Select **"Manually"** and click the **"Test"** button.
3.  In a new browser tab, open and submit your Microsoft Form. **You must upload a file**.
4.  Return to the Power Automate tab. The test run should complete successfully.
5.  Open the run history for that successful test and expand the **"Compose"** action.
6.  In the **"Outputs"** section, you will see a text string. **Copy this entire text string.** It will look similar to the example below:

    ```json
    [{"name":"FileName_EmployeeName.xlsx","link":"https://XXXXX","id":"XXXXX","type":XXXXX,"size":XXXXX,"referenceId":"XXXXX","driveId":"XXXXX",....}]
    ```

---

**If it does not work, ask an AI bot to transform the data into JSON format. Remove any sensitive data or information first.** 

````markdown
## Power Automate Parse JSON Schema EXAMPLE

```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "name": {
        "type": "string"
      },
      "link": {
        "type": "string"
      },
      "id": {
        "type": "string"
      },
      "type": {
        "type": ["string", "null"]
      },
      "size": {
        "type": "integer"
      },
      "referenceId": {
        "type": "string"
      },
      "driveId": {
        "type": "string"
      },
      "status": {
        "type": "integer"
      },
      "uploadSessionUrl": {
        "type": ["string", "null"]
      },
      "badgerToken": {
        "type": ["string", "null"]
      }
    },
    "required": [
      "name",
      "link",
      "id",
      "size",
      "referenceId",
      "driveId",
      "status"
    ]
  }
}
```
````


---

### Part 2: Building the Core Flow Logic

Now that we have the data sample, we can build the final flow.

#### Step 5: Parse the File Data with `Parse JSON`
1.  Return to the flow **editor** (click the "Edit" button).
2.  **Delete** the temporary **"Compose"** action.
3.  In its place, click **"+ New step"**.
4.  **Connector:** `Data Operation`
5.  **Action:** `Parse JSON`
6.  **Content:** From the dynamic content, again select the **file upload question** from the "Get response details" step.
7.  **Schema:** Click the **"Generate from sample"** button. Paste the text you copied in Step 4 into the textbox and click **"Done"**. Power Automate will auto-generate the required schema.

#### Step 6: Loop Through Files with `Apply to each`
Forms provides file information as a list (an array), so a loop is required even if you only allow one file.
1.  Click **"+ New step"**.
2.  **Connector:** `Control`
3.  **Action:** `Apply to each`
4.  **Select an output from previous steps:** From the dynamic content, under the "Parse JSON" header, select **`body`**.

*All subsequent actions must be added **inside** the "Apply to each" container.*

#### Step 7: Get the File Content from OneDrive
1.  Inside the loop, click **"Add an action"**.
2.  **Connector:** `OneDrive for Business`
3.  **Action:** `Get file content`
4.  **File:** From the dynamic content, under the "Parse JSON" header, select **`id`**. This is the unique file identifier.

#### Step 8: Send the Email with the Attachment
1.  Inside the loop, after "Get file content", click **"Add an action"**.
2.  **Connector:** `Outlook`
3.  **Action:** `Send an email (V2)`
4.  Fill in the primary fields:
    *   **To:** Enter a static email address or select **`Responders' Email`** from the "Get response details" step.
    *   **Subject:** Enter your desired subject line.
    *   **Body:** Write the body of the email.
5.  Click **"Show advanced options"** to reveal the attachment fields.
6.  **Attachments Name - 1:** From the dynamic content, under the "Parse JSON" header, select **`name`**.
7.  **Attachments Content - 1:** From the dynamic content, under the "Get file content" header, select **`File Content`**.

---

## Final Flow Overview

Your completed flow will have the following logical structure:

1.  **When a new response is submitted**
2.  **Get response details**
3.  **Parse JSON**
4.  **Apply to each** (Input: `body` from *Parse JSON*)
    *   └───> **Get file content (OneDrive for Business)**
    *   └───> **Send an email (V2)**

You have now successfully automated the process. Save and test the flow to ensure it works as expected.
Developed by Bekaiym Egemkulova.
