<img width="1030" height="638" alt="image" src="https://github.com/user-attachments/assets/5f91f181-a01e-4f36-bfd2-0b29a00eb78a" /># 📊 SIEM Visualization Example 3: Successful RDP Logon Related To Service Accounts

## 📌 SIEM Visualization Example 3: Successful RDP Logon Related To Service Accounts

In this SIEM visualization example, we aim to create a visualization to monitor successful RDP logons specifically related to service accounts. Service account credentials are never used for RDP logons in corporate/real-world environments. We have been informed by the IT Operations department that all service accounts on the environment start with `svc-`.

The motivation for this visualization stems from the fact that service accounts often possess exceptionally high privileges. We need to keep a close eye on how service accounts are used.

Our visualization will be based on the following Windows event log.

```text
4624: An account was successfully logged on
```

---

## 🚀 Spawn the Target System

Navigate to the bottom of this section and click on **Click here to spawn the target system!**.

---

## 📊 Open the Dashboard

Click on the **side navigation toggle**, and click on **"Dashboard"**.

A prebaked dashboard should be visible. Let's click on the **"pencil"/edit icon**.

<img width="1888" height="652" alt="image" src="https://github.com/user-attachments/assets/26587ea4-bf96-4d0c-8477-f20df43b86b3" />



---

## 📈 Create the Visualization

Now, to initiate the creation of our first visualization, we simply have to click on the **"Create visualization"** button.

Upon initiating the creation of our first visualization, the following new window will appear with various options and settings.

<img width="1030" height="638" alt="image" src="https://github.com/user-attachments/assets/56c347db-dafd-44a3-b842-2297ef51abaa" />



---

## 🔍 Four Things to Notice

There are four things for us to notice on this window:

### 1. 🔎 Filter Option

A filter option that allows us to filter the data before creating a graph.

In this case our goal is to display successful RDP logons specifically related to service accounts.

We can use a filter to only consider event IDs that match `4624` – An account was successfully logged on.

In this case though, we should also take into account the logon type which should be **RemoteInteractive** (`winlog.logon.type` field).

The following images demonstrates how we can specify such filters.

<img width="1427" height="802" alt="image" src="https://github.com/user-attachments/assets/9875175b-674a-4928-8324-018b166b956a" />

<img width="1426" height="809" alt="image" src="https://github.com/user-attachments/assets/b4893e7f-b17b-41a5-875a-010cadc4cff9" />


---

### 2. 🗂️ Index Pattern

This field indicates the data set (index) that we are going to use.

It is common for data from various infrastructure sources to be separated into different indices, such as network, Windows, Linux, etc.

In this particular example, we will specify `windows*` in the **"Index pattern"**.

---

### 3. 🔍 Field Search

This search bar provides us with the ability to double-check the existence of a specific field within our data set, serving as another way to ensure that we are looking at the correct data.

We are interested in the `user.name.keyword` field.

We can use the search bar to quickly perform a search and verify if this field is present and discovered within our selected data set.

This allows us to confirm that we are accessing the desired field and working with accurate data.

<img width="460" height="1043" alt="image" src="https://github.com/user-attachments/assets/a1e7d41b-2bfb-471b-bd83-4e04d9c27469" />


---

### 4. 📊 Visualization Type

Lastly, this drop-down menu enables us to select the type of visualization we want to create.

The default option displayed in the earlier image is **"Bar vertical stacked"**.

If we click on that button, it will reveal additional available options (image redacted as not all options fit on the screen).

From this expanded list, we can choose the desired visualization type that best suits our requirements and data presentation needs.

<img width="1030" height="793" alt="image" src="https://github.com/user-attachments/assets/a2b2c74c-972b-4725-be3c-9c2379691ae3" />


---

## 📋 Select the Table

For this visualization, let's select the **"Table"** option.

After selecting the **"Table"**, we can proceed to click on the **"Rows"** option.

This will allow us to choose the specific data elements that we want to include in the table view.

<img width="711" height="712" alt="image" src="https://github.com/user-attachments/assets/b605e1c2-e5a1-488b-9b9b-7f12e8463ff3" />


Let's configure the **"Rows"** settings as follows.

<img width="728" height="935" alt="image" src="https://github.com/user-attachments/assets/0d1345c8-1bbe-4363-b909-9cb46e5ff29f" />


---

## 🔢 Configure Metrics

Moving forward, let's close the **"Rows"** window and proceed to enter the **"Metrics"** configuration.

<img width="606" height="554" alt="image" src="https://github.com/user-attachments/assets/2cf108e6-24e1-450d-bb32-ef6dc15d5beb" />


In the **"Metrics"** window, let's select **"count"** as the desired metric.

<img width="703" height="839" alt="image" src="https://github.com/user-attachments/assets/d245349e-f1b5-4f4d-b066-fa9992fba407" />


---

## 💻 Add the Involved Machines

One final addition to the table is to include two more **"Rows"** settings to show the machine where the successful RDP logon attempt occurred and the machine that initiated the successful RDP logon attempt.

To do this, we will select the `host.hostname.keyword` field that represents the computer reporting the successful RDP logon attempt and the `related.ip.keyword` field that represents the IP of the computer initiating the succsessful RDP logon attempt.

This will allow us to display the involved machines alongside the count of successful logon attempts, as shown in the image.

<img width="458" height="875" alt="image" src="https://github.com/user-attachments/assets/707f565f-9ee2-4060-bea2-6623143e1784" />


<img width="458" height="821" alt="image" src="https://github.com/user-attachments/assets/fdfeb705-1607-45c8-b61c-15152da7e38e" />


---

## 🔐 Specify the Service Account Filter

As discussed, we want to monitor successful RDP logons specifically related to service accounts, knowing for a fact that all service accounts of the environment start with `svc-`.

So, to conclude our visualization we need to specify the following KQL query.

```kql
user.name: svc-*
```

> 📝 **Note:** As you can see we don't use the `.keyword` field in KQL queries.

<img width="1800" height="1218" alt="image" src="https://github.com/user-attachments/assets/df7051b0-dc2e-441d-9e93-3fa226d5cf73" />


---

## 📊 Table Columns

Now we can see four columns in the table, which contain the following information:

1. 👤 The service account whose credentials generated the successful RDP logon attempt event.
2. 💻 The machine on which the logon attempt occurred.
3. 🌐 The IP of the machine that initiated the logon attempt.
4. 🔢 The number of times the event has occurred (based on the specified time frame or the entire data set, depending on the settings).

---

## 💾 Save the Visualization

Finally, click on **"Save and return"**, and you will observe that the new visualization is added to the dashboard.
