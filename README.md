Go Order Tracker — 訂單管理與即時狀態追蹤系統

A Go-based order management and tracking application built with Gin, GORM, SQLite and Server-Sent Events (SSE).

本專案以 Go 開發後端訂單管理系統，涵蓋訂單建立、狀態追蹤、管理員身分驗證及事件通知，並透過模組化設計分離 HTTP 請求處理與資料存取邏輯。

Tech Stack

Category| Technology
Backend| Go 1.25, Gin
Database| SQLite
ORM| GORM
Authentication| Session-based Authentication, bcrypt
Middleware| Gin Middleware, gin-contrib/sessions
Validation| go-playground/validator
Real-time Communication| Server-Sent Events (SSE)
Logging| log/slog
Frontend| Go HTML Templates, React (experimental)

Core Features

1. Order Management

- 建立訂單及多筆訂單明細
- 使用 Short ID 產生訂單識別碼
- 透過訂單 ID 查詢訂單資訊
- 管理員查看、更新及刪除訂單
- 依建立時間排序訂單紀錄
- 使用 GORM Preload 載入訂單與明細關聯資料

2. Order Status Tracking

支援訂單生命週期管理，將訂單狀態呈現於顧客追蹤頁面。

目前程式碼定義的狀態包含：

1. 已成功下單
2. 製作中
3. 已完成（待交付）
4. 交付失敗／逾期
5. 已交付（完成交貨）

狀態透過資料庫保存，由管理員進行更新。

3. Real-time Event Notifications

使用 SSE（Server-Sent Events）建立伺服器向客戶端推送事件的通訊機制。

- 客戶端訂閱訂單相關事件
- 建立新訂單後發送通知
- 管理員可透過通知端點接收新訂單事件
- 以 NotificationManager 管理事件發布

4. Authentication & Authorisation

採用 Session-based Authentication，搭配 Gin Middleware 保護管理員功能。

- 使用 bcrypt 進行密碼雜湊與驗證
- 使用 GORM-backed Session Store 保存登入狀態
- 使用 HttpOnly Cookie 降低 Session 被 JavaScript 直接存取的風險
- Middleware 驗證 Session 與資料庫使用者
- 未通過驗證的請求無法繼續執行受保護 Handler

5. Request Validation

透過 Gin Binding 與 go-playground/validator 驗證使用者輸入。

- 必填欄位驗證
- 字串長度限制
- 商品類型及規格驗證
- Slice 元素的遞迴驗證
- 拒絕不符合規格的表單請求

System Architecture

Client
  |
  v
Gin Router
  |
  v
Middleware
  |-- Session Authentication
  |-- Request Validation
  |
  v
Handler Layer
  |-- Order Handlers
  |-- Customer Handlers
  |-- Authentication Handlers
  |-- Notification Manager
  |
  v
Model / Data Access Layer
  |-- OrderModel
  |-- UserModel
  |
  v
GORM
  |
  v
SQLite

Architecture Design

Router Layer

負責定義 HTTP Endpoints，並將請求分派至對應的 Handler。

Handler Layer

負責 HTTP Request/Response、表單處理、驗證結果及業務流程協調。

透過 "NewHandler()" 集中初始化所需的 Model 與 NotificationManager。

Model Layer

封裝資料庫操作，避免在各個 Handler 中重複撰寫資料存取邏輯。

Database Layer

透過 GORM 管理 SQLite 資料庫，包含資料新增、查詢、更新、刪除與關聯資料載入。

Database Design

Order

保存訂單主要資訊：

- ID
- Status
- CustomerName
- Phone
- Address
- CreatedAt
- UpdatedAt

OrderItem

保存每筆訂單的商品明細：

- ID
- OrderID
- Size
- Pizza
- Instructions

User

保存管理員登入資訊：

- ID
- Username
- Password (bcrypt hash)

Relationships

"Order (1) → (N) OrderItem"

使用 GORM Foreign Key 建立一對多關聯，並透過 "Preload("Items")" 查詢訂單明細。

API Endpoints

Method| Endpoint| Description
GET| "/"| 顯示訂單建立頁面
POST| "/new-order"| 建立新訂單
GET| "/customer/:id"| 查詢指定訂單
GET| "/notifications"| 顧客端事件訂閱
GET| "/login"| 管理員登入頁面
POST| "/login"| 管理員登入
POST| "/logout"| 管理員登出
GET| "/admin"| 管理員訂單列表
POST| "/admin/order/:id/update"| 更新訂單狀態
POST| "/admin/order/:id/delete"| 刪除訂單
GET| "/admin/notifications"| 管理員新訂單通知
GET| "/api/admin/dashboard"| 管理員 Dashboard JSON API

管理員相關端點透過 Authentication Middleware 進行存取控制。

Project Structure

go-pizza-order-tracker/
├── cmd/
│   ├── main.go
│   ├── routes.go
│   ├── handler.go
│   ├── customer.go
│   ├── middleware.go
│   ├── utils.go
│   └── validators.go
├── internal/
│   └── models/
│       ├── order.go
│       └── user.go
├── frontend/
│   └── ...
├── templates/
│   └── ...
├── data/
├── go.mod
├── go.sum
└── README.md

Getting Started

Requirements

- Go 1.25 or later
- SQLite
- Node.js and npm (optional, for React frontend)

Run Backend

git clone https://github.com/vc428803/go-pizza-order-tracker.git

cd go-pizza-order-tracker

go mod download

設定 Session Secret：

export SESSION_SECRET_KEY="replace-with-a-strong-random-secret"

啟動後端：

go run ./cmd

預設服務位置：

"http://localhost:8080"

Available Pages

- Order Form: "http://localhost:8080/"
- Admin Login: "http://localhost:8080/login"
- Admin Dashboard: "http://localhost:8080/admin"

React Frontend (Experimental)

cd frontend
npm install
npm run dev

預設 Vite 開發伺服器：

"http://localhost:5173"

React 前端屬於延伸開發部分，主要功能展示以 Go HTML Templates 為主。

Engineering Highlights

Middleware-based Access Control

透過 Gin Route Group 集中套用驗證 Middleware，將身分驗證與訂單 Handler 分離，減少重複驗證邏輯。

Structured Data Access

使用獨立的 "OrderModel" 與 "UserModel" 封裝 GORM 操作，讓 Handler 聚焦於 HTTP 請求及應用流程。

Event-driven Communication

透過 NotificationManager 發布訂單事件，並以 SSE 提供伺服器向客戶端推送訊息的通道。

Database Relationships

使用 GORM Association、Foreign Key、Index 及 Model Hooks 管理訂單與明細資料。

Input Validation

在請求進入主要業務流程前，使用 Binding Tags 與自訂驗證規則檢查資料格式。

Project Focus

本專案主要呈現以下後端工程能力：

Go Backend Development · RESTful Routing · Middleware · Session Authentication · ORM · Relational Database Design · SSE · Request Validation

---

Repository: https://github.com/vc428803/go-pizza-order-tracker
