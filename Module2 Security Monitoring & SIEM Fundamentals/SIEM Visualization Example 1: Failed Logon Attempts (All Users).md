# 📊 SIEM Visualization Example 1: Failed Logon Attempts (All Users)

> 🧠 Dashboards in SIEM solutions serve as containers for multiple visualizations, allowing us to organize and display data in a meaningful way.

In this and the following sections, we will create a dashboard and some visualizations from scratch.

---

# 🛠️ Developing Our First Dashboard & Visualization

Navigate to the bottom of this section and click on:

> 🚀 **Click here to spawn the target system!**

Now, navigate to:

Click on the **side navigation toggle**, and click on **"Dashboard"**.

Delete the existing **"SOC-Alerts"** dashboard as follows.

<img width="1337" height="559" alt="image" src="https://github.com/user-attachments/assets/86dc97cf-1cfd-4ccb-bdf6-42c5f94a088b" />

When visiting the Dashboard page again we will be presented with a message indicating that no dashboards currently exist. Additionally, there will be an option available to create a new Dashboard and its first visualization.

To initiate the creation of our first dashboard, we simply have to click on the:

> ➕ **"Create new dashboard"** button.

<img width="1913" height="1169" alt="image" src="https://github.com/user-attachments/assets/ab5930f3-fd65-496a-b878-a23ae3884ead" />

Now, to initiate the creation of our first visualization, we simply have to click on the:

<img width="1096" height="928" alt="image" src="https://github.com/user-attachments/assets/efcad83c-53de-4d24-b870-820de7dbb9cd" />

> 📊 **"Create visualization"** button.

<img width="1030" height="638" alt="image" src="https://github.com/user-attachments/assets/c0332148-373f-4f3d-a849-41a91dba31da" />

Upon initiating the creation of our first visualization, the following new window will appear with various options and settings.

---

# ⏰ Configure the Time Range

Before proceeding with any configuration, it is important for us to first click on the **calendar icon** to open the time picker.

Then, we need to specify the date range as:

```text
last 15 years
```

Finally, we can click on the **"Apply"** button to apply the specified date range to the data.

---

# 🔍 Four Things to Notice

<img width="1030" height="638" alt="image" src="https://github.com/user-attachments/assets/daca2919-4ba3-4df7-8a3b-49f69ae767ff" />

There are four things for us to notice on this window:

### 1. 🔎 Filter Option

Filter option that allows us to filter the data before creating a graph.

For example, if our goal is to display failed logon attempts, we can use a filter to only consider event IDs that match:

```text
4625 – Failed logon attempt on a Windows system
```

The following image demonstrates how we can specify such a filter.

---
<img width="1029" height="637" alt="image" src="https://github.com/user-attachments/assets/d8e61014-0538-4044-961c-762df2d70aae" />

### 2. 🗂️ Index Pattern

This field indicates the data set (index) that we are going to use.

It is common for data from various infrastructure sources to be separated into different indices, such as:

* 🌐 Network
* 🪟 Windows
* 🐧 Linux
* etc.

In this particular example, we will specify:

```text
windows*
```

in the **"Index pattern"**.

---

### 3. 🔍 Field Search

This search bar provides us with the ability to double-check the existence of a specific field within our data set, serving as another way to ensure that we are looking at the correct data.

For example, let's say we are interested in the:

```text
user.name.keyword
```

field.

We can use the search bar to quickly perform a search and verify if this field is present and discovered within our selected data set.

This allows us to confirm that we are accessing the desired field and working with accurate data.

<img width="460" height="1043" alt="image" src="https://github.com/user-attachments/assets/bad02069-c7ec-4c0b-aabd-0c80c5b53fa7" />

> ❓ **"Why `user.name.keyword` and not `user.name`?",** you may ask.

We should use the `.keyword` field when it comes to aggregations.

Please refer to this stackoverflow question for a more elaborate answer.

---

### 4. 📊 Visualization Type

Lastly, this drop-down menu enables us to select the type of visualization we want to create.

The default option displayed in the earlier image is:

```text
Bar vertical stacked
```

If we click on that button, it will reveal additional available options (image redacted as not all options fit on the screen).

From this expanded list, we can choose the desired visualization type that best suits our requirements and data presentation needs.

---

# 📋 Select the Table Visualization

<img width="1030" height="793" alt="image" src="https://github.com/user-attachments/assets/8b12e38d-4078-4aa5-8dd3-50f6097c0e7e" />

For this visualization, let's select the **"Table"** option.

After selecting the **"Table"**, we can proceed to click on the **"Rows"** option.

This will allow us to choose the specific data elements that we want to include in the table view.

Let's configure the **"Rows"** settings as follows.

<img width="728" height="935" alt="image" src="https://github.com/user-attachments/assets/b8834df3-424e-4c37-81eb-4d4505a61b58" />

> 📝 **Note:** You will notice Rank by Alphabetical and not Rank by Count of records like in the screenshot above. This is OK. By the time you perform the next configuration below, Count of records will become available.

Moving forward, let's close the **"Rows"** window and proceed to enter the **"Metrics"** configuration.

---

# 🔢 Configure Metrics

<img width="703" height="839" alt="image" src="https://github.com/user-attachments/assets/777d83a5-7733-40b6-ab8e-c7cc707aeeae" />

In the **"Metrics"** window, let's select **"count"** as the desired metric.

As soon as we select **"Count"** as the metric, we will observe that the table gets populated with data (assuming that there are events present in the selected data set).

---

# 💻 Add the Hostname

<img width="1029" height="504" alt="image" src="https://github.com/user-attachments/assets/3c8cab0c-d95f-4ef3-8325-6ddc1b7d0bae" />

One final addition to the table is to include another **"Rows"** setting to show the machine where the failed logon attempt occurred.

To do this, we will select the:

```text
host.hostname.keyword
```

field, which represents the computer reporting the failed logon attempt.

This will allow us to display the hostname or machine name alongside the count of failed logon attempts, as shown in the image.

---

# 📊 Table Information

Now we can see three columns in the table, which contain the following information:

| 📌 Information | 📝 Description                                                                                                                                                                                 |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 👤 Username    | The username of the individuals logging in (Note: It currently displays both users and computers. Ideally, a filter should be implemented to exclude computer devices and only display users). |
| 💻 Machine     | The machine on which the logon attempt occurred.                                                                                                                                               |
| 🔢 Count       | The number of times the event has occurred (based on the specified time frame or the entire data set, depending on the settings).                                                              |

Finally, click on **"Save and return"**, and you will observe that the new visualization is added to the dashboard, appearing as shown in the following image.


<img width="1030" height="761" alt="image" src="https://github.com/user-attachments/assets/0662e735-0c11-4d79-9cae-862c873e0268" />

Let's not forget to save the dashboard as well.

We can do so by simply clicking on the **"Save"** button.

---

# 🛠️ Refining The Visualization

Suppose the SOC Manager suggested the following refinements:

* 🏷️ Clearer column names should be specified in the visualization
* 🔐 The Logon Type should be included in the visualization
* ↕️ The results in the visualization should be sorted
* 🚫 The `DESKTOP-DPOESND`, `WIN-OK9BH1BCKSD`, and `WIN-RMMGJA7T9TC` usernames should not be monitored
* 🖥️ Computer accounts should not be monitored (not a good practice)

Let's refine the visualization we created, so that it fulfills the suggestions above.

---

# 📊 Edit the Dashboard

Click on the **side navigation toggle**, and click on **"Dashboard"**.

The dashboard we previously created should be visible.

Let's click on the **"pencil"/edit icon**.

<img width="1888" height="652" alt="image" src="https://github.com/user-attachments/assets/95448f21-66db-4930-adce-3720e96a75a4" />

Let's now click on the **"gear"** button at the upper-right corner of our visualization, and then click on **"Edit lens"**.

---

# 🏷️ Change Column Names

<img width="1913" height="985" alt="image" src="https://github.com/user-attachments/assets/93d77fd3-3b2d-468d-aaeb-9037d1bf6856" />

**"Top values of user.name.keyword"** should be changed as follows.

<img width="433" height="873" alt="image" src="https://github.com/user-attachments/assets/8fddb57a-2d1b-4df4-b086-c0c8c7d967ef" />

<img width="476" height="801" alt="image" src="https://github.com/user-attachments/assets/407ec331-ef28-4d94-99b3-9b35b2a1f1fb" />


**"Top values of host.hostname.keyword"** should be changed as follows.

<img width="473" height="789" alt="image" src="https://github.com/user-attachments/assets/970e74dc-163a-4d4d-9cc9-5505747d93c6" />


---

# 🔐 Add Logon Type

The **"Logon Type"** can be added as follows (we will use the:

```text
winlog.logon.type.keyword
```

field).

---

<img width="432" height="842" alt="image" src="https://github.com/user-attachments/assets/6daacf73-d563-4b21-a3d4-471e6d9af694" />

<img width="456" height="791" alt="image" src="https://github.com/user-attachments/assets/a2042525-50a5-4f67-92d1-eecab8acf39d" />


# 🔢 Change Count of Records

**"Count of records"** should be changed as follows.

<img width="459" height="1062" alt="image" src="https://github.com/user-attachments/assets/eb0ede89-3053-42ee-8ec8-b42621924724" />

---

# ↕️ Introduce Result Sorting

We can introduce result sorting as follows.

<img width="1915" height="1196" alt="image" src="https://github.com/user-attachments/assets/3ea3c131-0c8c-4189-bef9-b3453e0bdb5f" />

All we have to do now is click on **"Save and return"**.

---

# 🚫 Exclude Specific Usernames

The `DESKTOP-DPOESND`, `WIN-OK9BH1BCKSD`, and `WIN-RMMGJA7T9TC` usernames can be excluded by specifying additional filters as follows.

---

# 🖥️ Exclude Computer Accounts

<img width="1430" height="1046" alt="image" src="https://github.com/user-attachments/assets/378d7b22-f55a-4042-a773-5ac60de4cb70" />

Computer accounts can be excluded by specifying the following KQL query and clicking on the **"Update"** button.

```kql
NOT user.name: *$ AND winlog.channel.keyword: Security
```

The:

```text
AND winlog.channel.keyword: Security
```

part is to ensure that no unrelated logs are accounted for.

---

<img width="1917" height="1053" alt="image" src="https://github.com/user-attachments/assets/0945a499-c18a-420f-ad87-4c8d6271e094" />

# 📊 Visualization After Refinements

This is our visualization after all the refinements we performed.

```text
┌──────────────────────────────────────────────────────────────┐
│              Failed Logon Attempts Visualization             │
├──────────────┬──────────────┬──────────────┬─────────────────┤
│ Username     │ Hostname     │ Logon Type   │ Count of Records│
├──────────────┼──────────────┼──────────────┼─────────────────┤
│ ...          │ ...          │ ...          │ ...             │
└──────────────┴──────────────┴──────────────┴─────────────────┘
```

<img width="1917" height="1071" alt="image" src="https://github.com/user-attachments/assets/12c04abe-b491-4463-8374-ab242294990f" />

---

# 🏷️ Give the Visualization a Title

Finally, let's give our visualization a title by clicking on **"No Title"**.

<img width="1917" height="1059" alt="image" src="https://github.com/user-attachments/assets/2e812352-3850-423f-92c7-c92c0a6fa2d2" />

Don't forget to click on the **"Save"** button (the one on the upper-right hand side of the window).

---

# ✅ Final Workflow

```text
🚀 Spawn Target System
        │
        ▼
🌐 Open Dashboard
        │
        ▼
➕ Create New Dashboard
        │
        ▼
📊 Create Visualization
        │
        ▼
⏰ Set Time Range
        │
        ▼
🔎 Configure Filter
        │
        ▼
🗂️ Select windows*
        │
        ▼
📋 Select Table
        │
        ▼
👤 Configure Username
        │
        ▼
🔢 Configure Count
        │
        ▼
💻 Add Hostname
        │
        ▼
💾 Save Visualization
        │
        ▼
🛠️ Refine Visualization
        │
        ├── 🏷️ Rename Columns
        ├── 🔐 Add Logon Type
        ├── ↕️ Sort Results
        ├── 🚫 Exclude Usernames
        └── 🖥️ Exclude Computer Accounts
        │
        ▼
🏷️ Give Visualization a Title
        │
        ▼
💾 Save Dashboard
```
