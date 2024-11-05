# XSS-LibreNMS-IPMI


**Description:**


XSS on the list parameters (Replace $DEVICE_ID with your specific $DEVICE_ID value):
- `/device/device=$DEVICE_ID/tab=edit/section=ipmi` -> param: ipmi_hostname
- `/device/device=$DEVICE_ID/tab=edit/section=ipmi` -> param: ipmi_username
- `/device/device=$DEVICE_ID/tab=edit/section=ipmi` -> param: ipmi_password
- `/device/device=$DEVICE_ID/tab=edit/section=ipmi` -> ipmi_kg_key


of Librenms version 24.9.0 ([https://github.com/librenms/librenms](https://github.com/librenms/librenms)) allows remote attackers to inject malicious scripts. When a user views or interacts with the page displaying the data, the malicious script executes immediately, leading to potential unauthorized actions or data exposure


**Proof of Concept:**
1. Add a new device through the LibreNMS interface.
2. Edit the newly created device and select the IPMI site.
3. In any of the following fields: "IPMI/BMC Hostname", "IPMI/BMC Username", "IPMI/BMC Password" or "IPMIv2 Kg Key", enter the payload: `"><script>alert(document.cookie)</script>`.
4. Save the changes.
5. Observe that when the page loads, the XSS payload executes, triggering a popup that displays the current cookies.



![payload](https://github.com/user-attachments/assets/a3aaf5fc-ed97-4c76-8025-52fdcd6f90a3)
![executed](https://github.com/user-attachments/assets/bf91bd05-a1d2-4441-961a-94ab254e29a5)


**Impact:**

Execution of Malicious Code
