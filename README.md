# Stored XSS-LibreNMS-IPMI


**Description:**


Stored XSS on the list parameters (Replace $DEVICE_ID with your specific $DEVICE_ID value):
- `/device/device=$DEVICE_ID/tab=edit/section=ipmi` -> param: ipmi_hostname
- `/device/device=$DEVICE_ID/tab=edit/section=ipmi` -> param: ipmi_username
- `/device/device=$DEVICE_ID/tab=edit/section=ipmi` -> param: ipmi_password
- `/device/device=$DEVICE_ID/tab=edit/section=ipmi` -> param: ipmi_kg_key


of Librenms version 24.10.0 ([https://github.com/librenms/librenms](https://github.com/librenms/librenms)) allows remote attackers to inject malicious scripts. When a user views or interacts with the page displaying the data, the malicious script executes immediately, leading to potential unauthorized actions or data exposure.

The vulnerability is present at  [librenms/includes/html/pages/device/edit/ipmi.inc.php](https://github.com/librenms/librenms/blob/master/includes/html/pages/device/edit/ipmi.inc.php).

The issue here is that the input values from $_POST ($ipmi_hostname, $ipmi_port, $ipmi_username, $ipmi_password, and $ipmi_kg_key) are being assigned directly without validation or sanitization (at [Line 5-9](https://github.com/librenms/librenms/blob/master/includes/html/pages/device/edit/ipmi.inc.php#L5-L9)) and are then saved by `set_dev_attrib`.

```php
$ipmi_hostname = $_POST['ipmi_hostname'];
$ipmi_port = (int) $_POST['ipmi_port'];
$ipmi_username = $_POST['ipmi_username'];
$ipmi_password = $_POST['ipmi_password'];
$ipmi_kg_key = $_POST['ipmi_kg_key'];
```

And the values retrieved from get_dev_attrib($device, 'ipmi_.....') are displayed directly in the HTML value attribute without any encoding at lines: [Line 62](https://github.com/librenms/librenms/blob/master/includes/html/pages/device/edit/ipmi.inc.php#L62), [Line 68](https://github.com/librenms/librenms/blob/master/includes/html/pages/device/edit/ipmi.inc.php#L68), [Line 74](https://github.com/librenms/librenms/blob/master/includes/html/pages/device/edit/ipmi.inc.php#L74), [Line 80](https://github.com/librenms/librenms/blob/master/includes/html/pages/device/edit/ipmi.inc.php#L80), [Line 86](https://github.com/librenms/librenms/blob/master/includes/html/pages/device/edit/ipmi.inc.php#L86).

```html
<input id="ipmi_hostname" name="ipmi_hostname" class="form-control" value="<?php echo get_dev_attrib($device, 'ipmi_hostname'); ?>" />
...
<input id="ipmi_port" name="ipmi_port" class="form-control" value="<?php echo get_dev_attrib($device, 'ipmi_port'); ?>" placeholder="623" />
...
<input id="ipmi_username" name="ipmi_username" class="form-control" value="<?php echo get_dev_attrib($device, 'ipmi_username'); ?>" />
...
<input id="ipmi_password" name="ipmi_password" type="password" class="form-control" value="<?php echo get_dev_attrib($device, 'ipmi_password'); ?>" />
...
<input id="ipmi_kg_key" name="ipmi_kg_key" type="password" class="form-control" value="<?php echo get_dev_attrib($device, 'ipmi_kg_key'); ?>" placeholder="A0FE1A760B304... (Leave blank if none)" />
```

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
