# 📊 SIEM Visualization Example 2: Failed Logon Attempts (Disabled Users)

In this SIEM visualization example we want to create visualization to monitor failed login attempts against disabled users.

We mention **"failed"** because it is not possible to log in with a disabled user, so it will never be successful even if the correct credentials are provided. In a scenario where the correct credentials are provided, the Windows logs will contain an additional **SubStatus** value of `0xC0000072`, that indicates the reason of the failure.

---

## 🚀 Spawn the Target System

Navigate to the bottom of this section and click on **Click here to spawn the target system!**.

---

## 📊 Open the Dashboard

Click on the **side navigation toggle**, and click on **"Dashboard"**.

A prebaked dashboard should be visible. Let's click on the **"pencil"/edit icon**.

<img width="1888" height="652" alt="image" src="https://github.com/user-attachments/assets/b65e6efd-4ba5-4024-884f-76abcae17143" />

---

## 📈 Create the Visualization

Now, to initiate the creation of our first visualization, we simply have to click on the **"Create visualization"** button.

Upon initiating the creation of our first visualization, the following new window will appear with various options and settings.

<img width="1030" height="638" alt="image" src="https://github.com/user-attachments/assets/eab95c91-e47f-44f1-afc1-46bab5636e25" />


---

## 🔍 Four Things to Notice

There are four things for us to notice on this window:

### 1. 🔎 Filter Option

A filter option that allows us to filter the data before creating a graph. In this case our goal is to display failed logon attempts against disabled users only.

We can use a filter to only consider event IDs that match `4625` – Failed logon attempt on a Windows system, like we did in the previous visualization example.

In this case though, we should also take into account the **SubStatus** (`winlog.event_data.SubStatus` field) that indicates, when set to `0xC0000072`, that the failure is due to a logon with disabled user.

The following image demonstrates how we can specify such a filter.

<img width="1028" height="535" alt="image" src="https://github.com/user-attachments/assets/cc162de5-bc84-4107-aef1-66d0f9f293bc" />

---

### 2. 🗂️ Index Pattern

This field indicates the data set (index) that we are going to use.

It is common for data from various infrastructure sources to be separated into different indices, such as network, Windows, Linux, etc.

In this particular example, we will specify `windows*` in the **"Index pattern"**.

---

### 3. 🔍 Field Search

This search bar provides us with the ability to double-check the existence of a specific field within our data set, serving as another way to ensure that we are looking at the correct data.

Like in the previous visualization, we are interested in the `user.name.keyword` field.

We can use the search bar to quickly perform a search and verify if this field is present and discovered within our selected data set.

This allows us to confirm that we are accessing the desired field and working with accurate data.

<img width="1030" height="793" alt="image" src="https://github.com/user-attachments/assets/f36ba9e5-97a7-4051-b04d-37479e3aa53e" />


---

### 4. 📊 Visualization Type

Lastly, this drop-down menu enables us to select the type of visualization we want to create.

The default option displayed in the earlier image is **"Bar vertical stacked"**.

If we click on that button, it will reveal additional available options (image redacted as not all options fit on the screen).

From this expanded list, we can choose the desired visualization type that best suits our requirements and data presentation needs.

<img width="711" height="712" alt="image" src="https://github.com/user-attachments/assets/f82fb6fc-a68d-4ad4-a275-8086cd0d12b3" />


---

## 📋 Select the Table

For this visualization, let's select the **"Table"** option.

After selecting the **"Table"**, we can proceed to click on the **"Rows"** option.

This will allow us to choose the specific data elements that we want to include in the table view.

<img width="728" height="935" alt="image" src="https://github.com/user-attachments/assets/79dc14c6-4c49-422f-a50a-c7ac4d3c0f16" />


Let's configure the **"Rows"** settings as follows.

<img width="703" height="839" alt="image" src="https://github.com/user-attachments/assets/5dff9f7e-af0b-43c7-90a5-dd45340d9c3c" />


---

## 🔢 Configure Metrics

Moving forward, let's close the **"Rows"** window and proceed to enter the **"Metrics"** configuration.

<img width="606" height="554" alt="image" src="https://github.com/user-attachments/assets/2d8c9100-e7eb-4951-ad70-7e7ac5b31957" />


In the **"Metrics"** window, let's select **"count"** as the desired metric.

<img width="1027" height="281" alt="image" src="https://github.com/user-attachments/assets/16a17672-d839-4b43-b380-cf7b22327dc4" />


---

## 💻 Add the Hostname

ne final addition to the table is to include another **"Rows"** setting to show the machine where the failed logon attempt occurred.

To do this, we will select the `host.hostname.keyword` field, which represents the computer reporting the failed logon attempt.

This will allow us to display the hostname or machine name alongside the count of failed logon attempts, as shown in the image.

<img width="1027" height="281" alt="image" src="https://github.com/user-attachments/assets/3145cd72-a2fd-4af4-be83-24eedc56261c" />



---

## 📊 Table Columns

Now we can see three columns in the table, which contain the following information:

1. 👤 The disabled user whose credentials generated the failed logon attempt event.
2. 💻 The machine on which the logon attempt occurred.
3. 🔢 The number of times the event has occurred (based on the specified time frame or the entire data set, depending on the settings).


