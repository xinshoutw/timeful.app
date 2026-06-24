<div align="center">

<img src="./.github/assets/images/logo.svg" width="200px" alt="Timeful logo" />

[![License: AGPL v3](https://img.shields.io/badge/License-AGPL_v3-orange.svg?style=for-the-badge)](https://www.gnu.org/licenses/agpl-3.0)
[![Go](https://img.shields.io/badge/Go-1.21+-00ADD8?style=for-the-badge&logo=go&logoColor=white)](https://go.dev)
[![Vue](https://img.shields.io/badge/Vue_2-4FC08D?style=for-the-badge&logo=vue.js&logoColor=white)](https://vuejs.org)
[![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)](https://www.mongodb.com)

[繁體中文](README.md) | **English**

</div>

## Overview

Timeful is an open-source **group availability finder** that helps you find the best time for everyone to meet.

Create an event, share the link, and let everyone mark their available times. Timeful instantly shows when the most people overlap. No login required — fill in and go.

Live demo: https://timeful.ntust.org

## Features

### 📅 **Availability Matching**
- Create events with custom date and time ranges
- Share a link — everyone marks availability directly in the browser
- Real-time overlap visualization to find the best meeting time
- "Available" vs. "If needed" availability types
- Filter by specific members to check subset availability

### 📋 **Two-Column Respondents**
- Desktop uses a multi-column grid so you can see everyone's status at a glance
- Column-major reading order — top to bottom feels natural

### 🌐 **Traditional Chinese UI**
- Full interface localized to Traditional Chinese
- Default event title and description in Chinese

### 🌍 **Timezone Support**
- Automatic timezone detection
- People in different timezones filling the same event stay aligned

### 🧹 **Clean & Ad-Free**
- No ads, no popups
- No third-party login required — just fill in and go

### 🔧 **Self-Host Friendly**
- One-command deploy with Docker Compose
- OTP verification emails via configurable SMTP
- Free accounts can create up to 100 events
- Self-hosters can enable Premium directly (unlimited events)
- Avatar fallback via [Gravatar](https://gravatar.com/)

### 📦 **More**
- Duplicate events
- Email notifications and reminders
- CSV export
- Option to restrict responses to event creator only
- Browser extension API ([docs](./PLUGIN_API_README.md))

## Self-Hosting

See the [Deployment Guide](./DEPLOYMENT.md) for Docker Compose and NixOS setup instructions.

### SMTP Environment Variables

| Variable | Description |
|----------|-------------|
| `SMTP_HOST` | SMTP server address |
| `SMTP_PORT` | SMTP port (default 587) |
| `SMTP_USERNAME` | SMTP username |
| `SMTP_PASSWORD` | SMTP password |
| `SMTP_FROM_ADDRESS` | Sender email address |
| `SMTP_FROM_NAME` | Sender display name |

See `server/.env.template` for the full list of environment variables.

## License

This project is licensed under [GNU Affero General Public License v3.0](LICENSE).
