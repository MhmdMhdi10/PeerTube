# PeerTube Development Guide

## Quick Start for Developers

### Prerequisites

```bash
# Required versions
node --version  # >= 20.19 and < 21, or >= 22.12 and < 23
pnpm --version  # >= 10.x (PeerTube 8.0+)
ffmpeg -version # >= 4.3
psql --version  # >= 10.x
redis-server --version # >= 6.x
python3 --version # >= 3.8
```

### Installation for Development

```bash
# Clone the repository
git clone https://github.com/Chocobozzz/PeerTube.git
cd PeerTube

# Install dependencies
pnpm install

# Setup PostgreSQL
sudo -u postgres createuser -P peertube
sudo -u postgres createdb -O peertube -E UTF8 -T template0 peertube_dev
sudo -u postgres psql -c "CREATE EXTENSION pg_trgm;" peertube_dev
sudo -u postgres psql -c "CREATE EXTENSION unaccent;" peertube_dev

# Start Redis
redis-server

# Copy and configure development settings
cp config/default.yaml config/dev.yaml
# Edit config/dev.yaml with your database credentials

# Build the project
pnpm run build

# Start development server
pnpm run dev
```

### Development Commands

```bash
# Full build
pnpm run build

# Build specific components
pnpm run build:server    # Backend only
pnpm run build:client    # Frontend only
pnpm run build:embed     # Embed player

# Development mode (with hot reload)
pnpm run dev             # Full stack
pnpm run dev:server      # Backend only
pnpm run dev:client      # Frontend only

# Testing
pnpm run test            # Run all tests
pnpm run lint            # Run linter

# Database
pnpm run reset-password -- -u root  # Reset user password
```

---

## Project Structure

```
PeerTube/
├── apps/
│   ├── peertube-cli/        # CLI tool for instance management
│   └── peertube-runner/     # Remote transcoding runner
├── client/                   # Angular frontend
│   ├── src/
│   │   ├── app/             # Angular components
│   │   │   ├── +about/      # About pages
│   │   │   ├── +admin/      # Admin dashboard
│   │   │   ├── +login/      # Authentication
│   │   │   ├── +my-account/ # User account
│   │   │   ├── +video-watch/# Video player page
│   │   │   ├── core/        # Core services
│   │   │   └── shared/      # Shared components
│   │   ├── assets/          # Static assets
│   │   ├── locale/          # Translations
│   │   └── sass/            # Global styles
│   └── angular.json         # Angular configuration
├── config/                   # Configuration files
│   ├── default.yaml         # Default config (don't modify)
│   ├── dev.yaml             # Development config
│   └── production.yaml.example
├── packages/                 # Shared packages
│   ├── core-utils/          # Shared utilities
│   ├── ffmpeg/              # FFmpeg wrapper
│   ├── models/              # TypeScript models/types
│   ├── server-commands/     # Server command utilities
│   ├── tests/               # Test utilities
│   └── transcription/       # Video transcription
├── server/                   # Node.js backend
│   ├── core/
│   │   ├── controllers/     # API routes
│   │   │   ├── api/         # REST API endpoints
│   │   │   ├── activitypub/ # Federation endpoints
│   │   │   └── feeds/       # RSS/Atom feeds
│   │   ├── helpers/         # Utility functions
│   │   ├── initializers/    # Startup initialization
│   │   ├── lib/             # Business logic
│   │   ├── middlewares/     # Express middlewares
│   │   ├── models/          # Sequelize models
│   │   └── types/           # TypeScript types
│   └── server.ts            # Entry point
├── support/                  # Deployment support
│   ├── doc/                 # Documentation
│   │   ├── api/             # API documentation
│   │   ├── development/     # Dev guides
│   │   └── plugins/         # Plugin development
│   ├── docker/              # Docker configurations
│   ├── nginx/               # Nginx config template
│   └── systemd/             # Systemd service file
└── package.json
```

---

## Backend Architecture

### Database Models (Sequelize)

Located in `server/core/models/`:

```
models/
├── user/
│   ├── user.ts              # User model
│   ├── user-notification.ts # Notifications
│   └── user-video-history.ts
├── video/
│   ├── video.ts             # Main video model
│   ├── video-channel.ts     # Channels
│   ├── video-comment.ts     # Comments
│   ├── video-file.ts        # Video files
│   ├── video-playlist.ts    # Playlists
│   └── video-live.ts        # Live streams
├── account/
│   └── account.ts           # User accounts
├── actor/
│   ├── actor.ts             # ActivityPub actors
│   └── actor-follow.ts      # Subscriptions
└── oauth/
    └── oauth-token.ts       # Auth tokens
```

### API Controllers

Located in `server/core/controllers/api/`:

```
api/
├── users/
│   ├── index.ts             # User CRUD
│   ├── me.ts                # Current user
│   ├── my-subscriptions.ts  # Channel subscriptions
│   └── registrations.ts     # User registration
├── videos/
│   ├── index.ts             # Video CRUD
│   ├── upload.ts            # Video upload
│   ├── live.ts              # Live streaming
│   └── comment.ts           # Comments
├── video-channels/
│   └── index.ts             # Channel management
├── config.ts                # Instance config
└── plugins.ts               # Plugin management
```

### Key Services

Located in `server/core/lib/`:

```typescript
// Video transcoding
import { VideoTranscodingProfilesManager } from '@server/lib/transcoding/default-transcoding-profiles'

// User management
import { createUserAccountAndChannelAndPlaylist } from '@server/lib/user'

// Email
import { Emailer } from '@server/lib/emailer'

// Redis cache
import { Redis } from '@server/lib/redis'

// Job queue (BullMQ)
import { JobQueue } from '@server/lib/job-queue'

// Plugins
import { PluginManager } from '@server/lib/plugins/plugin-manager'
```

---

## Frontend Architecture (Angular)

### Key Modules

```
client/src/app/
├── core/                    # Singleton services
│   ├── auth/               # Authentication
│   ├── rest/               # API client
│   └── users/              # User service
├── shared/                  # Reusable components
│   ├── shared-video/       # Video components
│   ├── shared-forms/       # Form components
│   └── shared-main/        # Common components
├── +video-watch/           # Video player page
│   ├── video-watch.component.ts
│   └── comment/            # Comment section
└── +admin/                 # Admin dashboard
    ├── users/              # User management
    └── config/             # Instance config
```

### Services

```typescript
// Authentication
import { AuthService } from '@app/core/auth'

// API calls
import { VideoService } from '@app/shared/shared-main/video'
import { UserService } from '@app/core/users'

// Notifications
import { Notifier } from '@app/core/notification'
```

---

## API Reference

### Authentication

```bash
# Get OAuth token
curl -X POST https://your-instance.com/api/v1/users/token \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "grant_type=password" \
  -d "username=YOUR_USERNAME" \
  -d "password=YOUR_PASSWORD"

# Response
{
  "access_token": "...",
  "refresh_token": "...",
  "token_type": "Bearer",
  "expires_in": 86400
}
```

### Videos

```bash
# List videos
GET /api/v1/videos

# Get video details
GET /api/v1/videos/{id}

# Upload video (resumable)
POST /api/v1/videos/upload-resumable

# Update video
PUT /api/v1/videos/{id}

# Delete video
DELETE /api/v1/videos/{id}
```

### Users

```bash
# Get current user
GET /api/v1/users/me

# Update current user
PUT /api/v1/users/me

# List users (admin)
GET /api/v1/users

# Create user (admin)
POST /api/v1/users
```

### Full API Documentation

- OpenAPI spec: `support/doc/api/openapi.yaml`
- Online docs: https://docs.joinpeertube.org/api-rest-reference.html

---

## Plugin Development

### Plugin Structure

```
peertube-plugin-my-plugin/
├── package.json
├── main.js              # Server-side code
├── client/
│   └── common.js        # Client-side code
├── languages/           # Translations
│   └── en.json
└── static/              # Static files
    └── images/
```

### package.json

```json
{
  "name": "peertube-plugin-my-plugin",
  "version": "1.0.0",
  "description": "My awesome plugin",
  "engine": {
    "peertube": ">=6.0.0"
  },
  "keywords": ["peertube", "plugin"],
  "homepage": "https://github.com/...",
  "author": "Your Name",
  "bugs": "https://github.com/.../issues",
  "library": "./main.js",
  "staticDirs": {
    "static": "./static"
  },
  "css": ["./assets/style.css"],
  "clientScripts": [
    {
      "script": "client/common.js",
      "scopes": ["common"]
    }
  ],
  "translations": {
    "en": "./languages/en.json"
  }
}
```

### Server-side Plugin (main.js)

```javascript
async function register({
  registerHook,
  registerSetting,
  settingsManager,
  storageManager,
  videoCategoryManager,
  videoLicenceManager,
  videoLanguageManager,
  peertubeHelpers,
  getRouter,
  registerExternalAuth,
  registerIdAndPassAuth
}) {
  
  // Register settings
  registerSetting({
    name: 'my-setting',
    label: 'My Setting',
    type: 'input',
    default: 'default value',
    private: false
  })

  // Register hooks
  registerHook({
    target: 'action:api.video.uploaded',
    handler: ({ video, req, res }) => {
      console.log('Video uploaded:', video.name)
    }
  })

  registerHook({
    target: 'filter:api.video.get.result',
    handler: async (video) => {
      // Modify video before returning
      return video
    }
  })

  // Add custom routes
  const router = getRouter()
  
  router.get('/my-endpoint', async (req, res) => {
    const user = await peertubeHelpers.user.getAuthUser(res)
    res.json({ message: 'Hello', user: user?.username })
  })

  // Add video categories
  videoCategoryManager.addConstant(100, 'My Category')

  // Store data
  await storageManager.storeData('my-key', { value: 'data' })
  const data = await storageManager.getData('my-key')
}

async function unregister() {
  // Cleanup
}

module.exports = { register, unregister }
```

### Client-side Plugin

```javascript
function register({ registerHook, peertubeHelpers }) {
  
  // Hook into page load
  registerHook({
    target: 'action:video-watch.video.loaded',
    handler: ({ video, playlist }) => {
      console.log('Video loaded:', video.name)
    }
  })

  // Add UI elements
  registerHook({
    target: 'action:video-watch.init',
    handler: () => {
      const container = document.getElementById('plugin-placeholder-player-next')
      if (container) {
        const elem = document.createElement('div')
        elem.innerHTML = '<p>Custom content here</p>'
        container.appendChild(elem)
      }
    }
  })

  // Use helpers
  peertubeHelpers.getSettings().then(settings => {
    console.log('Plugin settings:', settings)
  })

  peertubeHelpers.notifier.success('Plugin loaded!')
}

export { register }
```

### Available Hooks

**Server-side Action Hooks:**
- `action:application.listening`
- `action:api.video.uploaded`
- `action:api.video.updated`
- `action:api.video.deleted`
- `action:api.user.created`
- `action:api.user.deleted`
- `action:api.video-comment.created`

**Server-side Filter Hooks:**
- `filter:api.video.get.result`
- `filter:api.videos.list.result`
- `filter:api.user.me.get.result`
- `filter:video.auto-blacklist.result`

**Client-side Hooks:**
- `action:application.init`
- `action:video-watch.init`
- `action:video-watch.video.loaded`
- `action:auth-user.logged-in`
- `action:auth-user.logged-out`

Full list: https://docs.joinpeertube.org/api/plugins

---

## Testing

### Running Tests

```bash
# All tests
pnpm run test

# Specific test file
pnpm run mocha -- packages/tests/src/api/videos/video-upload.ts

# With grep pattern
pnpm run mocha -- --grep "should upload"
```

### Writing Tests

```typescript
// packages/tests/src/api/my-test.ts
import { cleanupTests, createSingleServer, PeerTubeServer } from '@peertube/peertube-server-commands'

describe('My feature', function () {
  let server: PeerTubeServer

  before(async function () {
    this.timeout(30000)
    server = await createSingleServer(1)
    await server.login.getAccessToken()
  })

  it('should do something', async function () {
    const { data } = await server.videos.list()
    expect(data).to.have.lengthOf(0)
  })

  after(async function () {
    await cleanupTests([ server ])
  })
})
```

---

## Configuration Reference

### Key Configuration Options

```yaml
# config/production.yaml

# Web server settings
webserver:
  https: true
  hostname: 'your-domain.com'
  port: 443

# Database
database:
  hostname: 'localhost'
  port: 5432
  username: 'peertube'
  password: 'your-password'
  suffix: '_prod'

# Redis
redis:
  hostname: 'localhost'
  port: 6379

# Storage
storage:
  tmp: '/var/www/peertube/storage/tmp/'
  avatars: '/var/www/peertube/storage/avatars/'
  web_videos: '/var/www/peertube/storage/web-videos/'
  streaming_playlists: '/var/www/peertube/storage/streaming-playlists/'

# Object storage (optional)
object_storage:
  enabled: true
  endpoint: 's3.amazonaws.com'
  region: 'us-east-1'
  credentials:
    access_key_id: 'YOUR_KEY'
    secret_access_key: 'YOUR_SECRET'
  streaming_playlists:
    bucket_name: 'your-bucket-streaming'
  web_videos:
    bucket_name: 'your-bucket-videos'

# Transcoding
transcoding:
  enabled: true
  threads: 2
  resolutions:
    240p: true
    360p: true
    480p: true
    720p: true
    1080p: true
  hls:
    enabled: true

# Live streaming
live:
  enabled: true
  rtmp:
    port: 1935
  transcoding:
    enabled: true

# Signup
signup:
  enabled: true
  limit: 100
  requires_approval: true
  requires_email_verification: true

# Admin
admin:
  email: 'admin@your-domain.com'
```

---

## Troubleshooting

### Common Issues

**1. Video not transcoding**
```bash
# Check job queue
redis-cli LLEN bull:transcoding:waiting

# Check logs
tail -f /var/www/peertube/storage/logs/peertube.log | grep -i transcode
```

**2. Database connection issues**
```bash
# Test connection
psql -h localhost -U peertube -d peertube_prod

# Check PostgreSQL logs
sudo tail -f /var/log/postgresql/postgresql-*-main.log
```

**3. Redis connection issues**
```bash
# Test connection
redis-cli ping

# Check Redis status
sudo systemctl status redis
```

**4. FFmpeg issues**
```bash
# Check FFmpeg version
ffmpeg -version

# Test transcoding manually
ffmpeg -i input.mp4 -c:v libx264 -preset fast output.mp4
```

### Logs Location

```bash
# Application logs
/var/www/peertube/storage/logs/peertube.log

# Nginx logs
/var/log/nginx/access.log
/var/log/nginx/error.log

# PostgreSQL logs
/var/log/postgresql/

# Redis logs
/var/log/redis/
```

---

## Resources

- **Official Documentation**: https://docs.joinpeertube.org
- **API Reference**: https://docs.joinpeertube.org/api-rest-reference.html
- **Plugin API**: https://docs.joinpeertube.org/api/plugins
- **GitHub**: https://github.com/Chocobozzz/PeerTube
- **Community Chat**: https://matrix.to/#/#peertube:matrix.org
- **Forum**: https://framacolibri.org/c/peertube
