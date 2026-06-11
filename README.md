# 🌤️ Daily Weather Report Automation

![n8n](https://img.shields.io/badge/n8n-automation-%23EA664E?style=for-the-badge&logo=n8n&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-database-%233FCF8E?style=for-the-badge&logo=supabase&logoColor=white)
![OpenWeather](https://img.shields.io/badge/OpenWeather-API-%23EB6E4B?style=for-the-badge&logo=openweathermap&logoColor=white)
![Gmail](https://img.shields.io/badge/Gmail-email-%23EA4335?style=for-the-badge&logo=gmail&logoColor=white)
![Status](https://img.shields.io/badge/status-active-success?style=for-the-badge)

> **A fully automated weather reporting system** that fetches real-time weather data for multiple users and delivers personalized HTML email reports — twice daily, every day. Built entirely with **n8n** (no-code workflow automation) and powered by **Supabase**, **OpenWeather API**, and **Gmail**.

---

## 📋 Table of Contents

- [Features](#-features)
- [Workflow Overview](#-workflow-overview)
- [Architecture](#-architecture)
- [Technologies Used](#-technologies-used)
- [Database Schema](#-database-schema)
- [Installation & Setup](#-installation--setup)
- [Environment Variables & Credentials](#-environment-variables--credentials)
- [Importing the Workflow into n8n](#-importing-the-workflow-into-n8n)
- [Example Email Output](#-example-email-output)
- [Future Improvements](#-future-improvements)
- [Screenshots](#-screenshots)
- [Learning Outcomes](#-learning-outcomes)
- [License](#-license)

---

## ✨ Features

| Feature | Description |
|---|---|
| ⏰ **Scheduled Execution** | Runs automatically at **7:00 AM** and **7:00 PM** daily — never miss a report |
| 👥 **Multi-User Support** | Fetches all users from a Supabase `Users` table and generates individual reports |
| 🌡️ **Real-Time Weather Data** | Pulls live weather data from the **OpenWeather API** (temperature, conditions, humidity) |
| 📬 **Personalized HTML Emails** | Sends beautifully styled, responsive HTML emails with each user's name and local weather |
| 🗄️ **Data Persistence** | Stores every weather report in `weather_reports` and logs all notifications in `notification_logs` |
| 🔄 **Batch Processing** | Loops through users efficiently using n8n's split-in-batches node |
| 📊 **Full Audit Trail** | Every email sent is logged with a timestamp and status for complete traceability |

---

## 🔄 Workflow Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│  1. Schedule Trigger (7:00 AM / 7:00 PM)                           │
│     └─► 2. Fetch Users from Supabase (Users table)                 │
│            └─► 3. Loop Over Each User (Split in Batches)           │
│                   ├─► 4. Fetch Weather from OpenWeather API        │
│                   ├─► 5. Store Report in weather_reports table     │
│                   ├─► 6. Send HTML Email via Gmail                 │
│                   └─► 7. Log Notification in notification_logs     │
└─────────────────────────────────────────────────────────────────────┘
```

### Step-by-Step Breakdown

1. **Schedule Trigger** — An n8n Schedule Trigger node activates the workflow at **7:00 AM** and **7:00 PM** every day.
2. **Get Users** — Queries the `Users` table in Supabase to retrieve all registered users (name, email, location).
3. **Loop Over Items** — Iterates through each user using n8n's `SplitInBatches` node for controlled batch processing.
4. **Fetch Weather** — Makes an HTTP GET request to the OpenWeather API with the user's location to get current weather data (temperature, description, humidity).
5. **Create Weather Report** — Inserts a new record into the `weather_reports` table in Supabase with the fetched data.
6. **Send Email** — Composes and sends a personalized, responsive HTML email to the user via the **Gmail** node.
7. **Log Notification** — Records the delivery in the `notification_logs` table for auditing and tracking.

---

## 🏗️ Architecture

```
                    ┌──────────────┐
                    │   n8n (Self-  │
                    │  Hosted)      │
                    └──────┬───────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
   ┌──────────┐    ┌──────────────┐   ┌────────┐
   │ Supabase │    │ OpenWeather  │   │ Gmail  │
   │ (DB)     │    │ API          │   │ (SMTP) │
   └──────────┘    └──────────────┘   └────────┘
```

| Component | Role |
|---|---|
| **n8n** | Workflow automation engine — orchestrates the entire pipeline |
| **Supabase** | PostgreSQL database — stores users, weather reports, and notification logs |
| **OpenWeather API** | External weather data provider — supplies real-time temperature, humidity, and conditions |
| **Gmail API** | Email delivery service — sends personalized HTML reports to users |

---

## 🛠️ Technologies Used

| Technology | Purpose | Version |
|---|---|---|
| [n8n](https://n8n.io/) | Workflow automation & orchestration | Latest stable |
| [Supabase](https://supabase.com/) | PostgreSQL database & REST API | Latest |
| [OpenWeather API](https://openweathermap.org/api) | Real-time weather data | 2.5 (Current Weather) |
| [Gmail API](https://developers.google.com/gmail/api) | Email sending via OAuth 2.0 | v1 |

---

## 🗄️ Database Schema

### 📋 `Users` Table

Stores user profile and location data.

| Column | Type | Description |
|---|---|---|
| `id` | `uuid` / `int8` (PK) | Unique user identifier |
| `name` | `text` | User's full name (used in email greeting) |
| `email` | `text` | User's email address (report recipient) |
| `location` | `text` | City name (used for weather lookup) |

### 🌡️ `weather_reports` Table

Stores weather data fetched for each user.

| Column | Type | Description |
|---|---|---|
| `id` | `uuid` / `int8` (PK) | Unique report identifier |
| `user_id` | `uuid` / `int8` (FK → Users) | Associated user |
| `location` | `text` | City the weather was fetched for |
| `temperature` | `float8` | Current temperature in °C |
| `condition` | `text` | Weather condition description (e.g., "clear sky") |
| `humidity` | `int4` | Humidity percentage |
| `report_date` | `date` | Date the report was generated |

### 📬 `notification_logs` Table

Tracks email delivery status.

| Column | Type | Description |
|---|---|---|
| `id` | `uuid` / `int8` (PK) | Unique log identifier |
| `user_id` | `uuid` / `int8` (FK → Users) | Associated user |
| `sent_at` | `timestamptz` | Timestamp of when the email was sent |
| `status` | `text` | Delivery status (e.g., `sent`, `failed`) |

---

## 🚀 Installation & Setup

### Prerequisites

- A self-hosted **n8n** instance (or n8n Cloud account)
- A **Supabase** project (free tier works)
- An **OpenWeather API** key (free tier: 60 calls/minute)
- A **Google Cloud** project with Gmail API enabled

### Step 1: Set Up Supabase

1. Create a [Supabase](https://supabase.com/) project.
2. In the **SQL Editor**, run the following SQL to create the required tables:

```sql
-- Users table
CREATE TABLE users (
  id BIGSERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  email TEXT NOT NULL UNIQUE,
  location TEXT NOT NULL
);

-- Weather reports table
CREATE TABLE weather_reports (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT REFERENCES users(id),
  location TEXT NOT NULL,
  temperature FLOAT8 NOT NULL,
  condition TEXT NOT NULL,
  humidity INT4 NOT NULL,
  report_date DATE NOT NULL
);

-- Notification logs table
CREATE TABLE notification_logs (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT REFERENCES users(id),
  sent_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  status TEXT NOT NULL DEFAULT 'sent'
);
```

3. Insert sample users:

```sql
INSERT INTO users (name, email, location) VALUES
  ('Alice Johnson', 'alice@example.com', 'London'),
  ('Bob Smith', 'bob@example.com', 'Tokyo'),
  ('Carol Davis', 'carol@example.com', 'New York');
```

4. Copy your **Supabase URL** and **anon / service_role API key** from the project settings.

### Step 2: Get an OpenWeather API Key

1. Sign up at [OpenWeatherMap](https://openweathermap.org/api).
2. Subscribe to the **Current Weather Data** (free tier).
3. Copy your **API key** from the dashboard.

### Step 3: Set Up Gmail API

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Create a new project (or select existing).
3. Enable the **Gmail API**.
4. Configure the **OAuth consent screen** (External or Internal).
5. Create **OAuth 2.0 credentials** (Desktop app type).
6. Download the client credentials JSON file.

### Step 4: Configure n8n

1. Open your n8n instance.
2. Go to **Settings > Credentials** and set up the following:

---

## 🔐 Environment Variables & Credentials

### n8n Credentials

You'll need to configure three credential types in n8n:

| Credential | Type | Fields Required |
|---|---|---|
| **Supabase API** | Supabase | Supabase URL, Service Role API Key |
| **OpenWeather** | Header Auth | No credentials needed (API key is in the URL) |
| **Gmail OAuth2** | Gmail OAuth2 | Client ID, Client Secret, Refresh Token |

### API Key Configuration

The OpenWeather API key is embedded directly in the HTTP Request node's URL:

```
https://api.openweathermap.org/data/2.5/weather?q={{ $json.location }}&appid=YOUR_API_KEY&units=metric
```

> ⚠️ **Security Note:** For production, consider using n8n's credential system instead of hardcoding the API key. You can store it as a credential and reference it in the URL.

---

## 📥 Importing the Workflow into n8n

1. Download the `Daily Weather Report Automation.json` file.
2. Open your n8n instance.
3. Click **Workflows** in the left sidebar.
4. Click the **Import** button (or use **Ctrl/Cmd + I**).
5. Select the JSON file.
6. Once imported, update the following node settings:
   - **Supabase nodes** — Connect your Supabase credential.
   - **Gmail node** — Connect your Gmail OAuth2 credential.
   - **HTTP Request node** — Replace `YOUR_OPENWEATHER_API_KEY` with your actual API key.
7. Click **Save** and then **Active** to enable the workflow.

The workflow will now run automatically at **7:00 AM** and **7:00 PM** daily! 🎉

---

## 📧 Example Email Output

The email is a fully responsive HTML template with a clean, modern design. Here's what recipients see:

```
┌─────────────────────────────────────┐
│          DAILY WEATHER REPORT        │
│            London, UK                │
│         Monday, June 11, 2026        │
├─────────────────────────────────────┤
│                                     │
│   Good morning, Alice! Here's your  │
│   weather update for today.         │
│                                     │
│   ┌────────┐  ┌────────┐  ┌───────┐ │
│   │  22°C  │  │  Clear │  │  45%  │ │
│   │  Temp  │  │ Cond.  │  │ Humid │ │
│   └────────┘  └────────┘  └───────┘ │
│                                     │
│   ┌─────────────────────────────────┐│
│   │  ☀️ Enjoy the sunshine!         ││
│   └─────────────────────────────────┘│
│                                     │
│   Sent by Daily Weather Report Bot  │
└─────────────────────────────────────┘
```

The actual email uses a blue-themed header, card-style metric layout, and a motivational footer tailored to the weather condition.

---

## 🔮 Future Improvements

- [ ] **🌦️ Weather Forecasts** — Include 5-day or hourly forecasts alongside current conditions.
- [ ] **🌍 Multi-Language Support** — Send reports in the user's preferred language.
- [ ] **📱 SMS Alerts** — Add Twilio integration for SMS weather alerts.
- [ ] **🎨 Weather Icons** — Include dynamic weather icons based on conditions.
- [ ] **📊 Weekly Digest** — Send a weekly summary of weather trends.
- [ ] **⚙️ User Preferences** — Let users choose report frequency, temperature units (°C/°F), and notification channels.
- [ ] **🚨 Severe Weather Warnings** — Integrate alerts for extreme weather conditions.
- [ ] **📈 Dashboard** — Build a web dashboard (e.g., with Streamlit or Retool) for admins to view reports and logs.

---

## 📸 Screenshots

> *Screenshots coming soon! Here's what will be included:*
>
> - The complete n8n workflow editor view
> - A sample HTML email as rendered in Gmail
> - The Supabase table dashboard with populated data
> - Workflow execution history in n8n

---

## 🎓 Learning Outcomes

This project demonstrates proficiency in:

| Skill | Details |
|---|---|
| **No-Code Automation** | Building complex, production-grade workflows with **n8n** |
| **Database Design** | Structuring relational tables (`users`, `weather_reports`, `notification_logs`) with foreign keys |
| **API Integration** | Consuming a third-party REST API (OpenWeather) and parsing JSON responses |
| **Email Templating** | Crafting responsive **HTML email templates** with inline CSS |
| **OAuth 2.0 Authentication** | Configuring Gmail API with OAuth 2.0 credentials |
| **Batch Processing** | Iterating over datasets with controlled batch loops in n8n |
| **Scheduling & Automation** | Setting up cron-based triggers for hands-free daily execution |
| **Data Persistence & Logging** | Storing transactional data and maintaining audit trails |

---

## 📄 License

This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.

---

<p align="center">
  Made with ❤️ using <a href="https://n8n.io/">n8n</a> • <a href="https://supabase.com/">Supabase</a> • <a href="https://openweathermap.org/api">OpenWeather</a> • <a href="https://mail.google.com/">Gmail</a>
</p>
<p align="center">
  ⭐ If you found this project useful, consider giving it a star!
</p>
