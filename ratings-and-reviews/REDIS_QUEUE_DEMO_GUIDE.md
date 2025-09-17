# Redis Queue Demonstration Guide

## Overview
This guide demonstrates how Laravel queues work with Redis as the backend, using Horizon for monitoring and management in the Mumzworld Ratings and Reviews Service.

## Queue System Architecture

### Queues Used in This Application

| Queue Name | Purpose | Job Classes | Configuration File |
|------------|---------|-------------|-------------------|
| `cache-invalidation` | CloudFront CDN cache invalidation | `InvalidateCloudFrontCache` | `config/horizon.php` |
| `statistics` | Product rating statistics calculation | `UpdateProductStatisticsJob` | `config/horizon.php` |
| `translation` | Review content translation (Google Translate) | Translation jobs | `config/horizon.php` |
| `default` | Fallback queue for miscellaneous jobs | Various | `config/queue.php` |

### Redis as Queue Backend
- **Storage**: Jobs stored as serialized PHP objects in Redis lists
- **Persistence**: Jobs persist until processed, ensuring reliability
- **Performance**: Fast job queuing and retrieval using Redis operations (LPUSH/RPOP)
- **Scalability**: Multiple workers can process jobs concurrently
- **Monitoring**: Real-time visibility through Redis Commander and Horizon

### Key Files to Examine
- **Queue Configuration**: `config/queue.php` - Redis connection settings
- **Horizon Configuration**: `config/horizon.php` - Supervisor and queue assignments
- **Job Classes**: `app/Jobs/` directory
  - `InvalidateCloudFrontCache.php` - CloudFront cache invalidation
  - `UpdateProductStatisticsJob.php` - Statistics calculation
- **Services**: `app/Services/` directory
  - `CloudFrontService.php` - Dispatches cache invalidation jobs
  - `RatingsAndReviewsStatisticsService.php` - Dispatches statistics jobs
- **Controllers**: `app/Http/Controllers/API/ReviewController.php` - Triggers jobs via API calls
- **Docker Configuration**: `docker-compose.yml` - Redis and Horizon services

### How Redis Queues Work in This App
1. **Job Creation**: API calls or services dispatch jobs to specific queues
2. **Redis Storage**: Jobs serialized and stored in Redis lists with queue-specific keys
3. **Horizon Processing**: 3 supervisors monitor different queues and spawn workers
4. **Job Execution**: Workers pull jobs from Redis, execute them, and remove from queue
5. **Monitoring**: Horizon dashboard shows real-time processing metrics

---

## Required Environment Variables

### ✅ Critical Queue Configuration
```env
# Queue Backend - MUST be Redis
QUEUE_CONNECTION=redis

# Redis Connection Details
REDIS_HOST=redis                    # Docker service name
REDIS_PORT=6379                     # Default Redis port
REDIS_PASSWORD=null                 # No password for local
REDIS_CLIENT=phpredis               # PHP Redis client
```

### ✅ Session Configuration (Optional)
```env
# Sessions stored in Redis (for web routes)
SESSION_DRIVER=redis
SESSION_LIFETIME=120
```

### ✅ Cache Configuration
```env
# Cache Driver (currently using file, can be redis)
CACHE_STORE=file                    # Change to 'redis' to see cache data
CACHE_DRIVER=redis                  # Redis available as cache backend
```

### ✅ Docker Port Configuration
```env
# Redis Commander Access
DOCKER_REDIS_COMMANDER_HOST_PORT=8081

# Application Access
DOCKER_APP_HTTP_PORT=7001

# Horizon Service
DOCKER_HORIZON_DOCKERFILE=Dockerfile.dev
```

### ⚠️ What NOT to Change
```env
# These work together - don't modify
REDIS_HOST=redis                    # Must match docker-compose service name
QUEUE_CONNECTION=redis              # Must be 'redis' for this demo
```

---

## Prerequisites

### 1. Start Docker Services
```bash
docker-compose up -d
```

### 2. Verify Services Running
```bash
docker-compose ps
```
Should show: `app`, `redis`, `redis-commander`, `dynamodb`, `dynamodb-admin`

### 3. Access Points
- **Redis Commander**: http://localhost:8081
- **Horizon Dashboard**: http://localhost:7001/horizon
- **API Endpoints**: http://localhost:7001/api/

---

## Step 1: Stop Horizon & Clear Existing Jobs

```bash
# Stop Horizon to prevent automatic job processing
docker-compose stop horizon

# Clear existing jobs (optional - clean slate)
docker-compose exec app php artisan queue:clear redis

# Verify Horizon is stopped
docker-compose ps | grep horizon
```

**✅ Verify**: Check Redis Commander - should see fewer/no queue keys

---

## Step 2: Generate Queue Jobs via API Calls

### Create Review (triggers statistics + cache jobs)
```bash
curl -X POST http://localhost:7001/api/reviews \
  -H "Content-Type: application/json" \
  -d '{
    "product_id": "demo-product-123",
    "user_id": "demo-user-456", 
    "rating": 5,
    "title": "Demo Review for Queue Testing",
    "content": "This review will create queue jobs for demonstration",
    "language": "en",
    "country": "US"
  }'
```

### Alternative: Generate Jobs via Tinker
```bash
docker-compose exec app php artisan tinker --execute="
use App\Services\CloudFrontService;
use App\Services\RatingsAndReviewsStatisticsService;

\$cloudFront = app(CloudFrontService::class);
\$stats = app(RatingsAndReviewsStatisticsService::class);

// Create jobs with identifiable data
\$timestamp = date('His');
\$stats->queueStatsRecalculation('queue-demo-product-' . \$timestamp);
\$cloudFront->invalidateProductReviewsApi('queue-demo-product-' . \$timestamp);
\$cloudFront->invalidatePaths(['/demo/queue/test/' . \$timestamp]);

echo 'Queue jobs created at ' . date('Y-m-d H:i:s') . ' with timestamp: ' . \$timestamp;
"
```

---

## Step 3: Observe Jobs in Redis Commander

### Access Redis Commander
**Go to**: http://localhost:8081

### Navigate to Database 0
- Click on **Database 0** (default Redis database)

### Look for Queue Keys
Search for keys starting with:
- `mumzworld_laravel_service_starter_kit_database_queues:cache-invalidation`
- `mumzworld_laravel_service_starter_kit_database_queues:statistics`
- `mumzworld_laravel_service_starter_kit_database_queues:default`

### Examine Job Data
**Click on queue keys to see**:
- Job class names (`App\Jobs\UpdateProductStatisticsJob`)
- Serialized job data with your custom values
- Queue metadata and timestamps
- Your custom identifiers in the payload

**Example job structure**:
```json
{
  "uuid": "...",
  "displayName": "App\\Jobs\\InvalidateCloudFrontCache",
  "job": "Illuminate\\Queue\\CallQueuedHandler@call",
  "data": {
    "commandName": "App\\Jobs\\InvalidateCloudFrontCache",
    "command": "...serialized data containing your paths..."
  }
}
```

---

## Step 4: Start Horizon & Watch Processing

```bash
# Start Horizon to begin processing jobs
docker-compose start horizon

# Verify Horizon is running
docker-compose ps | grep horizon
```

### Access Horizon Dashboard
**Go to**: http://localhost:7001/horizon

### Observe Real-time Processing
- **Dashboard**: Jobs per minute, active workers, throughput
- **Workload**: Queue status and processing by supervisor
- **Recent Jobs**: Completed jobs with execution details
- **Failed Jobs**: Any errors or exceptions

### What You'll See
1. Jobs move from **Pending** → **Processing** → **Completed**
2. **3 Supervisors** processing different queue types:
   - `supervisor-1`: `cache-invalidation` queue
   - `supervisor-2`: `translation` queue  
   - `supervisor-3`: `statistics` queue
3. **Job Details**: Click on jobs to see your custom data

---

## Step 5: Create More Jobs While Horizon Runs

```bash
# Create multiple jobs to see real-time processing
docker-compose exec app php artisan tinker --execute="
for (\$i = 1; \$i <= 5; \$i++) {
    \$productId = 'realtime-demo-' . \$i . '-' . time();
    app(App\Services\RatingsAndReviewsStatisticsService::class)->queueStatsRecalculation(\$productId);
    app(App\Services\CloudFrontService::class)->invalidateProductReviewsApi(\$productId);
    echo 'Created jobs for product: ' . \$productId . PHP_EOL;
    sleep(1); // 1 second delay to see processing
}
echo 'All jobs created - watch Horizon dashboard!';
"
```

**Watch in Horizon**: Jobs appear and get processed immediately

---

## Step 6: Verify Complete Job Lifecycle

### Check Redis (jobs should be processed)
- Refresh Redis Commander
- Queue keys should be empty or have fewer jobs
- Processed jobs are removed from Redis

### Check Horizon Dashboard
- View **Recent Jobs** tab
- Check processing times and success rates
- Look for your custom data in job details
- Verify no jobs in **Failed Jobs**

---

## Understanding the Queue Architecture

### Redis Database Structure
```
Database 0 (default):
├── queues:cache-invalidation     # CloudFront invalidation jobs
├── queues:statistics            # Product statistics jobs  
├── queues:translation           # Review translation jobs
├── queues:default              # Fallback queue
└── sessions:*                  # User sessions (if any)
```

### Horizon Configuration
```php
// config/horizon.php - 3 Supervisors
'supervisor-1' => ['queue' => ['cache-invalidation']],
'supervisor-2' => ['queue' => ['translation']],
'supervisor-3' => ['queue' => ['statistics']],
```

### Job Flow
```
API Request → Service → Job Dispatch → Redis Queue → Horizon Worker → Job Execution
```

---

## Troubleshooting

### No Jobs Appearing in Redis
```bash
# Check queue connection
docker-compose exec app php artisan tinker --execute="
echo 'Queue connection: ' . config('queue.default');
echo 'Redis host: ' . config('database.redis.default.host');
"
```

### Jobs Not Processing
```bash
# Check Horizon status
docker-compose exec app php artisan horizon:status

# Restart Horizon
docker-compose restart horizon
```

### Redis Connection Issues
```bash
# Test Redis connection
docker-compose exec app php artisan tinker --execute="
use Illuminate\Support\Facades\Redis;
try {
    Redis::ping();
    echo 'Redis connection: OK';
} catch (Exception \$e) {
    echo 'Redis error: ' . \$e->getMessage();
}
"
```

---

## Quick Commands Reference

```bash
# Queue Management
docker-compose stop horizon              # Stop job processing
docker-compose start horizon             # Start job processing
docker-compose exec app php artisan queue:clear redis  # Clear all queues

# Monitoring
docker-compose logs horizon              # View Horizon logs
docker-compose exec app php artisan horizon:status     # Check status

# Testing
docker-compose exec app php artisan queue:work --once  # Process one job
```

---

## Expected Results

### ✅ What You Should See

1. **In Redis Commander**:
   - Queue keys with job data
   - Serialized PHP objects containing your custom data
   - Jobs disappear after processing

2. **In Horizon Dashboard**:
   - Real-time job processing metrics
   - Completed jobs with execution times
   - Your custom identifiers in job payloads
   - Separate supervisors handling different queue types

3. **Job Processing Flow**:
   - Jobs queued instantly when created
   - Processing within seconds when Horizon runs
   - Clean completion without errors

### ❌ Common Issues

- **No session data**: Normal for API-only applications
- **Cache not in Redis**: Using `CACHE_STORE=file` instead of `redis`
- **Jobs stuck**: Horizon not running or Redis connection issues

---

## Environment Variables Summary

| Variable | Value | Purpose |
|----------|-------|---------|
| `QUEUE_CONNECTION` | `redis` | **Required** - Queue backend |
| `REDIS_HOST` | `redis` | **Required** - Docker service name |
| `REDIS_PORT` | `6379` | **Required** - Redis port |
| `SESSION_DRIVER` | `redis` | Optional - Session storage |
| `CACHE_STORE` | `file`/`redis` | Optional - Cache backend |
| `DOCKER_REDIS_COMMANDER_HOST_PORT` | `8081` | **Required** - Redis UI access |
| `DOCKER_APP_HTTP_PORT` | `7001` | **Required** - App access |

**This guide demonstrates the complete Laravel queue ecosystem with Redis and Horizon!**