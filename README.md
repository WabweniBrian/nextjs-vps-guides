
# 1. Deploying a Next.js 14 App to a VPS

This guide walks you through deploying a Next.js 14 application to a Virtual Private Server (VPS). We will cover environment setup, application deployment, and running the app in production mode.

---

## Prerequisites

Before you begin, ensure you have:
- A VPS with a supported Linux distribution (e.g., Ubuntu 20.04 or later).
- Node.js (v20 or higher) installed on your VPS.
- PM2 installed globally on your VPS (`npm install -g pm2`).
- A Next.js 14 app ready to deploy.

---

## Step 1: Prepare Your VPS

1. **Update and Install Dependencies**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install -y git curl build-essential
