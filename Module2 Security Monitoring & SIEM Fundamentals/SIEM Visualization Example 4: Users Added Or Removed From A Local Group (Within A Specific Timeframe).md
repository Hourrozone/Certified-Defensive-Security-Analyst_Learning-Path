# 📊 SIEM Visualization Example 4: Users Added Or Removed From A Local Group (Within A Specific Timeframe)

In this SIEM visualization example, we aim to create a visualization to monitor user additions or removals from the local "Administrators" group from March 5th 2023 to date.

Our visualization will be based on the following Windows event logs.

* `4732`: A member was added to a security-enabled local group
* `4733`: A member was removed from a security-enabled local group

Navigate to the bottom of this section and click on **Click here to spawn the target system!**.

click on the side navigation toggle, and click on **"Dashboard"**.

A prebaked dashboard should be visible. Let's click on the **"pencil"/edit icon**.

<img width="1888" height="652" alt="image" src="https://github.com/user-attachments/assets/cdc6322a-2f34-4db1-8f93-b7ca683d68bc" />

Now, to initiate the creation of our first visualization, we simply have to click on the **"Create visualization"** button.

Upon initiating the creation of our first visualization, the following new window will appear with various options and settings.

<img width="1030" height="638" alt="image" src="https://github.com/user-attachments/assets/9e8be9f9-edff-4c24-af34-76edf287a927" />


## 🔎 There are four things for us to notice on this window:

### 1. 🔍 Filter Option

A filter option that allows us to filter the data before creating a graph. In this case our goal is to display user additions or removals from the local "Administrators" group. We can use a filter to only consider event IDs that match `4732` – A member was added to a security-enabled local group and `4733` – A member was removed from a security-enabled local group. We can also use a filter to only consider `4732` and `4733` events where the local group is the "Administrators" one.


<img width="1030" height="473" alt="image" src="https://github.com/user-attachments/assets/6bece19d-ca2a-4c5d-aa66-79fe7d233fd6" />


### 2. 🗂️ Index Pattern

This field indicates the data set (index) that we are going to use. It is common for data from various infrastructure sources to be separated into different indices, such as network, Windows, Linux, etc. In this particular example, we will specify `windows*` in the **"Index pattern"**.

### 3. 🔎 Search Bar

This search bar provides us with the ability to double-check the existence of a specific field within our data set, serving as another way to ensure that we are looking at the correct data. We are interested in the `user.name.keyword` field. We can use the search bar to quickly perform a search and verify if this field is present and discovered within our selected data set. This allows us to confirm that we are accessing the desired field and working with accurate data.

<img width="460" height="1043" alt="image" src="https://github.com/user-attachments/assets/c33889a4-dd04-4db4-9fa9-f41b620a2f38" />


### 4. 📊 Visualization Type

Lastly, this drop-down menu enables us to select the type of visualization we want to create. The default option displayed in the earlier image is **"Bar vertical stacked"**. If we click on that button, it will reveal additional available options (image redacted as not all options fit on the screen). From this expanded list, we can choose the desired visualization type that best suits our requirements and data presentation needs.

<img width="1030" height="793" alt="image" src="https://github.com/user-attachments/assets/b43ff6f8-2ff6-437a-b65a-dabf66b4d382" />


## 📋 Table Visualization

For this visualization, let's select the **"Table"** option. After selecting the **"Table"**, we can proceed to click on the **"Rows"** option. This will allow us to choose the specific data elements that we want to include in the table view.

<img width="711" height="712" alt="image" src="https://github.com/user-attachments/assets/a0e77776-ef7f-4f8d-8f26-4e252c603883" />


Let's configure the **"Rows"** settings as follows.

<img width="728" height="935" alt="image" src="https://github.com/user-attachments/assets/d670120f-c191-43b3-9c30-c2ed86d2b4c2" />


Moving forward, let's close the **"Rows"** window and proceed to enter the **"Metrics"** configuration.

<img width="606" height="554" alt="image" src="https://github.com/user-attachments/assets/f4acf274-7f23-4117-8ecc-c22b7881473c" />


In the **"Metrics"** window, let's select **"count"** as the desired metric.

<img width="1169" height="332" alt="image" src="https://github.com/user-attachments/assets/a804b24e-e8ca-4cd7-9355-29ff8e29fe80" />


## 🧩 Additional Rows Settings

One final addition to the table is to include some more **"Rows"** settings to enhance our understanding.

* Which user was added to or removed from the group? (`winlog.event_data.MemberSid.keyword` field)
* To which group was the addition or the removal performed? (double-checking that it is the "Administrators" one) (`group.name.keyword` field)
* Was the user added to or removed from the group? (`event.action.keyword` field)
* On which machine did the action occur? (`host.name.keyword` field)

<img width="703" height="839" alt="image" src="https://github.com/user-attachments/assets/f0b59121-fdf4-4ecb-ba5c-7432dad44ca4" />


Click on **"Save and return"**, and you will observe that the new visualization is added to the dashboard.

## ⏱️ Specific Timeframe

As discussed, we want to monitor user additions or removals from the local "Administrators" group within a specific timeframe (March 5th 2023 to date).

We can narrow the scope of our visualization as follows.

<img width="1917" height="1413" alt="image" src="https://github.com/user-attachments/assets/6964f461-4001-4e39-90a3-32c6e86b73c2" />


<img width="1921" height="1409" alt="image" src="https://github.com/user-attachments/assets/b558de1c-50a8-4da9-8c08-2954a29f86b4" />


<img width="1923" height="1403" alt="image" src="https://github.com/user-attachments/assets/35a76791-9a7b-4a1c-9ab2-57aecb6d6cd2" />


Finally, let's click on the **"Save"** button so that all our edits persist.

## Result

<img width="1903" height="293" alt="image" src="https://github.com/user-attachments/assets/a2a8aa93-722a-497d-a607-a417883754c0" />

**NOTE:** This Document is really helpful to guide us how to filter an event [ECS-Event](https://www.elastic.co/docs/reference/ecs/ecs-event)
