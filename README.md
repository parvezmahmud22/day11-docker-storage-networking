# day11-docker-storage-networking
Today's mission: run PostgreSQL in Docker, delete the container, and prove the data survives.

🔴 Problem
Containers are disposable. When one is removed, its writable layer and everything in it is gone. A database can't live like that.

🛠 What I built

docker network create app-net
docker volume create pgdata
docker run -d --name db --network app-net \
  -v pgdata:/var/lib/postgresql/data \
  postgres:16-alpine

✅ 1. Persistence proof

Inserted a row: I survive!
Ran docker rm -f db and deleted the container
Started a new container with the same volume
SELECT * FROM notes; returned the row again 🎉

The data lived in the named volume, not in the container.

🌐 2. Custom bridge network and DNS
A second container connected with psql -h db, using the container name.

Default bridge: ping db → name not resolved ❌
Custom app-net: ping db → resolved ✅

User-defined bridges have built-in DNS. Names stay stable even when IPs change.

🚪 3. Port conflict on purpose
Two nginx containers on host port 18080 → port is already allocated.
Fix: -p 18081:80.
Containers can share a container port, but never the same host port.

💾 4. Volume backup
Stopped the database, then archived the volume:

docker run --rm -v pgdata:/data -v "$(pwd):/backup" alpine \
  tar czf /backup/pgdata-backup.tgz -C /data .

Backup size: [X] MB. Restarted db and pg_isready confirmed it was accepting connections.

🧠 What I learned

A volume is live storage; a backup is a separate copy for recovery. A volume is not a backup
Stop the database before a file-level copy, or you risk an inconsistent backup
A backup is only useful if you've tested restoring it
-e POSTGRES_PASSWORD=changeme is for local labs only, never real systems

🐛 A bug I hit
My INSERT with 'I survive!' silently failed in bash: event not found. The ! triggers history expansion. Fix: set +H, or write the text as $$I survive!$$. Always check that your data actually landed.

💡 Analogy: the container is a rented flat, the volume is a bank locker. Move out and the furniture is gone, but the locker keeps your things.
