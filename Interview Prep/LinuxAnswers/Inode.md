# 📦 What is an Inode in Linux?

## Definition

An **inode (Index Node)** is a data structure in Linux that stores **metadata about a file**, NOT the file name or file content.

 It contains a file's size, permissions, ownership, timestamps, and the disk block pointers needed to locate the file's contents.
---

## What inode stores

Each inode contains:

- File type (file, directory, etc.)
- File size
- Permissions (rwx)
- Owner (user/group)
- Timestamps (created, modified, accessed)
- Number of links (hard links count)
- Pointers to actual data blocks on disk

---

## What inode does NOT store

- ❌ File name
- ❌ File path
- ❌ File content

File names are stored in **directory entries**, not in inode.

---

## Simple Flow

```text id="inodeflow"
File Name → Directory Entry → Inode → Data Blocks (actual content)


###Check inode number

ls -i file.txt


output - 10245 file.txt


Why inode is important?


Hard links work using inode


Linux identifies files using inode internally


File name is just a label, inode is the real identity

