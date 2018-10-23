---
layout: post
title:  "How to quickly test a Suricata rule"
date:   2018-10-23 19:14:00 +0200
categories: 
---

This is the procedure for Ubuntu.

1. Install Suricata:

```
sudo apt-get install suricata
```

2. Edit `/etc/suricata/suricata.yaml`
  * Remove unwanted rules files
  * Add new custom rule file (containing your rule to test)

3. Run Suricata:

```
sudo suricata -c /etc/suricata/suricata.yaml -i eno1
```

4. Check `/var/log/suricata/fast.log` for eventual hits.
