# PeerTube Features & FAQ Summary

*Extracted from the official PeerTube FAQ at joinpeertube.org*

---

## What is PeerTube?

PeerTube is a tool you install on a web server to create a video hosting website - essentially your "homemade YouTube." Unlike YouTube, it's not designed to centralize videos on a single massive platform. Instead, PeerTube creates a network of multiple small interconnected video hosting providers.

---

## Main Advantages

### Open Code Under Free/Libre License
- PeerTube is freely provided, no payment required to install
- Source code is auditable and transparent
- Can be enhanced by community contributions

### Federation of Interconnected Hosting Providers
- Decentralizes video storage and decision-making power
- Display videos and accounts from other PeerTube instances
- Based on [ActivityPub](https://activitypub.rocks) - connects with tools like Mastodon

### Peer-to-Peer Broadcasting (WebRTC)
- Reduces server bandwidth overload if a video goes viral
- Viewers become active participants in video broadcasting
- Based on WebRTC, a free and open-source project

---

## Features for Viewers

### Player Features
- Video chapters visible in progress bar
- Preview frames while scrubbing
- Automatic player size adjustment based on video ratio
- Subtitles, playback speed, resolution settings
- Live chat integration (via plugin)

### P2P Participation
- Help share videos using P2P
- Easily disable P2P in account settings

### Interface Customization
- Update profile (name, avatar, description)
- NSFW policy settings (display, blur, or hide)
- Theme selection
- Video filtering (language, category, live/VOD)
- Autoplay preferences

### Library Management
- Built-in "Watch Later" playlist
- Public or private playlists
- Automatic video resume
- Watch history

### Subscriptions
- Subscribe to local or remote channels
- Dedicated subscriptions feed

### Sharing & Interaction
- Share video/playlist URLs with custom attributes
- Download videos
- Comment and rate using PeerTube or ActivityPub software (Mastodon, Pleroma)

### Search
- Search local or remote content
- Advanced filters (tags, category, licence)
- Fetch remote content via URL

---

## Features for Content Creators

### Platform Choice
- Join communities based on topic, terms of service, or code of conduct
- Talk to real human moderators
- Export all data (videos, settings, channels) and import to another instance

### Upload & Import
- Upload or import from web (YouTube, Dailymotion) or torrent
- Automatic channel/playlist synchronization
- Metadata: category, licence, language, tags, description, thumbnail
- Privacy options: public, unlisted, password protected, internal, private
- Subtitles and chapters support
- Audio file upload (PeerTube creates video with image)

### Video Management
- Original file storage option
- Detailed statistics (views, watch time, unique viewers)
- Video studio (cut, watermark, intro/outro)
- Upload new video versions

### Live Streaming
- RTMP compatible (OBS, Restream, ffmpeg)
- Permanent lives (same URL for multiple streams)
- Replay creation
- Live chat integration

### Channel Management
- Multiple channels per account
- Channel branding (name, banner, avatar)
- Support button for donations
- ActivityPub subscriptions (Mastodon, Pleroma)
- Public playlists per channel

---

## Features for Administrators

### Free and Open-Source
- Install on your own server
- Official Docker support
- Documented REST API and CLI tools

### Federation (ActivityPub)
- Enable/disable federation
- Display remote videos/accounts
- Auto-follow from public index

### P2P (WebRTC)
- Reduce bandwidth with P2P on VOD and live
- Enable/disable P2P per instance
- Redundancy system for mutual instance support

### Video Transcoding
- Multiple resolution generation with ffmpeg
- Additional video extensions (.mkv, .mov, .avi) and audio uploads
- HLS playlists or raw MP4
- Custom ffmpeg profiles via plugins
- Remote runner transcoding

### Live Streaming
- Enable/disable live streaming
- Replay support
- Simultaneous stream limits
- Multi-resolution transcoding
- Remote runner support

### Video Imports
- youtube-dl integration (YouTube, Dailymotion)
- WebTorrent support (torrent file or magnet URI)

### Moderation Tools
- Signup control (enable/disable, manual approval, upload limits)
- Moderator/admin roles
- Abuse report management dashboard
- Auto-block untrusted user videos pending review
- Video blocking with reasons
- User bans, account/instance muting

### Configuration
- Instance metadata (avatar, banner, name, description, categories)
- Custom homepage (markdown/HTML)
- Plugin and theme installation
- External auth (LDAP, OpenID Connect)
- Default user settings
- Filesystem or object storage
- User notification banners

---

## Hardware Requirements

### Minimum
- 1 vCore
- 1.5 GB RAM
- Sufficient storage for videos
- 20 Mbit/s upload network speed

### Recommended (1,000 concurrent viewers)
- 4 vCore
- 4 GB RAM
- Sufficient storage for videos
- 1 Gbit/s upload network speed
- See [scalability guide](https://docs.joinpeertube.org/maintain/configuration#scalability)

### With Local Transcoding
- 8 vCore
- 8 GB RAM

### Detailed Requirements

**CPU:** PeerTube is not CPU-bound except for transcoding. One thread is sufficient for basic operation, but transcoding benefits greatly from additional cores. Transcoding can be offloaded to [remote runners](https://docs.joinpeertube.org/admin/remote-runners).

**RAM:** 1.5 GB is sufficient for basic instances (typically uses ~500 MB). More may be needed if Redis/PostgreSQL are on non-SSD systems.

**Storage Considerations:**
- Total size of videos to stream
- Transcoding multiplier (multiple resolutions)
- Format choices (Web Video + HLS doubles storage)
- Consider [Object Storage](https://docs.joinpeertube.org/maintain/remote-storage) for large instances

**Network:** Divide available bandwidth by average stream bandwidth for capacity estimate. Example: 1 Gbit/s uplink with 5 Mbit/s streams = ~200 simultaneous viewers maximum.

---

## Supported Browsers

### Desktop
- Firefox 78+
- Latest Edge
- Latest Chrome
- Safari 14+ (macOS)

### Mobile
- Latest Chrome
- Latest Firefox
- Safari 14.5+ (iOS)

---

## Mobile Applications

### Android
- [Google Play](https://play.google.com/store/apps/details?id=org.framasoft.peertube)
- [F-Droid](https://f-droid.org/packages/org.framasoft.peertube/)
- [Manual APK](https://builds.joinpeertube.org/mobile/)

### iOS
- [App Store](https://apps.apple.com/app/peertube/id6737834858)

Third-party clients: [docs.joinpeertube.org/use/third-party-application](https://docs.joinpeertube.org/use/third-party-application)

---

## Scaling Options

PeerTube doesn't support multiple nodes behind a load balancer, but can scale horizontally in these areas:

- **Bandwidth:** [Redundancy system](https://docs.joinpeertube.org/admin/following-instances#instances-redundancy) and cache servers
- **Storage:** [S3/Object storage](https://docs.joinpeertube.org/maintain/remote-storage)
- **Transcoding:** [Remote transcoding workers](https://docs.joinpeertube.org/admin/remote-runners)

---

## Release Policy

- ~4 major releases per year
- No LTS version - only latest stable is supported
- [Nightly builds](https://builds.joinpeertube.org/nightly/) available for testing
- Federation compatibility maintained with latest minor version only

---

## GDPR Compatibility

Core PeerTube is GDPR compatible:
- P2P can be disabled by default
- No personal data sent to third parties (with P2P disabled)
- Users can delete accounts
- Data export/import (PeerTube >= 6.1)
- [Privacy guide](https://docs.joinpeertube.org/admin/privacy-guide) for admins

---

## Moderation History

Every release since PeerTube 1.0 (October 2018) has added moderation features:

- **1.1:** Bulk actions, instance/account muting
- **1.2:** Unfederate on blacklist, notification system
- **1.3:** Auto-quarantine for untrusted users, follower management
- **1.4:** Plugin system for custom moderation rules
- **2.0:** Moderation policy setup, auto-follow public indexes
- **2.1:** Internal privacy mode, registration plugins (geoblocking, captchas)
- **2.2:** Improved abuse management, moderation hooks/helpers
- **2.3:** Predefined report reasons, bulk comment deletion
- **2.4:** Account/comment reporting, messaging system
- **3.0:** Dedicated comment management view

---

## Developer Features

### REST API
- [Getting started guide](https://docs.joinpeertube.org/api/rest-getting-started)
- [API documentation](https://docs.joinpeertube.org/api/rest-reference.html)

### Player Embed API
- Embed PeerTube player on external websites
- [Embed API documentation](https://docs.joinpeertube.org/api/embed-player)

### Plugins & Themes
- [Plugin development guide](https://docs.joinpeertube.org/contribute/plugins)
- [Plugin/theme API documentation](https://docs.joinpeertube.org/api/plugins)

---

## Technology Choices

**Why ActivityPub?** W3C recommended federation protocol, also used by Mastodon.

**Why not IPFS?** Still too young for large file streaming. PeerTube's HTTP/WebRTC-based P2P and redundancy system are more mature and maintainable.

**Why not blockchain-based solutions (DTube/Steemit)?** PeerTube doesn't impose any monetization model - this is a deliberate choice to remain neutral.

---

## Important Notes

### Content Control
Framasoft (PeerTube developers) does NOT control content on PeerTube instances. Each instance administrator is responsible for their own moderation and legal compliance.

Framasoft only hosts:
- [framatube.org](https://framatube.org)
- [peer.tube](https://peer.tube)
- [sepiasearch.org](https://sepiasearch.org) (search index)

### Trademark
"PeerTube" is a registered trademark held by Framasoft. Create your own identity for your instance rather than using "PeerTube" in your domain/project name.

### License
PeerTube is licensed under AGPL-3.0.

---

## Resources

- **Official Website:** [joinpeertube.org](https://joinpeertube.org)
- **Documentation:** [docs.joinpeertube.org](https://docs.joinpeertube.org)
- **Installation Guide:** [docs.joinpeertube.org/install/any-os](https://docs.joinpeertube.org/install/any-os)
- **Forum:** [framacolibri.org/c/peertube](https://framacolibri.org/c/peertube)
- **GitHub:** [github.com/Chocobozzz/PeerTube](https://github.com/Chocobozzz/PeerTube)
- **Security Policy:** [github.com/Chocobozzz/PeerTube/security](https://github.com/Chocobozzz/PeerTube/security)
