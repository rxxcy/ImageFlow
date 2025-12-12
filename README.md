# ImageFlow
<div align="center">

[![English Document](https://img.shields.io/badge/-English%20Document-6366f1?logo=readthedocs&style=flat-square&logoColor=white)](README.md)
|
[![部署说明](https://img.shields.io/badge/-%E9%83%A8%E7%BD%B2%E8%AF%B4%E6%98%8E-6366f1?logo=docker&style=flat-square&logoColor=white)](https://catcat.blog/imageflow-install.html)
|
[![贡献指南](https://img.shields.io/badge/-%E8%B4%A1%E7%8C%AE%E6%8C%87%E5%8D%97-6366f1?logo=github&style=flat-square&logoColor=white)](contributing.md)
|
[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/Yuri-NagaSaki/ImageFlow)
</div>

[English](README.md) | [中文文档](README_CN.md)

ImageFlow is a full-stack image management platform that automatically optimizes images for different devices and browsers, while providing powerful filtering and distribution capabilities.

## ✨ Features

### 🚀 **Core Capabilities**
- **Smart Image Conversion**: Automatic WebP/AVIF generation with libvips for optimal performance
- **Device-Aware Serving**: Intelligent orientation detection (portrait for mobile, landscape for desktop)
- **Advanced Random API**: Multi-tag filtering, exclusion rules, and format preferences
- **Dual Storage Support**: Local filesystem or S3-compatible storage
- **Real-time Processing**: Background worker pool for async image conversion

### 🎯 **Advanced Filtering**
- **Multi-tag Combinations**: AND logic for precise content selection
- **Exclusion Filters**: Prevent NSFW or private content from public APIs
- **Orientation Control**: Force landscape/portrait regardless of device
- **Format Negotiation**: Client-aware format selection (AVIF > WebP > Original)

### 🛡️ **Security & Privacy**
- **API Key Authentication**: Secure upload and management endpoints
- **Smart Defaults**: Auto-exclude sensitive content from random API
- **Expiry Management**: Automatic cleanup of expired images
- **Metadata Protection**: Redis-based metadata with file fallback

### 🎨 **Modern Frontend**
- **Next.js 14**: App Router with TypeScript and Tailwind CSS
- **Drag & Drop**: Intuitive file upload with batch processing
- **Dark Mode**: Beautiful UI that adapts to user preferences
- **Responsive Design**: Works perfectly on all device sizes

## 🏃‍♂️ Quick Start

### Using Docker (Recommended)

```bash
# Clone the repository
git clone https://github.com/Yuri-NagaSaki/ImageFlow.git
cd ImageFlow

# Start with Docker Compose
docker-compose up -d

# Your ImageFlow instance is now running at http://localhost:8080
```

### Manual Installation

#### Prerequisites
- **Go 1.22+**
- **Node.js 18+**
- **libvips** (for image processing)
- **Redis** (optional but recommended)

#### Backend Setup

```bash
# Install Go dependencies
go mod tidy

# Configure environment
cp .env.example .env
# Edit .env with your settings

# Build and run
go build -o imageflow
./imageflow
```

#### Frontend Setup

```bash
cd frontend

# Install dependencies
npm install

# Development
npm run dev

# Production build
npm run build
```

## 🔧 Configuration

Create a `.env` file in the project root:

```bash
# API Keys
API_KEY=your_api_key_here

# Storage Configuration
STORAGE_TYPE=s3 # Options: local, s3
METADATA_STORE_TYPE=redis
LOCAL_STORAGE_PATH=static/images

# Redis Configuration
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_DB=0
REDIS_TLS_ENABLED=false

# S3 Configuration
S3_ENDPOINT=
S3_REGION=
S3_ACCESS_KEY=
S3_SECRET_KEY=
S3_BUCKET=
CUSTOM_DOMAIN=

# Upload and Conversion Settings
# Maximum number of images allowed in a single upload (Need Self build default: 20)
# Image quality for WebP/AVIF conversion (1-100, default: 80)
# Number of parallel worker threads for batch processing (default: 4)
# SPEED: Range: 0-8, 0=slowest/highest quality, 8=fastest/lowest quality
# Size of worker pool for concurrent image processing (default: 4)
MAX_UPLOAD_COUNT=20
IMAGE_QUALITY=75
WORKER_THREADS=4
SPEED=5
WORKER_POOL_SIZE=4

# Frontend Configuration Only for Docker
# if you just want export static site, you can set below to empty
# NEXT_PUBLIC_API_URL=http://localhost:8686

NEXT_PUBLIC_API_URL=

# Remote patterns for Next.js Image component (comma-separated)
# Examples:
# Single domain: example.com
# Multiple domains: example.com,cdn.example.com
# With protocol: https://example.com,http://cdn.example.com
NEXT_PUBLIC_REMOTE_PATTERNS=

# Debug Mode
DEBUG_MODE=false
```

## 📚 API Usage

### Random Image API

The crown jewel of ImageFlow - get perfectly filtered random images:

```bash
# Basic random image
GET /api/random?tag=nature

# Advanced filtering
GET /api/random?tags=nature,landscape&exclude=nsfw&orientation=landscape&format=webp

# Mobile-optimized
GET /api/random?tag=wallpaper&orientation=portrait
```

### Upload API

```bash
curl -X POST "https://your-domain.com/api/upload" \
  -H "Authorization: Bearer your-api-key" \
  -F "images[]=@photo1.jpg" \
  -F "images[]=@photo2.png" \
  -F "tags=nature,landscape" \
  -F "expiryMinutes=1440"
```

### Management API

```bash
# List images with filtering
GET /api/images?page=1&tag=nature&orientation=landscape

# Delete image
POST /api/delete-image
Content-Type: application/json
{"id": "image-uuid"}

# Get all tags
GET /api/tags
```

For complete API documentation, see [API_USAGE_GUIDE.md](API_USAGE_GUIDE.md).

## 🏗️ Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Next.js 14   │    │    Go Backend    │    │  Storage Layer  │
│                 │    │                  │    │                 │
│ • TypeScript    │◄──►│ • Fiber HTTP     │◄──►│ • Local Files   │
│ • Tailwind CSS  │    │ • libvips        │    │ • S3 Compatible │
│ • App Router    │    │ • Worker Pool    │    │ • Redis Cache   │
│ • Static Export │    │ • Auto Formats   │    │ • Metadata      │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

### Key Components

- **Image Processor**: libvips-powered conversion engine
- **Worker Pool**: Async processing for optimal performance  
- **Metadata Manager**: Redis + file-based dual storage
- **Smart Router**: Device-aware content delivery
- **Security Layer**: API key auth + intelligent filtering

## 🚀 Deployment

### Single Container (Recommended)

```bash
docker-compose up -d
```

### Separated Services

```bash
# Frontend and backend as separate containers
docker-compose -f docker-compose-separate.yaml up -d
```

## 🔨 Development

### Project Structure

```
ImageFlow/
├── main.go                 # Application entry point
├── config/                 # Configuration management
├── handlers/               # HTTP request handlers
│   ├── random.go          # Advanced random image API
│   ├── upload.go          # Multi-file upload handler
│   └── *.go               # Other API endpoints
├── utils/                  # Core utilities
│   ├── converter_bimg.go  # libvips image processing
│   ├── redis.go           # Metadata and caching
│   ├── worker_pool.go     # Async processing
│   └── *.go               # Storage, auth, etc.
├── frontend/              # Next.js application
│   ├── app/               # App Router structure
│   │   ├── components/    # React components
│   │   ├── hooks/         # Custom hooks
│   │   └── utils/         # Frontend utilities
│   └── package.json       # Dependencies
├── static/                # Generated assets
└── docker-compose*.yaml   # Deployment configs
```

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **libvips** - High-performance image processing
- **Redis** - Lightning-fast metadata storage
- **Next.js** - Amazing React framework
- **Fiber** - Express-inspired Go web framework
- **Tailwind CSS** - Utility-first CSS framework

## 📞 Support

- 📖 [Documentation](API_USAGE_GUIDE.md)
- 🐛 [Report Issues](https://github.com/Yuri-NagaSaki/ImageFlow/issues)
- 💬 [Discussions](https://github.com/Yuri-NagaSaki/ImageFlow/discussions)

---

**Made with ❤️ by the catcat.blog team**

*Transform your image workflow today!*
