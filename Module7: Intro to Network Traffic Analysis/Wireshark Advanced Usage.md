# Wireshark Advanced Usage

In this section, we will cover some advanced usage with Wireshark. The project developers have included many different capabilities ranging from tracking TCP conversations to cracking wireless credentials. The inclusion of many different plugins makes Wireshark one of the best traffic analysis tools.

## Plugins

The analyze and statistics radials provide a plethora of plugins to run against the capture. In this section, we will work through a couple of them. We would cover all of which Wireshark offers, but sadly, it is simply not achievable in an introductory module. I urge everyone to experiment and play as we go through this journey.

## The Statistics and Analyze Tabs

The Statistics and Analyze tabs can provide us with great insight into the data we are examining. From these points, we can utilize many of the baked-in plugins Wireshark has to offer.

The plugins here can give us detailed reports about the network traffic being utilized. It can show us everything from the top talkers in our environment to specific conversations and even breakdown by IP and protocol.

### Statistics Tab

<img width="1891" height="1059" alt="image" src="https://github.com/user-attachments/assets/9e0f6c9b-9419-459a-8018-6dd8c50c812d" />

### Analyze

From the Analyze tab, we can utilize plugins that allow us to do things such as following TCP streams, filter on conversation types, prepare new packet filters and examine the expert info Wireshark generates about the traffic. Below are a few examples of how to use these plugins.

### Analyze Tab

<img width="595" height="568" alt="image" src="https://github.com/user-attachments/assets/fd0c5c2d-a7b3-4c04-91f8-e083384b29a7" />

## Following TCP Streams

Wireshark can stitch TCP packets back together to recreate the entire stream in a readable format. This ability also allows us to pull data (images, files, etc.) out of the capture. This works for almost any protocol that utilizes TCP as a transport mechanism.

To utilize this feature:

1. right-click on a packet from the stream we wish to recreate.
2. select follow → TCP
3. this will open a new window with the stream stitched back together. From here, we can see the entire conversation.

### Follow A Stream Via GUI

<img width="3810" height="1935" alt="image" src="https://github.com/user-attachments/assets/7768a443-402f-488a-b5cf-1a215869f470" />

Alternatively, we can utilize the filter `tcp.stream eq #` to find and track conversations captured in the pcap file.

### Filter For A Specific TCP Stream

<img width="3839" height="770" alt="image" src="https://github.com/user-attachments/assets/d7bb7532-cf0b-4557-8824-009d25b3411b" />

Notice that the first three packets in the image above have a full TCP handshake. Following those packets, we can see the stream transferring data. We have cleared anything not related out of view by utilizing the filter, and we now can see the conversation in order.

## Extracting Data and Files From a Capture

Wireshark can recover many different types of data from streams. It requires you to have captured the entire conversation. Otherwise, this ability will fail to put an incomplete datagram back together. If we want a more in-depth understanding of how this capability works, check out the Network Foundations Module or research TCP/IP fragmentation.

To extract files from a stream:

1. stop your capture.
2. Select the File radial → Export → , then select the protocol format to extract from.
3. (DICOM, HTTP, SMB, etc.)

### Extract Files From The GUI

<img width="3810" height="1935" alt="image" src="https://github.com/user-attachments/assets/99e185c4-51f9-4586-9607-bad452676289" />

Another exciting way to grab data out of the pcap file comes from FTP. The File Transfer Protocol moves data between a server and host to pull it out of the raw bytes and reconstruct the file. (image, text documents, etc.) FTP utilizes TCP as its transport protocol and uses ports 20 & 21 to function. TCP port 20 is used to transfer data between the server and host, while port 21 is used as the FTP control port. Any commands such as login, listing files, and issuing download or uploads happen over this port. To do so, we need to look at the different FTP display filters in Wireshark. A complete list of these can be found here. For now, we will look at three:

- `ftp` - Will display anything about the FTP protocol.

  We can utilize this to get a feel for what hosts/servers are transferring data over FTP.

### FTP Desector
<img width="1215" height="874" alt="image" src="https://github.com/user-attachments/assets/c8a980ae-36c4-45c7-998f-3f242fb9b194" />

- `ftp.request.command` - Will show any commands sent across the ftp-control channel ( port 21 )

  We can look for information like usernames and passwords with this filter. It can also show us filenames for anything requested

  ### FTP-Request-Command Filter

<img width="1291" height="869" alt="image" src="https://github.com/user-attachments/assets/00bd1212-d1a0-458d-a379-985ebb2fa9fb" />

- `ftp-data` - Will show any data transferred over the data channel ( port 20 )

  If we filter on a conversation and utilize ftp-data, we can capture anything sent during the conversation. We can reconstruct anything transferred by placing the raw data back into a new file and naming it appropriately.

  ### FTP-Data Filter

<img width="1083" height="818" alt="image" src="https://github.com/user-attachments/assets/c976f7a7-119d-4c5f-8238-0fe39b309d02" />

Since FTP utilizes TCP as its transport mechanism, we can utilize the follow tcp stream function we utilized earlier in the section to group any conversation we wish to explore. The basic steps to dissect FTP data from a pcap are as follows:

1. Identify any FTP traffic using the ftp display filter.
2. Look at the command controls sent between the server and hosts to determine if anything was transferred and who did so with the ftp.request.command filter.
3. Choose a file, then filter for ftp-data. Select a packet that corresponds with our file of interest and follow the TCP stream that correlates to it.
4. Once done, Change "Show and save data as" to "Raw" and save the content as the original file name.
5. Validate the extraction by checking the file type.
