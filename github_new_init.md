# НОВЫЙ репозиторий

Пошаговый план:
1. Подготовка проекта

```bash
cd /opt/digital_signage_system

# Создайте .gitignore
cat > .gitignore << 'EOF'
# Dependencies
node_modules/
.pnpm-store/

# Build
dist/
build/

# Environment
.env
.env.local
.env.production

# Logs
*.log
npm-debug.log*

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/

# Uploads
public/uploads/*
!public/uploads/.gitkeep

# Database
*.sql.backup
EOF

# Создайте пустой файл чтобы папка uploads сохранилась
mkdir -p public/uploads
touch public/uploads/.gitkeep
```
2. Создайте README.md
```bash
cat > README.md << 'EOF'
# Digital Signage System

Modern digital signage management system built with React, tRPC, and MySQL.

## Features

- 📺 Display device management
- 🎬 Playlist creation with media & text overlays
- 📊 Real-time statistics & monitoring
- 🔄 Auto-refresh playlists
- 📱 Mobile-responsive admin panel
- 🎨 Smooth slide transitions
- 💓 Device heartbeat tracking

## Tech Stack

- **Frontend:** React 18, TypeScript, Tailwind CSS, Shadcn/ui
- **Backend:** Node.js, tRPC, Drizzle ORM
- **Database:** MySQL
- **Build:** Vite, pnpm

## Installation

### Prerequisites

- Node.js 20+
- MySQL 8+
- pnpm

### Setup

1. Clone repository:
```bash
git clone https://github.com/YOUR_USERNAME/digital-signage-system.git
cd digital-signage-system
```

2. Install dependencies:
```bash
pnpm install
```

3. Configure database:
```bash
mysql -u root -p
CREATE DATABASE digital_signage;
CREATE USER 'signage'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON digital_signage.* TO 'signage'@'localhost';
FLUSH PRIVILEGES;
```

4. Copy environment file:
```bash
cp .env.example .env
```

5. Edit `.env` with your settings:
```env
DATABASE_URL="mysql://signage:your_password@localhost:3306/digital_signage"
JWT_SECRET=your-secret-key
OWNER_OPEN_ID=local-user-1
OWNER_NAME=Admin
```

6. Run database migrations:
```bash
mysql -u signage -p digital_signage < drizzle/schema.sql
mysql -u signage -p digital_signage < drizzle/add_device_logs.sql
```

7. Build and start:
```bash
pnpm build
pnpm start
```

### Development
```bash
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000)

### Usage

#### Create Playlist
1. Go to Playlists → New Playlist
2. Add media files (images/videos)
3. Add text overlays
4. Configure display duration

#### Setup Display
1. Go to Devices → Add Device
2. Assign playlist to device
3. Open player URL on display device
4. View statistics in Device Details

### Player URL Format
```
http://your-server:3000/player?device=DEVICE_ID&playlist=PLAYLIST_ID
```

Debug mode:
```
http://your-server:3000/player?device=DEVICE_ID&debug=true
```

### Systemd Service (Production)
```bash
sudo nano /etc/systemd/system/digital-signage.service
```
```ini
[Unit]
Description=Digital Signage System
After=network.target mysql.service

[Service]
Type=simple
User=your_user
WorkingDirectory=/path/to/digital_signage_system
Environment="NODE_ENV=production"
ExecStart=/usr/bin/pnpm start
Restart=always

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl enable digital-signage.service
sudo systemctl start digital-signage.service
```

### License

MIT

### Version

v1.1 - December 2025
EOF
3. Создайте .env.example
bashcat > .env.example << 'EOF'
## Database
DATABASE_URL="mysql://signage:password@localhost:3306/digital_signage"

## JWT
JWT_SECRET=your-secret-key-here

## Owner
OWNER_OPEN_ID=local-user-1
OWNER_NAME=Admin
OWNER_EMAIL=admin@localhost

## OAuth (optional, can be disabled)
DISABLE_OAUTH=true
OAUTH_SERVER_URL=http://localhost:3000

## Application
VITE_APP_TITLE=Digital Signage System
VITE_APP_LOGO=/logo.png
NODE_ENV=production
EOF
4. Создайте репозиторий на GitHub

Откройте https://github.com/new
Repository name: digital-signage-system
Description: "Modern digital signage management system"
Public или Private (выбирайте)
НЕ добавляйте README, .gitignore, license (у вас уже есть)
Create repository

5. Загрузите код
bashcd /opt/digital_signage_system

## Инициализируйте git
git init

## Добавьте все файлы
git add .

## Первый коммит
git commit -m "Initial commit - v1.1

Features:
- Display device management
- Playlist creation with media & text
- Real-time statistics
- Smooth slide transitions
- Mobile responsive
- Device heartbeat tracking
- OAuth bypass for local development"

## Подключите к GitHub (замените YOUR_USERNAME)
git remote add origin https://github.com/YOUR_USERNAME/digital-signage-system.git

## Отправьте код
git branch -M main
git push -u origin main

## Создайте тег версии
git tag -a v1.1 -m "Version 1.1 - Production ready"
git push origin v1.1
6. Настройте GitHub репозиторий
После загрузки:

Добавьте Topics (Settings → Topics):

digital-signage
react
typescript
trpc
mysql


Создайте Releases (Releases → Create new release):

Tag: v1.1
Title: Version 1.1 - Production Ready
Description: список фич


Добавьте LICENSE (Create new file → LICENSE):

MIT License (рекомендую)




Дополнительно:
Создайте CHANGELOG.md
markdown# Changelog

### [1.1] - 2025-12-10

#### Added
- Smooth slide transitions for media
- Device heartbeat monitoring
- Device statistics dashboard
- Mobile-responsive interface
- Real-time playlist updates
- Text overlay positioning (top/center/bottom)
- Media upload with drag & drop
- OAuth bypass for local development

#### Fixed
- Display duration field editing
- Image loading transitions
- Text fade animations

### [1.0] - 2025-12-01

#### Added
- Initial release
- Basic playlist management
- Device management
- Media upload
