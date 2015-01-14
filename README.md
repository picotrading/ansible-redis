Redis
=======

This role does an install of Redis via yum and starts the service for Redhat platforms.

No further configuration required.

Example
-------

```
- name: Install redis
  yum:
    name=redis
    state=present
  when:
    ansible_os_family == "RedHat"
```


Role variables
--------------
```
None
```

License
-------

MIT


Author
------

Robert Readman
