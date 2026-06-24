<div align="center">

<img src="./.github/assets/images/logo.svg" width="200px" alt="Timeful logo" />

[![License: AGPL v3](https://img.shields.io/badge/License-AGPL_v3-orange.svg?style=for-the-badge)](https://www.gnu.org/licenses/agpl-3.0)
[![Go](https://img.shields.io/badge/Go-1.21+-00ADD8?style=for-the-badge&logo=go&logoColor=white)](https://go.dev)
[![Vue](https://img.shields.io/badge/Vue_2-4FC08D?style=for-the-badge&logo=vue.js&logoColor=white)](https://vuejs.org)
[![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)](https://www.mongodb.com)

**繁體中文** | [English](README-en.md)

</div>

## 總覽

Timeful 是一個開源的**團體時間配對工具**，幫助你快速找到一群人都有空的時段

建立活動、分享連結，每個人標記自己的空檔，Timeful 自動算出最多人有空的時間。不需要登入，填完即走

線上體驗：https://timeful.ntust.org

## 功能

### 📅 **時間配對**
- 建立活動，設定日期與時段範圍
- 分享連結，所有人直接在網頁上標記空檔
- 即時顯示重疊結果，一眼找出最佳時間
- 支援「有空」與「如果需要」兩種標記
- 可篩選特定成員查看重疊情況

### 📋 **雙欄回覆列表**
- 桌面版使用多欄排列，一眼看到所有人的狀態

### 🌐 **繁體中文介面**
- 全站中文化，包含建立活動、填寫空檔、查看結果、設定等
- 預設活動標題與描述為中文

### 🌍 **跨時區支援**
- 自動偵測使用者時區
- 不同時區的人填寫同一份活動也不會錯位

### 🧹 **乾淨無廣告**
- 沒有廣告、沒有彈窗
- 不需要第三方登入，填完即走

### 🔧 **適合自架**
- Docker Compose 一鍵部署
- OTP 驗證信支援自訂 SMTP
- 免費帳號可建立 100 個活動
- Self-host 可直接開啟 Premium（無限活動）
- 頭像支援 [Gravatar](https://gravatar.com/)

### 📦 **其他**
- 活動複製
- Email 通知與提醒
- 匯出 CSV
- 可設定僅活動建立者能查看回覆
- 瀏覽器外掛 API（[文件](./PLUGIN_API_README.md)）

## Self-Hosting

請參考 [Deployment Guide](./DEPLOYMENT.md)，內含 Docker Compose 與 NixOS 的設定說明

### SMTP 環境變數

| 變數 | 說明 |
|------|------|
| `SMTP_HOST` | SMTP 伺服器位址 |
| `SMTP_PORT` | SMTP 連接埠（預設 587） |
| `SMTP_USERNAME` | SMTP 帳號 |
| `SMTP_PASSWORD` | SMTP 密碼 |
| `SMTP_FROM_ADDRESS` | 寄件人地址 |
| `SMTP_FROM_NAME` | 寄件人顯示名稱 |

完整環境變數清單請參考 `server/.env.template`

## 授權

本專案採用 [GNU Affero General Public License v3.0](LICENSE) 授權
