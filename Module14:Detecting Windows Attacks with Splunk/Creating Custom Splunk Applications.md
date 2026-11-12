<img width="1262" height="414" alt="image" src="https://github.com/user-attachments/assets/79a0428e-1bfe-44de-85d9-5dd1138757a3" /># Creating Custom Splunk Applications

## How To Create A Custom Splunk Application

### 1. Access Splunk Web

Open your web browser and navigate to Splunk Web.

### 2. Go to Manage Apps

From the menu bar at the top, select **Apps** and then choose **Manage Apps**.

<img width="333" height="245" alt="image" src="https://github.com/user-attachments/assets/134ac9e8-5f0d-4fdf-b71b-e639b4b9707f" />

### 3. Create a New App

On the Apps page, click on **Create app**.

### 4. Enter App Details

On the Add new page, complete the properties for your new app:

- **Name:** Enter the name for your app, for example, `<Your app name>`.
- **Folder name:** Specify the folder name, which should be similar to `<App_name>`. This will correspond to the app's directory under `$SPLUNK_HOME/etc/apps/`.
- **Version:** Input `"1.0.0"`.
- **Description:** Provide a description for your app, for instance, `<App description>`.
- **Template:** Choose **barebones** from the drop-down menu.

<img width="939" height="652" alt="image" src="https://github.com/user-attachments/assets/2c33f589-3c06-426d-a50a-54c0017c42b3" />

### 5. Save the App

Click on **Save**. You can verify that your app has been created by going to the Apps menu. Your new app should now be listed there. Also, if you navigate to the Splunk Web home page, you'll find your app listed under the Apps list as **Academy hackthebox - Detection of Active Directory Attacks**.

<img width="348" height="269" alt="image" src="https://github.com/user-attachments/assets/80e9f4d4-d932-474e-b805-df8eda30b5f7" />

<img width="273" height="229" alt="image" src="https://github.com/user-attachments/assets/c2892aff-eb8a-41da-aa28-438b848374f9" />

### 6. Explore the Directory Structure

Use a file browser to navigate to `$SPLUNK_HOME/etc/apps`. Here you'll find your app directory, which includes the following folders:

- **`/bin`:** This is where scripts are stored.
- **`/default`:** This directory holds files for configuration, views, dashboards, and app navigation.
- **`/local`:** This directory contains user-modified versions of files for configuration, views, dashboards, and app navigation.
- **`/metadata`:** This directory holds permissions files.

<img width="1110" height="198" alt="image" src="https://github.com/user-attachments/assets/ad29f9df-3941-424c-a748-0ec82faaf206" />

### 7. View the Navigation File

The navigation configuration file is an XML file. Using a text editor, open:

```text
$SPLUNK_HOME/etc/apps/<your app>/default/data/ui/nav/default.xml
```

Here you'll find the default navigation definition for an app:

<img width="714" height="233" alt="image" src="https://github.com/user-attachments/assets/dc03f63d-09f9-4ccd-8005-0407bbdf44d6" />

In this XML, the top-level `nav` tag acts as the parent. The `search_view` attribute designates the default view for searches. In this case, the search view is employed, which is inherited from the Search & Reporting app. The next level in the XML hierarchy corresponds to items displayed on the app bar. The list of `view` tags denotes different views to show. Each of the views corresponds to a view from the Search & Reporting app. The attribute `default='true'` indicates the view to use as the app home page – here, the search view serves as the home page.

### 8. Create Your First Dashboard

Go to dashboards and click on **Create New Dashboard**. Enter the dashboard name, provide a description if necessary, set permissions, and select **Classic Dashboards**.

<img width="1256" height="530" alt="image" src="https://github.com/user-attachments/assets/acac9716-eacc-46f1-b744-3b002eebf3dd" />

<img width="541" height="521" alt="image" src="https://github.com/user-attachments/assets/44699ada-7fd5-40d4-ae95-3cfc200e70e4" />

### 9. Configure the Dashboard

You'll now see the dashboard editor page, where you can configure panels, inputs, etc., to facilitate your monitoring process. Add time input for the dashboard and adjust the default time range to suit your needs. Next, add a statistical table panel, select a time range for the Shared Time Picker, add the Content Title (e.g., `"<Panel name>"`), and input the Search String. To use input in searches, enclose the input token with dollar signs, like `$user$`. Click **Add to Dashboard** when ready. Save your changes.

<img width="1262" height="414" alt="image" src="https://github.com/user-attachments/assets/d2ac78a3-19f8-453a-9012-8fa3ebb23b05" />

<img width="517" height="379" alt="image" src="https://github.com/user-attachments/assets/573826e9-f050-43f4-95e0-f136a5b64b34" />

<img width="741" height="391" alt="image" src="https://github.com/user-attachments/assets/c0a26233-cf2e-48c1-ae26-0706cc373aa7" />

<img width="1249" height="847" alt="image" src="https://github.com/user-attachments/assets/e431d2f9-eae1-4555-b1e4-d3807fa742fc" />

### 10. Dashboard Storage

All dashboards you've created in your app are stored at:

```text
"<AppPath>/local/data/ui/views/dashboard_title.xml"
```

To add your dashboard to the navigation bar, simply append the dashboard title to the navigation default page XML:

```text
"<AppPath>/local/data/ui/nav/default.xml"
```

<img width="406" height="191" alt="image" src="https://github.com/user-attachments/assets/bb7b4b90-430e-4336-9538-25d1b5ce7bb3" />

### 11. Restart Splunk

Reboot your Splunk instance. Once restarted, you should see your dashboard in the navigation bar.

<img width="1253" height="242" alt="image" src="https://github.com/user-attachments/assets/d60e5439-dce4-4f91-b80a-a545f44c520c" />

### 12. Grouping Dashboards

If you wish to group multiple dashboards under a single entry in the navigation bar, use the `collection` tag.

<img width="405" height="103" alt="image" src="https://github.com/user-attachments/assets/948b2c03-e376-4347-b4da-13e74aefb688" />

## Updating & Exploring The "Academy hackthebox - Detection of Active Directory Attacks" Splunk Application

`Detection-of-Active-Directory-Attacks.tar.gz.tar` can be downloaded from the Resources section of this module (upper right corner) and used to update the existing **Academy hackthebox - Detection of Active Directory Attacks** Splunk Application by clicking **Apps -> Manage Apps -> Install app from file -> Browse -> ✓ Upgrade app**. Checking this will overwrite the app if it already exists. -> **Upload**.

Now, take some time to explore this custom Splunk application and see how it can significantly improve our monitoring capabilities.

## 🔑 Key Takeaway

Creating a custom Splunk application involves setting up the app, exploring its directory structure and navigation configuration, creating and configuring dashboards, and adding dashboards to the navigation bar. The existing Academy application can also be updated by installing the provided app archive through Splunk's Manage Apps interface.
