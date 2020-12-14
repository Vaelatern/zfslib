# zfslib
ZFS Libraries for Python

This code is based on code from [fs-tools by Rudd-O](https://github.com/Rudd-O/zfs-tools) and is used as a sample of how to use his ZFS libraries to write your own utilities that work with ZFS DataSets / Snapshots.


Notable additions so far:
### <Dataset>.get_snapshots(dict)
```
    # find_snapshots(dict) - Query all snapshots in Dataset and optionally filter by: 
    #  * name: Snapshot name (wildcard supported) 
    #  * dt_from: datetime to start
    #  * tdelta: timedelta -or- string of nC where: n is an integer > 0 and C is one of y,m,d,H,M,S. Eg 5H = 5 Hours
    #  * dt_to: datetime to stop 
    #  * Date searching is any combination of:
    #    .  (dt_from --> dt_to) | (dt_from --> dt_from + tdelta) | (dt_to - tdelta --> dt_to)
    ...
```
### <Dataset>.get_diffs()
```
    # get_diffs() - Gets Diffs in snapshot or between snapshots (if snap_to is specified)
    # snap_from - Left side of diff
    # snap_to - Right side of diff. If not specified, diff is to current working version
    # include - list of glob expressions to include (eg ['*_pycache_*'])
    # ignore - list of glob expressions to ignore (eg ['*_pycache_*'])
    # file_type - Filter on the following
        # B       Block device
        # C       Character device
        # /       Directory
        # >       Door
        # |       Named pipe
        # @       Symbolic link
        # P       Event port
        # =       Socket
        # F       Regular file
    # chg_type - Filter on the following:
        # -       The path has been removed
        # +       The path has been created
        # M       The path has been modified
        # R       The path has been renamed
```

### <Snapshot>.snap_path
```
    # Returns the path to read only zfs_snapshot directory (<ds_mount>/.zfs/snapshots/<snapshot>)
```

### <Snapshot>.resolve_snap_path(path)
```
    # Resolves the path to file/dir within the zfs_snapshot dir
    # Returns: tuple(of bool, str) where:
    # - bool = True if item is found
    # - str = Path to item if found else path to zfs_snapshot dir
```

### <Diff>.snap_path_left
```
    # Path to resource on left side of diff in zfs_snapshot dir
```

### <Diff>.snap_path_right
```
    # Path to resource on right side of diff in .zfs_snapshot dir or working copy
```

See `test.py` for sample code


```python
from connection import ZFSConnection

pool_name = 'rpool'
ds_name = 'devel'

# Read ZFS information from local computer
# Change properties as needed
src_conn = ZFSConnection(host='localhost',properties=["name", "avail", "usedsnap", "usedds", "usedrefreserv", "usedchild", "creation"])

# Load pool
pool = src_conn.pools.lookup(pool_name)

# Load dataset
ds = pool.get_child(ds_name)

# Load snapshots by name and date/time range
snapshots = zfslib.getSnapshots(ds, name='autosnap*', dt_to=datetime.now(), tdelta=timedelta(hours=24))

for snap in snapshots:
    diffs = zfslib.get_diffs(src_conn, snap_from=snap, ignore=['*.vscod*', '*_pycache_*'])
    for diff in diffs:
        print('{} - {}'.format(snap.name, diff))

ds_name_full = f"{pool_name}/{ds_name}"

# Print datasets creation date
print(f"{ds_name_full} creation date: {ds.get_creation()}")

# Grab Snapshots
snapshots = ds.get_snapshots()

# Load snapshot by index or iterate and filter as required
snap = snapshots[1]

# Get Snapshot Creation date
print(f"{ds_name_full}@{snap.name} creation date: {ds.get_creation()}")

# Read property from DataSet / Snapshot
print(f"{ds_name_full}@{snap.name} usedsnap: {snap.get_property('usedsnap')}")


```
