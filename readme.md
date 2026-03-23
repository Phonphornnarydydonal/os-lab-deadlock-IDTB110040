# Quantum Vault Deadlock Lab

This README documents the step-by-step completion of the Quantum Vault Deadlock Lab. Screenshots and explanations for each level are included.

---

## Level 1: Virtual Vault Provisioning

- Created two virtual drives: `vault_alpha.img` and `vault_beta.img` (10MB each).  
- Formatted both with `ext4` filesystem.  
- Mounted the drives using `udisksctl` and created symlinks `mount_alpha` and `mount_beta`.  

![Level 1 Mount Verification](screenshots/1.png)

**Explanation:**  
The `df -h | grep loop` output shows both loop devices are mounted, verifying that the virtual drives are accessible.

---

## Level 2 & 3: Local Deadlock

- Created scripts `sync_up` (locks Alpha → Beta) and `sync_down` (locks Beta → Alpha).  
- Executed simultaneously, the scripts froze due to circular wait (classic deadlock).  
- Verified with `ps aux | grep sync_` to see stuck processes.  

![Level 2 Deadlock](screenshots/2.png)
![Level 3 Deadlock](screenshots/3.png)

**Explanation:**  
`sync_up` holds the Alpha lock and waits for Beta, while `sync_down` holds Beta and waits for Alpha. Both scripts cannot proceed, causing a deadlock.

---

## Level 4: Multiplayer Deadlock

- Created public DMZ folders (`public_dr_alpha` and `public_dr_beta`) with lock files accessible by a partner.  
- Wrote cross-user scripts `cross_sync_alpha` and `cross_sync_beta`.  
- Simultaneous execution caused a distributed deadlock across user accounts.  

![Level 4 Multiplayer Deadlock](screenshots/4.png)

**Explanation:**  
Both scripts wait for the partner’s lock while holding their own, simulating a cross-user distributed denial-of-service scenario.

---

## Level 5: Global Resource Ordering (Deadlock Prevention)

- Enforced a global lock acquisition order: Alpha → Beta.  
- Updated `cross_sync_beta` to acquire Alpha first, then Beta.  
- Re-tested simultaneous execution; scripts completed sequentially without freezing.  

![Level 5 Resource Ordering](screenshots/5.png)

**Explanation:**  
By acquiring locks in the same global order, the circular wait condition is broken. Deadlock is prevented, and synchronization completes safely.

---

## Level 6: Deadlock Recovery (Timeout Patch)

- Created `sync_timeout` script using `flock -w 5` to wait a maximum of 5 seconds.  
- If a lock cannot be acquired, the script aborts gracefully instead of freezing.  
- Tested against `sync_up` holding Alpha lock.  

![Level 6 Timeout Recovery](screenshots/6.png)

**Explanation:**  
The timeout ensures that hung processes do not block the system indefinitely, improving server stability.

---

## Level 7: Safe Ejection (Teardown)

- Created `teardown` script to safely unmount both vaults and detach loop devices.  
- Verified with `df -h | grep loop`; no loop devices remained.  

![Level 7 Teardown](screenshots/7.png)

**Explanation:**  
Proper teardown prevents filesystem corruption and frees kernel resources, ensuring safe system cleanup.

---

## Folder Structure
