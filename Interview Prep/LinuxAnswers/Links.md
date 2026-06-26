## What is a Link?

A **link** is a way to access the same file using another name or path.


Linux provides two types of links:

1. **Soft Link (Symbolic Link)**
2. **Hard Link**

# 1. Soft Link (Symbolic Link)

## Definition

A **soft link** (or **symbolic link**) is a special file that stores the **path** of another file or directory.


## Create a Soft Link

ln -s source_file softlink_name

# 2. Hard Link

## Definition

A **hard link** is another directory entry that points to the **same inode** (same physical data on disk) as the original file.


## Create a Hard Link


ln source_file hardlink_name


# Key Differences

| Feature | Soft Link | Hard Link |
|----------|-----------|-----------|
| Points to | File path | Same inode |
| Inode number | Different | Same |
| Can link directories | ✅ Yes | ❌ Usually No |
| Cross filesystem | ✅ Yes | ❌ No |
| Works if original deleted | ❌ No (broken link) | ✅ Yes |
| Similar to | Windows shortcut | Another name for the same file |
