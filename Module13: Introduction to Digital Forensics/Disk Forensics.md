# Disk Forensics

As we've previously highlighted, adhering to the sequence of data volatility is crucial. It's imperative that we scrutinize each byte to detect the subtle traces left by cyber adversaries. Having covered memory forensics, let's now shift our attention to the area of disk forensics (disk image examination and analysis).

## Key Functionalities of Disk Forensic Tools

Many disk forensic tools, both commercial and open-source, come packed with features. However, for incident response teams, certain functionalities stand out:

- **File Structure Insight:** Being able to navigate and see the disk's file hierarchy is crucial. Top-tier forensic tools should display this structure, allowing quick access to specific files, especially in known locations on a suspect system.
- **Hex Viewer:** For those moments when you need to get up close and personal with your data, viewing files in hexadecimal is essential. This capability is especially handy when dealing with threats like tailored malware or unique exploits.
- **Web Artifacts Analysis:** With so much user data tied to web activities, a forensic tool must efficiently sift through and present this data. It's a game-changer when you're piecing together events leading up to a user landing on a malicious website.
- **Email Carving:** Sometimes, the trail leads to internal threats. Maybe it's a rogue employee or just someone who slipped up. In such cases, emails often hold the key. A tool that can extract and present this data streamlines the process, making it easier to connect the dots.
- **Image Viewer:** At times, the images stored on systems can tell a story of their own. Whether it's for policy checks or deeper dives, having a built-in viewer is a boon.
- **Metadata Analysis:** Details like file creation timestamps, hashes, and disk location can be invaluable. Consider a scenario where you're trying to match the launch time of an app with a malware alert. Such correlations can be the linchpin in your investigation.

## Autopsy

Enter Autopsy: a user-friendly forensic platform built atop the open-source Sleuth Kit toolset. It mirrors many features you'd find in its commercial counterparts: timeline assessments, keyword hunts, web and email artifact retrievals, and the ability to sift results based on known malicious file hashes.

Once you've loaded a forensic image and processed the data, you'll see the forensic artifacts neatly organized on the side panel. From here, you can:

<img width="1001" height="728" alt="image" src="https://github.com/user-attachments/assets/1694cee9-f04c-4223-b792-d1065427543c" />

### Explore Data Sources

Dive into Data Sources to explore files and directories.

<img width="558" height="343" alt="image" src="https://github.com/user-attachments/assets/d721eb9c-8ee6-4e50-9957-d5c52f8db079" />

### Examine Web Artifacts

Examine Web Artifacts.

<img width="1999" height="737" alt="image" src="https://github.com/user-attachments/assets/f45c735a-76ef-4aaa-9769-f34a7807b4ea" />

### Check Attached Devices

Check Attached Devices.

<img width="1999" height="737" alt="image" src="https://github.com/user-attachments/assets/1f5661f2-8cd7-4c4d-b9e7-c7e20011ff5b" />

### Recover Deleted Files

Recover Deleted Files.

<img width="1999" height="705" alt="image" src="https://github.com/user-attachments/assets/4b6c2e90-5d84-4f66-b3d5-fc8d4671429a" />

### Conduct Keyword Searches

Conduct Keyword Searches.

<img width="1999" height="741" alt="image" src="https://github.com/user-attachments/assets/32edf9b6-c49f-48a5-ada6-df53ec21877e" />

<img width="1999" height="743" alt="image" src="https://github.com/user-attachments/assets/705b4a1c-2256-4f61-b45c-8fc3639b7c96" />

### Use Keyword Lists

Use Keyword Lists for targeted searches.

<img width="408" height="380" alt="image" src="https://github.com/user-attachments/assets/142d3fd8-6388-4332-9efc-5dc54d6c24cb" />

<img width="1999" height="737" alt="image" src="https://github.com/user-attachments/assets/3c78b032-9b84-44a1-8c8c-cf5866eb3678" />

### Undertake Timeline Analysis

Undertake Timeline Analysis to map out events.

<img width="1705" height="877" alt="image" src="https://github.com/user-attachments/assets/825cab61-088d-4fd1-ac5e-1f657492d637" />

We'll be heavily utilizing Autopsy in the forthcoming "Practical Digital Forensics Scenario" section.

## 🔑 Key Takeaway

Disk forensics involves examining disk images to uncover files, web and email artifacts, deleted data, metadata, and timelines that can help reconstruct suspicious activity. Autopsy, built on The Sleuth Kit, provides a user-friendly platform for exploring data sources, recovering deleted files, conducting keyword searches, examining artifacts, and performing timeline analysis.
