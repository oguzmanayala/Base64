# Base64 Team Onboarding Guide

Welcome to **Base64** - Chicago's #1 AI Automation for Small Businesses. This guide will help you get set up with all the services and tools we use.

---

## About Base64

**Domain:** chicagomarketing.ai
**Focus:** AI Marketing Automation + Answer Engine Optimization (AEO)
**Target Market:** Small businesses in Chicago & Chicagoland area

### What We Do

- AI-powered marketing automation for small businesses
- Answer Engine Optimization (AEO) to improve visibility in AI search results
- 7-day setup with a money-back guarantee
- Proven results: 150% more leads for our clients

### Our Clients

We serve 50+ Chicago small businesses including wellness studios, restaurants, tech startups, and local service providers.

---

## Service Overview

| Service | Purpose | Access URL |
|---------|---------|------------|
| **n8n** | Workflow automation & orchestration | http://localhost:5678 (Docker) |
| **Supabase** | Database for clients, leads & tasks | https://supabase.com/dashboard |
| **Portal** | Internal dashboard | http://localhost:3000/portal |
| **GitHub** | Source code repository | (your org URL) |

---

## 1. n8n (Workflow Automation)

n8n is our workflow automation hub. We use it to automate client onboarding, lead nurturing, notifications, and integrations.

### What You'll Use It For

- Managing automated workflows for clients
- Monitoring workflow executions
- Setting up integrations (webhooks, APIs, email)
- Managing environment variables for workflows

### Accessing n8n

n8n runs locally via Docker. Once you have Docker set up:

**URL:** http://localhost:5678

You should have received an invite to join as a user. Your role will be one of:
- **Owner** - Full admin access
- **Admin** - Can manage workflows and users
- **Member** - Can view and execute workflows

### n8n Setup (Docker)

1. **Install Docker Desktop**
   - Windows: https://docs.docker.com/desktop/install/windows-install/
   - Mac: https://docs.docker.com/desktop/install/mac-install/

2. **Pull and run n8n**
   ```bash
	   docker run -it --rm --name n8n -p 5678:5678 -e GENERIC_TIMEZONE="America/Chicago" -e TZ="America/Chicago" -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true -e N8N_RUNNERS_ENABLED=true -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
   ```

3. **Accept Your Invite**
   - Check your email for the n8n user invite
   - Click the link to set up your account
   - Log in at http://localhost:5678

### Key n8n Features in Portal

Our portal (`/portal/workflows`) provides a management interface for:
- Viewing all workflows and their status (active/paused)
- Monitoring execution history and success rates
- Retrying failed executions
- Managing environment variables

---

## 2. Supabase (Database)

Supabase is our PostgreSQL database with a nice UI and API layer.

### What We Store

| Table | Purpose |
|-------|---------|
| `clients` | Business client information (name, website, status, config) |
| `leads` | Sales pipeline (name, email, status, source) |
| `tasks` | Client task tracking |

### Lead Pipeline Stages

```
lead → scheduled → first_offer_sent → negotiation_offer_sent → deal_complete → deal_ended
```

### Accessing Supabase

1. **Accept Your Invite**
   - Check your email for the Supabase project invite
   - Create an account or log in at https://supabase.com

2. **Access the Dashboard**
   - Go to https://supabase.com/dashboard
   - Select the Base64 project
   - You can view/edit data in the Table Editor

### Key Supabase Features

- **Table Editor** - View and edit client/lead data
- **SQL Editor** - Run custom queries
- **Auth** - Manage user authentication
- **API Docs** - Auto-generated API documentation

---

## 3. Development Environment Setup

### Prerequisites

- **Node.js** 18+ (LTS recommended)
- **npm** or **pnpm**
- **Docker Desktop** (for n8n)
- **Git**
- **VS Code** (recommended)

### Clone & Install

```bash
# Clone the repository
git clone <repository-url>
cd nuxt-app

# Install dependencies
npm install

# Copy environment file
cp .env.example .env
```

### Configure Environment Variables

Edit `.env` with the following values (ask team lead for actual values):

```bash
# Supabase
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_KEY=your-anon-key

# n8n API (for portal integration)
N8N_API_URL=http://localhost:5678/api/v1
N8N_API_KEY=your-n8n-api-key
N8N_URL=http://localhost:5678

# Webhooks (optional for local dev)
WEBHOOK_URL=
WEBHOOK_SECRET=

# Site URL
NUXT_PUBLIC_SITE_URL=http://localhost:3000
```

### Getting Your n8n API Key

1. Log into n8n at http://localhost:5678
2. Click your profile icon (bottom left)
3. Go to **Settings** → **API**
4. Create a new API key
5. Copy and add to your `.env` file

### Run the Development Server

```bash
npm run dev
```

The app will be available at:
- **Landing Page:** http://localhost:3000
- **Portal Dashboard:** http://localhost:3000/portal

---

## 4. Portal Overview

The portal is our internal dashboard for managing the business.

### Portal Pages

| Page | URL | Purpose |
|------|-----|---------|
| Dashboard | `/portal` | Overview stats, recent activity |
| Clients | `/portal/clients` | Manage client accounts |
| Leads | `/portal/leads` | Sales pipeline management |
| Workflows | `/portal/workflows` | n8n workflow management |
| Executions | `/portal/executions` | Workflow execution logs |
| Users | `/portal/users` | n8n user management |
| Variables | `/portal/variables` | Workflow environment variables |

### Dashboard Stats

The dashboard shows real-time metrics:
- n8n connection status
- Active client count
- Total leads in pipeline
- Active workflows
- Workflow success rate

---

## 5. Project Structure

```
nuxt-app/
├── app/
│   ├── pages/           # Routes
│   │   ├── index.vue    # Landing page
│   │   └── portal/      # Admin portal pages
│   ├── components/      # Vue components
│   │   ├── landing/     # Marketing page components
│   │   ├── forms/       # Form components
│   │   ├── ui/          # Reusable UI
│   │   └── portal/      # Portal components
│   ├── layouts/         # Page layouts
│   ├── types/           # TypeScript definitions
│   └── composables/     # Vue composables
├── server/
│   └── api/
│       ├── contact.post.ts  # Form submissions
│       └── n8n/             # n8n API routes
├── content/             # Markdown content pages
├── docs/                # Documentation
└── public/              # Static assets
```

---

## 6. Common Commands

```bash
# Development
npm run dev          # Start dev server (http://localhost:3000)

# Building
npm run build        # Build for production
npm run generate     # Generate static site
npm run preview      # Preview production build

# Code Quality
npm run lint         # Check for linting errors
npm run lint:fix     # Auto-fix linting issues
```

---

## 7. Useful Links

### Documentation
- [Nuxt 3 Docs](https://nuxt.com/docs)
- [Vue 3 Docs](https://vuejs.org/guide/introduction.html)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [Supabase Docs](https://supabase.com/docs)
- [n8n Docs](https://docs.n8n.io/)

### Tools
- [Supabase Dashboard](https://supabase.com/dashboard)
- [n8n (Local)](http://localhost:5678)

---

## 8. Getting Help

1. **Check existing documentation** in `/docs`
2. **Ask in team chat** for quick questions
3. **Create a GitHub issue** for bugs or feature requests

---

## Quick Start Checklist

- [ ] Accept Supabase project invite
- [ ] Accept n8n user invite
- [ ] Install Docker Desktop
- [ ] Start n8n container
- [ ] Clone repository
- [ ] Install dependencies (`npm install`)
- [ ] Configure `.env` file
- [ ] Get n8n API key and add to `.env`
- [ ] Run dev server (`npm run dev`)
- [ ] Access portal at http://localhost:3000/portal
- [ ] Verify n8n connection shows "Connected" on dashboard

---

Welcome to the team!
