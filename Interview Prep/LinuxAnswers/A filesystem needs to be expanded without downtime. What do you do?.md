**A filesystem needs to be expanded without downtime. What do you do?**


1. df -h      → Which filesystem is full?

2. lsblk      → Which LV/disk is it?

   <img width="557" height="502" alt="image" src="https://github.com/user-attachments/assets/3e0870d2-7da6-472f-91b2-ab6820f4085f" />


4. vgs        → Is there free space in the VG?

   <img width="412" height="466" alt="image" src="https://github.com/user-attachments/assets/ebb1b8ce-9996-4306-a154-16c2da1afc5d" />


6. pvs        → Does the PV have free space?

7. lvs        → Which LV should I extend?

   <img width="282" height="286" alt="image" src="https://github.com/user-attachments/assets/1638bce0-3287-4165-9510-077ef4f8f208" />


9. lvextend   → Extend the logical volume

    <img width="612" height="300" alt="image" src="https://github.com/user-attachments/assets/6bf30fd3-33cf-4bcf-a0c4-6791a9afcafd" />


11. xfs_growfs / resize2fs → Extend the filesystem

<img width="567" height="437" alt="image" src="https://github.com/user-attachments/assets/30dc0d29-be0e-4c25-a125-e7acf5d45d58" />


12. df -h      → Verify

    <img width="541" height="260" alt="image" src="https://github.com/user-attachments/assets/2388f4b3-d415-4bd0-8c72-9c376ae65f5c" />


