ubuntu 18.04 更换默认python版本
```
root@en-us-cpt-k8s-crm-prod:~# update-alternatives --config python
There are 4 choices for the alternative python (providing /usr/bin/python).

  Selection    Path                Priority   Status
------------------------------------------------------------
* 0            /usr/bin/python3     150       auto mode
  1            /usr/bin/python2     100       manual mode
  2            /usr/bin/python2.7   2         manual mode
  3            /usr/bin/python3     150       manual mode
  4            /usr/bin/python3.6   1         manual mode

Press <enter> to keep the current choice[*], or type selection number: 2
update-alternatives: using /usr/bin/python2.7 to provide /usr/bin/python (python) in manual mode
root@en-us-cpt-k8s-crm-prod:~# update-alternatives --config python
There are 4 choices for the alternative python (providing /usr/bin/python).

  Selection    Path                Priority   Status
------------------------------------------------------------
  0            /usr/bin/python3     150       auto mode
  1            /usr/bin/python2     100       manual mode
* 2            /usr/bin/python2.7   2         manual mode
  3            /usr/bin/python3     150       manual mode
  4            /usr/bin/python3.6   1         manual mode

Press <enter> to keep the current choice[*], or type selection number: 
root@en-us-cpt-k8s-crm-prod:~# python
Python 2.7.17 (default, Sep 30 2020, 13:38:04) 
[GCC 7.5.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
```