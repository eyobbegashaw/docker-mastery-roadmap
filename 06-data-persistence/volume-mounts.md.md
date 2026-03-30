 

### Core Concept
Volumes are Docker-managed storage units independent of container lifecycle. They persist data across container removals, support backup/restore, and can be shared between containers. Volumes use dedicated storage directory (`/var/lib/docker/volumes/`) and can use various drivers (local, NFS, EBS, Azure File).

### Key Terminologies
- **Volume Driver**: Plugin implementing volume operations (create, mount, unmount, delete)
- **Named Volume**: User-created with explicit name (vs anonymous volumes)
- **Volume Mount**: Mounting volume to container path (`-v myvol:/data`)
- **Volume Options**: Driver-specific options (e.g., `type=nfs,device=:/export,o=addr=10.0.0.1`)
- **Volume Backup**: Tar streaming volume contents
- **Volume Prune**: Removing unused volumes

### How it Works
Volume lifecycle:
1. `docker volume create` - calls driver's `Create` operation, creates directory in `/var/lib/docker/volumes/volumename/_data`
2. Container mount - during container creation, kernel performs bind mount from volume directory to container path
3. Driver may handle network storage (NFS mount inside volume directory)
4. Volume persists after container deletion
5. `docker volume rm` - calls driver's `Remove` after ensuring no mounts

Volume drivers implement mount propagation (private, rshared, rslave) for complex setups. Local driver supports `type=tmpfs` for in-memory volumes.

### CLI Commands/Syntax
```bash
# Volume management
docker volume create --driver local --opt type=tmpfs --opt device=tmpfs --opt o=size=100m,uid=1000 fast-vol
docker volume create --driver local --opt type=nfs --opt o=addr=192.168.1.100,rw,nfsvers=4 --opt device=:/export nfs-vol
docker run -v myapp_data:/var/lib/mysql --mount type=volume,src=myapp_data,dst=/data,readonly mysql:8

# Backup and restore
docker run --rm -v myapp_data:/data -v $(pwd):/backup alpine tar czf /backup/myapp_backup.tar.gz -C /data .
docker run --rm -v myapp_data:/data -v $(pwd):/backup alpine tar xzf /backup/myapp_backup.tar.gz -C /data

# Inspection and cleanup
docker volume ls --filter "dangling=true"
docker volume inspect myapp_data | jq '.[0].Mountpoint'
docker volume prune -f --filter "label!=keep"

# Share volume between containers
docker run -d --name writer --mount source=shared-vol,target=/data alpine sh -c "while true; do date >> /data/log; sleep 1; done"
docker run --rm --name reader --mount source=shared-vol,target=/data alpine cat /data/log
```

### Best Practices
- **Always use named volumes** for production (anonymous volumes difficult to manage)
- **Backup volumes before container upgrades** using `--volumes-from` pattern
- **Use volume labels** for organization: `docker volume create --label environment=prod --label team=db db-data`
- **Set appropriate mount propagation** for Docker-in-Docker scenarios: `--mount type=volume,src=vol,dst=/var/lib/docker,propagation=rslave`
- **Monitor volume disk usage**: `du -sh /var/lib/docker/volumes/*/_data`
- **Use `--mount` syntax** over `-v` for clarity (explicit options)
- **Implement volume encryption** via driver if storing sensitive data

---