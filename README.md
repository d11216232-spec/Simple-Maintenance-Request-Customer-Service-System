# 設備報修與追蹤系統（Repair Ticket System）

## 📋 專案概述

這是一個**設備報修與追蹤系統**，用於幫助客戶提交維修請求，並讓管理人員能夠完整追蹤每個報修單的處理過程。

**核心特色**：完整的追蹤記錄機制，確保每次修改都有審計日誌。

---

## 👥 系統角色

| 角色 | 權限 | 功能 |
|-----|------|------|
| **Customer（用戶）** | 基礎 | 提交報修單、查看自己的單據、查看處理進度 |
| **IT Staff（IT人員）** | 進階 | 查看全部報修單、更新狀態、添加處理說明 |
| **Admin（管理員）** | 完全 | 用戶管理、數據統計、生成報表 |

---

## ✨ 系統功能規格

### 核心功能

1. **用戶認證系統**
   - 用戶登入 / 註冊
   - JWT token 管理
   - 角色權限驗證

2. **報修單管理**
   - 創建報修單（標題、描述、設備類型、優先級）
   - 查詢報修單列表（支援篩選、排序）
   - 查看單個報修單詳情
   - 更新報修單狀態

3. **追蹤記錄系統** ⭐ （核心亮點）
   - 自動記錄每次操作（誰、何時、做了什麼）
   - 查詢歷史記錄
   - 完整的審計日誌

4. **通知系統**
   - 狀態變更時自動通知用戶
   - Email 或 SMS 通知

5. **統計報表**
   - 按月統計報修數量
   - 統計完成率
   - 設備故障類型分析

6. **搜尋與篩選**
   - 按狀態篩選
   - 按日期篩選
   - 按優先級篩選

---

## 🗄️ 數據庫設計

### Users 表（用戶管理）

```sql
CREATE TABLE Users (
  user_id    INT PRIMARY KEY AUTO_INCREMENT,
  username   VARCHAR(50)  UNIQUE NOT NULL,
  password   VARCHAR(255) NOT NULL,
  email      VARCHAR(100),
  role       ENUM('customer', 'it_staff', 'admin') DEFAULT 'customer',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### Tickets 表（報修單）

```sql
CREATE TABLE Tickets (
  ticket_id    INT PRIMARY KEY AUTO_INCREMENT,
  title        VARCHAR(200) NOT NULL,
  description  TEXT,
  device_type  VARCHAR(100),
  priority     ENUM('low', 'medium', 'high', 'urgent') DEFAULT 'medium',
  status       ENUM('open', 'in_progress', 'resolved', 'closed') DEFAULT 'open',
  customer_id  INT NOT NULL,
  assignee_id  INT,
  created_at   DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at   DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (customer_id)  REFERENCES Users(user_id),
  FOREIGN KEY (assignee_id)  REFERENCES Users(user_id)
);
```

### TicketLogs 表（審計日誌）

```sql
CREATE TABLE TicketLogs (
  log_id      INT PRIMARY KEY AUTO_INCREMENT,
  ticket_id   INT NOT NULL,
  changed_by  INT NOT NULL,
  action      VARCHAR(100) NOT NULL,
  old_value   TEXT,
  new_value   TEXT,
  note        TEXT,
  created_at  DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (ticket_id)  REFERENCES Tickets(ticket_id),
  FOREIGN KEY (changed_by) REFERENCES Users(user_id)
);
```

---

## 🔌 API 端點規格

### 認證

| 方法 | 路徑 | 說明 |
|------|------|------|
| POST | `/api/auth/register` | 用戶註冊 |
| POST | `/api/auth/login`    | 用戶登入，回傳 JWT |
| POST | `/api/auth/logout`   | 登出 |

### 報修單

| 方法   | 路徑                  | 說明 | 權限 |
|--------|-----------------------|------|------|
| GET    | `/api/tickets`        | 取得報修單列表 | Customer（僅自己）/ IT Staff / Admin |
| POST   | `/api/tickets`        | 建立報修單 | Customer+ |
| GET    | `/api/tickets/:id`    | 取得單一報修單 | Customer（僅自己）/ IT Staff / Admin |
| PATCH  | `/api/tickets/:id`    | 更新報修單狀態 | IT Staff+ |
| DELETE | `/api/tickets/:id`    | 刪除報修單 | Admin |

### 追蹤記錄

| 方法 | 路徑                        | 說明 | 權限 |
|------|-----------------------------|------|------|
| GET  | `/api/tickets/:id/logs`     | 取得單一報修單的操作記錄 | IT Staff+ |
| GET  | `/api/logs`                 | 取得所有操作記錄 | Admin |

### 統計報表

| 方法 | 路徑                  | 說明 | 權限 |
|------|-----------------------|------|------|
| GET  | `/api/stats/monthly`  | 按月統計報修數量 | Admin |
| GET  | `/api/stats/rate`     | 完成率統計 | Admin |
| GET  | `/api/stats/devices`  | 設備故障類型分析 | Admin |

---

## 🚀 快速開始

### 環境需求

- Node.js 18+
- MySQL 8.0+
- npm 或 yarn

### 安裝步驟

```bash
# 複製專案
git clone https://github.com/d11216232-spec/Simple-Maintenance-Request-Customer-Service-System.git
cd Simple-Maintenance-Request-Customer-Service-System

# 安裝依賴
npm install

# 複製環境設定檔並填入設定
cp .env.example .env

# 初始化資料庫
npm run db:migrate

# 啟動開發伺服器
npm run dev
```

### 環境變數（.env）

```env
PORT=3000
DB_HOST=localhost
DB_PORT=3306
DB_NAME=repair_system
DB_USER=root
DB_PASS=your_password
JWT_SECRET=your_jwt_secret
JWT_EXPIRES_IN=7d
```

---

## 📁 專案結構

```
├── src/
│   ├── controllers/     # 請求處理器
│   ├── middlewares/     # JWT 驗證、角色授權
│   ├── models/          # 資料庫模型
│   ├── routes/          # API 路由定義
│   ├── services/        # 業務邏輯
│   └── utils/           # 工具函式（通知、統計等）
├── tests/               # 單元測試 / 整合測試
├── .env.example
├── package.json
└── README.md
```

---

## 🛡️ 安全設計

- 密碼使用 **bcrypt** 雜湊儲存
- API 以 **JWT Bearer Token** 進行認證
- 依角色實施 **RBAC**（Role-Based Access Control）
- 所有狀態變更自動寫入審計日誌

---

## 📄 授權

MIT License