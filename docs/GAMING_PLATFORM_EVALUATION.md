# PeerTube Evaluation for Gaming Video Platform MVP

## Executive Summary

**Recommendation: Use PeerTube as your foundation**

PeerTube is a mature, production-ready video streaming platform that can significantly accelerate your MVP development. For a gaming tutorial/promotion platform with subscription-based access, PeerTube provides 80-90% of the core functionality out of the box.

---

## 1. What PeerTube Provides Out of the Box

### Core Video Features
- Video upload, transcoding, and streaming (HLS + Web Videos)
- Live streaming support (RTMP/RTMPS)
- Video chapters, captions, and storyboards
- Automatic video transcription (AI-powered)
- Multiple resolution support (144p to 4K)
- Video playlists and channels
- Video comments and interactions
- Video search with tags and categories

### User Management
- User registration with approval workflow
- Role-based access (Admin, Moderator, User)
- Video quotas per user
- User blocking/unblocking
- Two-factor authentication
- OAuth2 authentication
- External auth (LDAP, SAML, OpenID Connect via plugins)

### Content Creator Features
- Video channels (up to 20 per user)
- Channel collaborators
- Video scheduling
- Video studio (cut, watermark, intro/outro)
- Analytics and view statistics
- Support button for donations

### Admin Features
- Full admin dashboard
- User management
- Content moderation (abuse reports, blocklists)
- Instance configuration via web UI
- Plugin/theme management
- Federation controls

### Technical Infrastructure
- REST API (OpenAPI 3.0 documented)
- WebSocket support
- Object storage support (S3-compatible)
- Redis caching
- PostgreSQL database
- Docker deployment ready
- Horizontal scaling support

---

## 2. What You Need to Build/Customize

### For Your MVP (Subscription Model)

#### High Priority Customizations

1. **Subscription/Payment System**
   - PeerTube has NO built-in payment/subscription system
   - Options:
     - Build a plugin integrating Stripe/PayPal
     - Use external service (Memberful, Patreon integration)
     - Custom middleware between your payment system and PeerTube API

2. **User Type Differentiation (Contributors vs Subscribers)**
   - Modify user roles or create custom roles via plugin
   - Contributors = Users with upload rights
   - Subscribers = Users with view-only access
   - Use video privacy settings (Private/Internal) + access control

3. **Gaming-Specific Categories**
   - Add custom video categories via plugin:
   ```javascript
   videoCategoryManager.addConstant(100, 'Game Tutorials')
   videoCategoryManager.addConstant(101, 'Game Promotions')
   videoCategoryManager.addConstant(102, 'Gameplay')
   ```

#### Medium Priority Customizations

4. **Custom Landing Page/Branding**
   - Create a custom theme
   - Override default templates
   - Custom CSS variables for branding

5. **Contributor Verification System**
   - Plugin to verify game developers/creators
   - Badge system for verified contributors

6. **Enhanced Analytics for Contributors**
   - Plugin to provide detailed video analytics
   - Revenue sharing reports (if applicable)

---

## 3. Technical Architecture

### Recommended Stack
```
┌─────────────────────────────────────────────────────────┐
│                    Your Custom Layer                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │ Payment     │  │ Subscription│  │ Custom          │  │
│  │ Gateway     │  │ Manager     │  │ Analytics       │  │
│  └─────────────┘  └─────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                    PeerTube Core                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │ Video       │  │ User        │  │ API             │  │
│  │ Engine      │  │ Management  │  │ (REST/WebSocket)│  │
│  └─────────────┘  └─────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                    Infrastructure                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐ │
│  │PostgreSQL│  │  Redis   │  │  S3/Minio│  │ FFmpeg  │ │
│  └──────────┘  └──────────┘  └──────────┘  └─────────┘ │
└─────────────────────────────────────────────────────────┘
```

### Infrastructure Requirements

#### Minimum (MVP - up to 100 concurrent users)
- 2 CPU cores
- 4GB RAM
- 100GB SSD storage
- 100 Mbps bandwidth

#### Recommended (Growth - up to 1000 concurrent users)
- 4-8 CPU cores
- 8-16GB RAM
- 500GB+ SSD or Object Storage
- 1 Gbps bandwidth
- Separate transcoding workers

#### Production (Scale)
- Kubernetes deployment
- Multiple PeerTube instances behind load balancer
- Dedicated transcoding cluster (or use PeerTube Remote Runners)
- CDN for video delivery
- Object storage (AWS S3, Backblaze B2, etc.)

---

## 4. Development Effort Estimation

### Using PeerTube (Recommended)

| Task | Effort | Timeline |
|------|--------|----------|
| Initial setup & deployment | Low | 1-2 days |
| Custom theme/branding | Medium | 3-5 days |
| Subscription plugin development | High | 2-3 weeks |
| User role customization | Medium | 1 week |
| Gaming categories setup | Low | 1 day |
| Testing & QA | Medium | 1 week |
| **Total MVP** | | **4-6 weeks** |

### Building from Scratch

| Task | Effort | Timeline |
|------|--------|----------|
| Video upload/storage system | Very High | 4-6 weeks |
| Video transcoding pipeline | Very High | 3-4 weeks |
| HLS streaming implementation | High | 2-3 weeks |
| User authentication system | Medium | 1-2 weeks |
| Video player development | High | 2-3 weeks |
| Admin dashboard | High | 2-3 weeks |
| API development | High | 3-4 weeks |
| Subscription system | High | 2-3 weeks |
| Testing & QA | High | 2-3 weeks |
| **Total MVP** | | **20-30 weeks** |

---

## 5. Plugin Development Guide

### Creating a Subscription Plugin

```javascript
// peertube-plugin-gaming-subscriptions/main.js
async function register({
  registerHook,
  registerSetting,
  settingsManager,
  storageManager,
  peertubeHelpers,
  getRouter
}) {
  // Add subscription settings
  registerSetting({
    name: 'stripe-api-key',
    label: 'Stripe API Key',
    type: 'input-password',
    private: true
  })

  // Hook into video access
  registerHook({
    target: 'filter:api.video.get.result',
    handler: async (video, { user }) => {
      if (!video) return video
      
      // Check if user has active subscription
      const hasSubscription = await checkUserSubscription(user, storageManager)
      
      if (!hasSubscription && video.privacy.id === 4) { // Internal videos
        throw new Error('Subscription required')
      }
      
      return video
    }
  })

  // Custom API routes
  const router = getRouter()
  
  router.post('/subscribe', async (req, res) => {
    // Handle subscription creation
  })
  
  router.get('/subscription-status', async (req, res) => {
    const user = await peertubeHelpers.user.getAuthUser(res)
    // Return subscription status
  })
}

module.exports = { register }
```

### Plugin package.json Structure

```json
{
  "name": "peertube-plugin-gaming-subscriptions",
  "version": "1.0.0",
  "description": "Subscription management for gaming platform",
  "engine": {
    "peertube": ">=6.0.0"
  },
  "keywords": ["peertube", "plugin", "subscription", "gaming"],
  "library": "./main.js",
  "staticDirs": {},
  "css": [],
  "clientScripts": [
    {
      "script": "client/subscription-ui.js",
      "scopes": ["common"]
    }
  ]
}
```

---

## 6. Deployment Options

### Option A: Docker (Recommended for MVP)

```bash
# Quick start
cd /your/peertube/directory
curl https://raw.githubusercontent.com/chocobozzz/PeerTube/master/support/docker/production/docker-compose.yml > docker-compose.yml
curl https://raw.githubusercontent.com/Chocobozzz/PeerTube/master/support/docker/production/.env > .env

# Configure .env with your settings
nano .env

# Start
docker compose up -d
```

### Option B: Manual Installation

See `support/doc/production.md` for detailed steps:
1. Install dependencies (Node.js 20+, PostgreSQL, Redis, FFmpeg)
2. Create peertube user
3. Setup database
4. Download and configure PeerTube
5. Setup Nginx reverse proxy
6. Configure SSL with Let's Encrypt
7. Setup systemd service

### Option C: Cloud Platforms

- **AWS**: EC2 + RDS + S3 + CloudFront
- **DigitalOcean**: Droplet + Managed PostgreSQL + Spaces
- **Hetzner**: VPS + Object Storage (cost-effective)

---

## 7. API Integration Examples

### User Registration with Subscription

```javascript
// 1. Create user via API
const response = await fetch('https://your-instance.com/api/v1/users', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${adminToken}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    username: 'newuser',
    email: 'user@example.com',
    password: 'securepassword',
    role: 2, // Regular user
    videoQuota: -1,
    channelName: 'user-channel'
  })
})

// 2. After payment confirmation, update user role/permissions
// via your subscription plugin
```

### Video Upload for Contributors

```javascript
// Get upload URL
const initResponse = await fetch('https://your-instance.com/api/v1/videos/upload-resumable', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${userToken}`,
    'X-Upload-Content-Type': 'video/mp4',
    'X-Upload-Content-Length': fileSize
  },
  body: JSON.stringify({
    name: 'Game Tutorial - Level 1',
    channelId: channelId,
    privacy: 4, // Internal (subscribers only)
    category: 100, // Game Tutorials
    tags: ['tutorial', 'gaming', 'beginner']
  })
})

// Upload video chunks...
```

---

## 8. Customization Constraints

### What You CAN Easily Customize
- UI themes and branding
- Video categories and licenses
- User registration flow
- Access control via plugins
- Custom API endpoints
- Email templates
- Landing pages

### What Requires More Effort
- Core video player modifications
- Database schema changes
- Authentication flow changes
- Federation behavior (if you want to disable it)

### What You Should NOT Change
- Core transcoding pipeline (use profiles instead)
- ActivityPub implementation (unless you fork)
- Core security mechanisms

---

## 9. Licensing Considerations

PeerTube is licensed under **AGPL-3.0**:
- You CAN use it commercially
- You CAN modify it
- You MUST release source code of modifications if you distribute
- You MUST keep the same license for derivatives
- Plugins are NOT considered derivatives (you can keep them proprietary)

**For your startup**: 
- Keep core PeerTube as-is
- Build features as plugins (can be proprietary)
- If you modify core, you must open-source those changes

---

## 10. Recommended MVP Roadmap

### Week 1-2: Foundation
- [ ] Deploy PeerTube instance (Docker)
- [ ] Configure basic settings
- [ ] Setup domain and SSL
- [ ] Create admin accounts

### Week 3-4: Customization
- [ ] Create custom theme with your branding
- [ ] Add gaming-specific categories
- [ ] Configure user registration (approval required)
- [ ] Setup email notifications

### Week 5-6: Subscription System
- [ ] Develop subscription plugin
- [ ] Integrate payment gateway (Stripe)
- [ ] Implement access control for premium content
- [ ] Test payment flows

### Week 7: Testing & Launch
- [ ] End-to-end testing
- [ ] Performance testing
- [ ] Security review
- [ ] Soft launch with beta users

---

## 11. Hardware Requirements (from Official FAQ)

### Minimum Requirements
- 1 vCore
- 1.5 GB RAM
- Sufficient storage for videos
- 20 Mbit/s upload network speed

### Recommended for 1,000 Concurrent Viewers
- 4 vCore
- 4 GB RAM
- 1 Gbit/s upload network speed
- See [scalability guide](https://docs.joinpeertube.org/maintain/configuration#scalability)

### With Local Transcoding
- 8 vCore
- 8 GB RAM

> For detailed hardware guidance, see `docs/PEERTUBE_FEATURES_FAQ.md`

---

## 12. Cost Estimation (Monthly)

### MVP Phase (Small Scale)
| Item | Cost |
|------|------|
| VPS (4 CPU, 8GB RAM) | $40-80 |
| Object Storage (500GB) | $10-25 |
| Domain + SSL | $1-2 |
| Email service | $0-20 |
| **Total** | **$50-130/month** |

### Growth Phase
| Item | Cost |
|------|------|
| Multiple VPS instances | $200-400 |
| Object Storage (2TB+) | $40-100 |
| CDN | $50-200 |
| Managed Database | $50-100 |
| Monitoring | $20-50 |
| **Total** | **$360-850/month** |

---

## 13. Supported Platforms

### Web Browsers
- Firefox 78+ (desktop)
- Latest Edge (desktop)
- Latest Chrome (desktop/mobile)
- Safari 14+ (macOS), Safari 14.5+ (iOS)
- Latest Firefox (mobile)

### Mobile Apps (Official)
- **Android:** Google Play, F-Droid, Manual APK
- **iOS:** App Store

---

## 14. Related Documentation

- `docs/PEERTUBE_FEATURES_FAQ.md` - Complete feature list from official FAQ
- `docs/DEVELOPMENT_GUIDE.md` - Technical development reference
- `docs/QUICK_START_DEPLOYMENT.md` - Deployment instructions

---

## Conclusion

**PeerTube is the right choice for your MVP** because:

1. **Time to Market**: 4-6 weeks vs 20-30 weeks from scratch
2. **Cost Efficiency**: Focus budget on unique features, not reinventing video streaming
3. **Proven Technology**: Battle-tested by hundreds of instances worldwide
4. **Extensibility**: Plugin system allows custom features without forking
5. **Community**: Active development and community support
6. **Scalability**: Can grow from MVP to production scale

**Key Actions**:
1. Start with Docker deployment today
2. Begin subscription plugin development in parallel
3. Plan your custom theme/branding
4. Define your contributor verification process

The main development effort will be the subscription/payment integration, which is a well-understood problem with many existing solutions to reference.
