# n8n Production Docker Compose with Resource Limits

Production-ready Docker Compose configurations for n8n with PostgreSQL, optimized for different hardware specifications. Each configuration includes strict resource limits to prevent system instability and ensure predictable performance.

**✨ Updated for n8n 1.117.3** with Task Runners support for secure Code node execution.

## 🎯 Philosophy

These configurations are built on four core principles:

1. **Strict Resource Limits**: CPU and RAM are explicitly divided between services to prevent resource starvation
2. **Controlled Concurrency**: Hard limits on parallel workflow executions prevent CPU overload
3. **Efficiency Over Quantity**: Fewer workers with higher concurrency outperform many workers with low concurrency
4. **Secure Code Execution**: Task runners provide isolated, secure execution for JavaScript and Python code

## 📊 Available Configurations

### 1 vCPU / 4 GB RAM - Single Instance Mode
**Path**: [`1vcpu-4gb-main-mode/`](./1vcpu-4gb-main-mode)

- **Architecture**: Single n8n instance with internal task runners
- **Best for**: Small deployments, development, testing
- **Concurrency**: 5 parallel workflows
- **Task Runners**: Internal mode (no extra container)
- **Resource allocation**:
  - PostgreSQL: 0.5 vCPU + 1.5 GB RAM
  - n8n: 0.5 vCPU + 1.5 GB RAM
  - Total: 1 vCPU + 3 GB RAM (1 GB free for OS)

### 2 vCPU / 8 GB RAM
**Path**: [`2vcpu-8gb/`](./2vcpu-8gb)

#### Single Instance Mode ([`main-mode/`](./2vcpu-8gb/main-mode))
- **Architecture**: Single n8n instance with internal task runners
- **Best for**: Moderate workloads, simpler setup
- **Concurrency**: 10 parallel workflows
- **Task Runners**: Internal mode (no extra container)
- **Resource allocation**:
  - PostgreSQL: 1 vCPU + 3.5 GB RAM
  - n8n: 1 vCPU + 4 GB RAM
  - Total: 2 vCPU + 7.5 GB RAM (0.5 GB free for OS)

#### Queue Mode ([`queue-mode/`](./2vcpu-8gb/queue-mode))
- **Architecture**: 1 main + 1 worker + Redis + task-runners
- **Best for**: Better separation of concerns, webhook reliability
- **Concurrency**: 8 per worker
- **Task Runners**: External mode (sidecar container)
- **Resource allocation**:
  - PostgreSQL: 1 vCPU + 3.5 GB RAM
  - Redis: 0.2 vCPU + 512 MB RAM
  - n8n-main: 0.5 vCPU + 1.5 GB RAM
  - n8n-worker: 1 vCPU + 2.5 GB RAM
  - task-runners: 0.3 vCPU + 512 MB RAM
  - Total: 3 vCPU + 8.5 GB RAM

### 4 vCPU / 16 GB RAM
**Path**: [`4vcpu-16gb/`](./4vcpu-16gb)

#### Single Instance Mode ([`main-mode/`](./4vcpu-16gb/main-mode))
- **Architecture**: Single n8n instance with internal task runners
- **Best for**: High workloads without queue complexity
- **Concurrency**: 20 parallel workflows
- **Task Runners**: Internal mode (no extra container)
- **Resource allocation**:
  - PostgreSQL: 2 vCPU + 7 GB RAM
  - n8n: 2 vCPU + 9 GB RAM
  - Total: 4 vCPU + 16 GB RAM

#### Queue Mode - 2 Workers ([`queue-mode/2-workers/`](./4vcpu-16gb/queue-mode/2-workers)) ⭐ **RECOMMENDED**
- **Architecture**: 1 main + 2 workers + Redis + task-runners
- **Best for**: Maximum performance and stability
- **Concurrency**: 12 per worker (24 total)
- **Task Runners**: External mode (sidecar container)
- **Resource allocation**:
  - PostgreSQL: 1.5 vCPU + 6 GB RAM
  - Redis: 0.3 vCPU + 1 GB RAM
  - n8n-main: 0.7 vCPU + 2 GB RAM
  - n8n-worker-1: 0.75 vCPU + 3.5 GB RAM
  - n8n-worker-2: 0.75 vCPU + 3.5 GB RAM
  - task-runners: 0.3 vCPU + 512 MB RAM
  - Total: 4.55 vCPU + 16.5 GB RAM

#### Queue Mode - 3 Workers ([`queue-mode/3-workers/`](./4vcpu-16gb/queue-mode/3-workers)) ⚠️ **NOT RECOMMENDED**
- **Architecture**: 1 main + 3 workers + Redis + task-runners
- **Best for**: Only if 2-workers are constantly at 100% CPU (rare)
- **Concurrency**: 10 per worker (30 total)
- **Task Runners**: External mode (sidecar container)
- **Resource allocation**:
  - PostgreSQL: 1.3 vCPU + 5.5 GB RAM
  - Redis: 0.3 vCPU + 1 GB RAM
  - n8n-main: 0.6 vCPU + 1.8 GB RAM
  - n8n-worker-1: 0.6 vCPU + 2.5 GB RAM
  - n8n-worker-2: 0.6 vCPU + 2.5 GB RAM
  - n8n-worker-3: 0.6 vCPU + 2.5 GB RAM
  - task-runners: 0.3 vCPU + 512 MB RAM
  - Total: 4.7 vCPU + 16.3 GB RAM

## 🚀 Quick Start

### 1. Choose Your Configuration

Select the configuration that matches your hardware:

```bash
cd 2vcpu-8gb/queue-mode/
```

### 2. Configure Environment Variables

Copy and edit the `.env` file (or `dockploy-environment-settings.env`):

```bash
# Generate encryption key
openssl rand -base64 32

# Generate database password
openssl rand -base64 64
```

Required variables:
```env
N8N_HOST=n8n.yourdomain.com
N8N_ENCRYPTION_KEY=your_generated_key_here
POSTGRES_PASSWORD=your_generated_password_here
POSTGRES_USER=n8n
POSTGRES_DB=n8n
GENERIC_TIMEZONE=Europe/Paris

# For queue mode only (external task runners)
N8N_RUNNERS_AUTH_TOKEN=your_generated_token_here
```

### 3. Deploy

```bash
docker-compose up -d
```

### 4. Verify

```bash
# Check all services are running
docker-compose ps

# Check logs
docker-compose logs -f n8n
```

## 🔒 Task Runners (New in n8n 1.117+)

### What are Task Runners?

Task runners provide secure, isolated execution for JavaScript and Python code in the Code node. They prevent malicious or buggy code from affecting your n8n instance.

### Internal vs External Mode

#### Internal Mode (Single Instance Deployments)
- **Used in**: All single instance (main mode) configurations
- **How it works**: n8n launches task runners as child processes
- **Pros**: Simple, no extra containers, lower overhead
- **Cons**: Shares resources with n8n process
- **Best for**: 1-2 vCPU systems, development, testing

#### External Mode (Queue Mode Deployments)
- **Used in**: All queue mode configurations
- **How it works**: Separate `task-runners` container (sidecar)
- **Pros**: Better isolation, independent scaling, more secure
- **Cons**: Extra container overhead
- **Best for**: Production queue mode deployments

### Configuration

Task runners are pre-configured in all docker-compose files. For queue mode, you need to set:

```env
N8N_RUNNERS_AUTH_TOKEN=your_secure_token_here
```

Generate with: `openssl rand -base64 32`

### Monitoring Task Runners

Check if task runners are active:
```bash
# Check n8n logs for task runner registration
docker-compose logs n8n | grep -i "runner"

# You should see: "Registered runner 'JS Task Runner'"
```

## 📈 Performance Tuning

### Why Fewer Workers Perform Better

Based on real-world testing, **2 workers with high concurrency outperform 8 workers with low concurrency by 63%**.

**The Problem with Many Workers:**
- More database lock contention
- More connection overhead
- Workers spend time waiting for each other
- Database becomes the bottleneck

**The Solution:**
- Fewer workers (2-3 maximum)
- Higher concurrency per worker (10-15)
- Efficient connection reuse
- Less database contention

### Database Connection Math

**2 Workers at Concurrency 15:**
- Connections used: ~16-24
- Connections available: ~270+ (out of 300)
- Plenty of room for traffic spikes

**8 Workers at Concurrency 5:**
- Connections used: ~48-64
- Connections available: ~30-50
- One spike and you're done

### When to Scale Up

Only add more workers if:
1. ✅ Current workers are at 100% CPU constantly
2. ✅ Redis queue depth consistently > 50
3. ✅ Database is NOT the bottleneck (< 70% CPU)
4. ✅ You've monitored for several days

## 🔧 Monitoring

### Check Database Connections

```sql
SELECT count(*) FROM pg_stat_activity;
```
Should stay under 50 with 2 workers.

### Check Redis Queue

```bash
docker exec -it <redis-container> redis-cli LLEN bull:n8n:jobs:waiting
```
Should stay under 50. If consistently higher, increase concurrency (not workers).

### Check Worker Health

In n8n UI: Settings > Workers
- Shows active workers and their status
- Monitor execution times (should be consistent)

### Check Resource Usage

```bash
docker stats
```
Monitor CPU and memory usage of each container.

## ⚠️ Common Mistakes

### ❌ DON'T: Add more workers when performance is slow
**Why**: More workers = more database contention = slower performance

### ✅ DO: Increase concurrency per worker
**Why**: Better connection reuse = better performance

### ❌ DON'T: Set concurrency below 5
**Why**: Inefficient database connection usage

### ✅ DO: Use 10-15 concurrency per worker
**Why**: Optimal balance between throughput and stability

### ❌ DON'T: Run more than 3 workers without enterprise database
**Why**: Database locks become a major bottleneck

### ✅ DO: Stick with 2 workers for most use cases
**Why**: Best performance-to-stability ratio

## 🔐 Security Best Practices

1. **Never commit `.env` files** - Use `.env.example` instead
2. **Backup your encryption key** - Store it securely offline (you cannot recover credentials without it!)
3. **Use strong passwords** - Generate with `openssl rand -base64 64`
4. **Secure task runners** - Generate unique `N8N_RUNNERS_AUTH_TOKEN` for queue mode
5. **Enable HTTPS** - Use a reverse proxy (Nginx/Traefik/Dokploy)
6. **Regular backups** - Backup PostgreSQL and n8n volumes
7. **Environment variable access** - Set `N8N_BLOCK_ENV_ACCESS_IN_NODE=true` if you don't need env vars in Code nodes
8. **Git node security** - `N8N_GIT_NODE_DISABLE_BARE_REPOS=true` is enabled by default

## 📦 Backup & Restore

### Backup

```bash
# Backup PostgreSQL
docker exec <postgres-container> pg_dump -U n8n n8n > backup.sql

# Backup n8n data
docker run --rm -v n8n_data:/data -v $(pwd):/backup alpine tar czf /backup/n8n-data.tar.gz /data
```

### Restore

```bash
# Restore PostgreSQL
docker exec -i <postgres-container> psql -U n8n n8n < backup.sql

# Restore n8n data
docker run --rm -v n8n_data:/data -v $(pwd):/backup alpine tar xzf /backup/n8n-data.tar.gz -C /
```

## 🐛 Troubleshooting

### "Too many connections" error

**Solution**: Increase `max_connections` in PostgreSQL or reduce worker concurrency.

### Workers not processing jobs

**Check**:
1. All workers use the same `N8N_ENCRYPTION_KEY`
2. Redis is accessible from all workers
3. PostgreSQL is accessible from all workers

### Task runners not starting (Queue Mode)

**Check**:
1. `N8N_RUNNERS_AUTH_TOKEN` is set and identical in n8n-main and task-runners
2. Task runners can reach n8n-main on port 5679
3. Check logs: `docker-compose logs task-runners`

### Code node execution fails

**Check**:
1. Task runners are registered (check n8n logs for "Registered runner")
2. For external mode: task-runners container is running
3. Check task runner logs for errors

### High memory usage

**Solution**: Reduce concurrency or add more RAM. Check for memory-heavy workflow nodes.

### Slow workflow execution

**Check**:
1. Database CPU usage (should be < 70%)
2. Number of database connections (should be < 50)
3. Redis queue depth (should be < 50)

### Deprecation warnings on startup

**Solution**: These configurations are updated for n8n 1.117.3. Old variables like `EXECUTIONS_PROCESS=main` and `EXECUTIONS_MODE=queue` have been removed.

## 📚 Additional Resources

- [n8n Official Documentation](https://docs.n8n.io/)
- [n8n Task Runners Documentation](https://docs.n8n.io/hosting/configuration/task-runners/)
- [n8n Queue Mode Guide](https://docs.n8n.io/hosting/scaling/queue-mode/)
- [n8n Environment Variables](https://docs.n8n.io/hosting/configuration/environment-variables/)
- [PostgreSQL Connection Pooling](https://www.postgresql.org/docs/current/runtime-config-connection.html)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Dokploy Documentation](https://dokploy.com/docs) (if using Dokploy)

## 🤝 Contributing

Contributions are welcome! Please:
1. Test your changes thoroughly
2. Update documentation
3. Follow the existing configuration style
4. Include resource allocation details

## 📄 License

MIT License - Feel free to use and modify for your needs.

## 🆕 Changelog

### Version 2.0 (November 2024)
- ✅ Updated for n8n 1.117.3
- ✅ Added Task Runners support (internal and external modes)
- ✅ Removed deprecated environment variables (`EXECUTIONS_PROCESS`, `EXECUTIONS_MODE`)
- ✅ Added security variables (`N8N_BLOCK_ENV_ACCESS_IN_NODE`, `N8N_GIT_NODE_DISABLE_BARE_REPOS`)
- ✅ Optimized worker concurrency for better stability
- ✅ Added task-runners sidecar containers for queue mode
- ✅ Updated documentation with task runners information

### Version 1.0 (Initial Release)
- Initial production-ready configurations
- Resource limits for 1, 2, and 4 vCPU systems
- Main and queue mode variants

## ⭐ Acknowledgments

These configurations are based on:
- Real-world production testing
- n8n community best practices
- Performance optimization research
- Database scaling principles
- n8n 1.117.3 official documentation

## 🙏 Special Thanks

- n8n team for the excellent workflow automation platform
- Community members who shared their production experiences
- Contributors who helped test and improve these configurations

---

**Need help?** Open an issue with:
- Your hardware specs
- Current configuration
- Observed behavior
- Expected behavior
- n8n version
