
 

### Core Concept
Union filesystems overlay multiple directories (branches) to present a single unified view. Docker uses OverlayFS (overlay2 storage driver) where image layers are read-only, container layer is read-write. Copy-on-Write (CoW) means file modifications copy to writable layer, preserving original in read-only layer.

### Key Terminologies
- **Lowerdir**: Read-only layers (image layers)
- **Upperdir**: Read-write layer (container changes)
- **Workdir**: Filesystem for atomic operations (rename exchange)
- **Merged**: Unified view presented to container
- **Whiteout File**: Character device marking deleted files from lower layers
- **Metacopy**: Performance optimization for small files

### How it Works
OverlayFS operations:
1. **File read**: Check upperdir → if exists, return; else check lowerdirs (highest priority first)
2. **File write (copy_up)**: Copy file from lowerdir to upperdir, then modify
3. **File delete**: Create whiteout (.wh.filename) in upperdir
4. **Directory rename**: Requires copy_up and index entry for hardlink management

Layer storage layout:
```
/var/lib/docker/overlay2/
├── <layer1>/diff/    # Actual file content
├── <layer1>/link     # Shortened layer ID
├── <layer2>/diff/
└── <container>/      # Upperdir + workdir + merged
```

### CLI Commands/Syntax
```bash
# Inspect storage driver
docker info | grep -A 5 "Storage Driver"
docker system df -v  # Layer disk usage

# Manual OverlayFS mount
mkdir -p /tmp/{lower,upper,work,merged}
echo "from lower" > /tmp/lower/file.txt
sudo mount -t overlay overlay -o lowerdir=/tmp/lower,upperdir=/tmp/upper,workdir=/tmp/work /tmp/merged

# Convert layers to tarballs
docker save alpine:latest -o alpine.tar
tar -xf alpine.tar -C alpine-extracted
cat alpine-extracted/manifest.json | jq .

# Optimize layer merging
docker build --squash -t myapp:squashed .  # (experimental)
```

### Best Practices
- **Prefer overlay2** driver (default, best performance, supports page cache sharing)
- **Order RUN commands** from least to most frequently changing layers (leverage cache)
- **Avoid storing secrets** in any layer - use Docker secrets or external vaults
- **Monitor inode exhaustion** in overlay directories: `df -i /var/lib/docker`
- **Use `--storage-opt size=10G`** for devicemapper (not needed for overlay2)
- **Clean dangling layers** weekly: `docker system prune -f --filter "until=168h"`

---