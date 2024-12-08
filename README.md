# VPS Setup and Configuration Guides

This guide provides step-by-step instructions on setting up and deploying a Next.js app on a VPS, configuring a PostgreSQL database, connecting a custom domain, setting up SSL, and managing your data using tools like Prisma and pgAdmin.

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
   bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install -y git curl build-essential
2. **Install Node.js: Use the NodeSource setup script to install Node.js:**:
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
   sudo apt install -y nodejs
   ```
3. **Verify Installation:**
   ```bash
    node -v
    npm -v
   ```
4. **Install PM2:**
   ```bash
   npm install -g pm2
   ```

## Step 2: Upload Your Next.js App to the VPS

1. clone your Next.js app repository to your VPS:

   ```bash
   git clone <YOUR_REPO_URL> /var/www/your-app

   cd /var/www/your-app
   ```

2. Install dependencies:

   ```bash
   npm install
   ```

3. Build the app:

   ```bash
   npm run build
   ```

## Step 3: Configure PM2 to Run Your App

1. Start your Next.js app with PM2:

   ```bash
   pm2 start npm --name "next-app" -- start
   ```

2. Set Up PM2 to Start on Boot:

   ````bash
   pm2 startup
   pm2 save
   3. Check App Status:

   ```bash
   pm2 status
   ````

## Step 4: Configure a Reverse Proxy (Optional but Recommended)

To make your app accessible on port 80 or 443 (HTTP/HTTPS), configure a reverse proxy using Nginx:

1. Install Nginx:

   ```bash
   sudo apt install -y nginx
   ```

2. Create a new Nginx configuration file:

   ```bash
   sudo nano /etc/nginx/sites-available/your-app
   ```

3. Add the following configuration:

   ```nginx
   server {
       listen 80;
       server_name your-domain.com;

       location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection 'upgrade';
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
       }
   }
   ```

4. Enable the site and restart Nginx:

   ```bash
   sudo ln -s /etc/nginx/sites-available/your-app /etc/nginx/sites-enabled/

   sudo nginx -t

   sudo systemctl restart nginx
   ```

## Step 5: Test Your Deployment

1. Open your browser and navigate to your domain (e.g., `http://your-domain.com`).
2. Your Next.js app should be running successfully on your VPS.
3. Check logs if there are issues:

   ```bash
   pm2 logs next-app
   ```

---

# 2. Setting Up CI/CD for Your Next.js App

This guide explains how to set up Continuous Integration and Continuous Deployment (CI/CD) for a Next.js app using GitHub Actions. We'll automate the process of deploying your app to a Virtual Private Server (VPS) whenever changes are pushed to the main branch.

---

## Prerequisites

Before starting, ensure the following:

1. **VPS**: A Virtual Private Server with an appropriate OS (e.g., Ubuntu).
2. **GitHub Repository**: Your Next.js app should be stored in a GitHub repository.
3. **SSH Key**: SSH access to your VPS, with the private key stored as a GitHub secret (`VM_SSH_PRIVATE_KEY`).
4. **PM2**: PM2 installed on your VPS for process management.
5. **Node Version Manager (NVM)**: A tool to manage different versions of Node.js on the VPS.

---

## Step 1: Create GitHub Secrets

In your GitHub repository, navigate to **Settings > Secrets and variables > Actions** and add the following secrets:

- `VM_HOST`: The IP address/host name of your VPS.
- `VM_USER`: The username for SSH access (e.g., `root` or `ubuntu`).
- `VM_SSH_PRIVATE_KEY`: The private SSH key used to access the VPS (ensure this is stored securely).

**Tips**:

- If you haven't generate ssh keys, do the following:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

- Ensure the public key (id_rsa.pub) is added to the ~/.ssh/authorized_keys file on your VM:

```bash
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

- Verify SSH access with:

```bash
ssh -i ~/.ssh/id_rsa your_user@your_vm_ip
```

- Update GitHub Secrets:
- Copy the private key:

```bash
cat ~/.ssh/id_rsa
```

- Add it as VM_SSH_PRIVATE_KEY in your GitHub repository's secrets.

---

## Step 2: Set Up the GitHub Actions Workflow

Create a `.github/workflows/deploy.yml` file in your GitHub repository with the following content:

```yaml
name: Deploy Next.js App

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      # Checkout the repository
      - name: Checkout code
        uses: actions/checkout@v4

      # You can perform other jobs here like testing, linting, etc.

      # Set up SSH deployment
      - name: Deploy to VPS
        uses: appleboy/ssh-action@v0.1.6
        with:
          host: ${{ secrets.VM_HOST }}
          username: ${{ secrets.VM_USER }}
          key: ${{ secrets.VM_SSH_PRIVATE_KEY }}
          script: |
            # Load Node Version Manager (NVM)
            export NVM_DIR="$HOME/.nvm"
            [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

            # Use the correct Node.js version (ensure your VPS has NVM installed)
            nvm use 20 || nvm install 20

            # Install PM2 globally if not installed
            npm install -g pm2

            # Navigate to the project directory
            cd /var/www/your-nextjs-app

            # Pull the latest changes from the main branch
            git pull origin main

            # Install dependencies using npm
            npm ci

            # Build the application
            npm run build

            # Restart the app with PM2 (or start if not running)
            npx pm2 restart your-app-name || npx pm2 start npm --name "your-app-name" -- start

            # Save PM2 process list to auto-start on boot
            npx pm2 save
```

---

# 3. Connecting a Custom Domain to Your Next.js App

After deploying your Next.js app to a VPS, you might want to use a custom domain for your application. Here’s how you can connect a custom domain to your Next.js app hosted on a VPS.

---

## Step 1: Purchase a Domain

First, you'll need to purchase a domain name. You can get a domain from registrars like:

- [Namecheap](https://www.namecheap.com)
- [GoDaddy](https://www.godaddy.com)
- [Google Domains](https://domains.google)
- [Hostinger](https://www.hostinger.com)

Once you have your domain, you can proceed to configure the DNS settings.

---

## Step 2: Configure DNS Records

After logging in to your domain registrar's dashboard, you need to configure the DNS settings to point to your VPS's IP address.

1. **A Record**: Set an A record for your domain (e.g., `www.yourdomain.com`) to point to the IP address of your VPS.

| Type | Name | Value       | TTL  |
| ---- | ---- | ----------- | ---- |
| A    | @    | your_vps_ip | 3600 |
| A    | www  | your_vps_ip | 3600 |

2. **CNAME Record**: If you want to use a subdomain (e.g., `app.yourdomain.com`), set a CNAME record to point to your domain.

| Type  | Name | Value          | TTL  |
| ----- | ---- | -------------- | ---- |
| CNAME | @    | yourdomain.com | 3600 |
| CNAME | www  | yourdomain.com | 3600 |

## Step 3: Configure Nginx or Apache (Web Server)

Now that the domain points to your server, configure your web server (e.g., **Nginx** or **Apache**) to serve your Next.js app.

### For Nginx

1. Install Nginx on your VPS (if not already installed):

```bash
sudo apt update
sudo apt install -y nginx
```

2. Create an Nginx configuration file for your domain:

```bash
sudo nano /etc/nginx/sites-available/yourdomain.com
```

3. Add the following content to the file:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;  # Port where Next.js is running
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

```

4. Enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
```

5. Test the Nginx configuration and restart:

```bash
sudo nginx -t

sudo systemctl restart nginx
```

Your custom domain should now point to your Next.js app.

---

# 4. Setting Up SSL for Your App

To ensure that your application is secure and accessible via HTTPS, you need to set up SSL. We will use Let’s Encrypt, a free SSL certificate provider.

## Step 1: Install Certbot

Install Certbot on your VPS to obtain and renew SSL certificates automatically:

```bash
sudo apt update

sudo apt install certbot python3-certbot-nginx
```

## Resolving ACME Challenge Issues for SSL Certificate Generation

When you're setting up SSL certificates with Let’s Encrypt using Certbot, one common issue that may occur is the ACME challenge error. This challenge is part of the process used by Certbot to verify that you own the domain for which you're requesting an SSL certificate.

If Certbot is unable to verify the domain, it could be due to access restrictions on the .well-known/acme-challenge/ directory, which is used for verification. To resolve this issue, you need to ensure that Nginx serves this directory properly, and that your server is configured to handle the challenge correctly.

## Steps You Took to Resolve the ACME Challenge

1. **Create the ACME Challenge Directory:** First, you created the directory where the ACME challenge files will be stored during the verification process.

```bash
    sudo mkdir -p /var/www/your-app/public/.well-known/acme-challenge
```

2. **Set Proper Permissions:** You then ensured that the Nginx server (www-data user) has the right permissions to access the challenge directory.

```bash
sudo chown -R www-data:www-data /var/www/printitug/public/.well-known
sudo chmod -R 755 /var/www/printitug/public/.well-known
```

3. **Run Certbot to Obtain the SSL Certificate:** Once the above configurations are in place, you can now run the Certbot command to request an SSL certificate:

```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

With this setup, Certbot will complete the ACME challenge and automatically configure Nginx with SSL.

---

Below is how the Full Nginx configuration should look like:

```nginx
server {
  server_name yourdomain.com www.yourdomain.com;

  location /.well-known/acme-challenge/ {
      root /var/www/your-app/public;
      allow all;
  }

  location / {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot


}

server {
    if ($host = www.yourdomain.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    if ($host = yourdomain.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


  listen 80;
  server_name yourdomain.com www.yourdomain.com;
    return 404; # managed by Certbot
}
```

---

# 5. Setting Up PostgreSQL on Your VPS

This guide walks you through the process of installing and setting up PostgreSQL on your VPS to use with your Next.js application. PostgreSQL is a powerful, open-source relational database management system (RDBMS) often used for web applications due to its performance and robustness.

## Step 1: Update Your Server

Before installing any software, make sure your server’s package list is up-to-date.

```bash
sudo apt update
sudo apt upgrade -y
```

## Step 2: Install PostgreSQL

1. Install PostgreSQL using the following command:

```bash
sudo apt install postgresql postgresql-contrib -y
```

- `postgresql`: Installs the core PostgreSQL database.
  `postgresql-contrib`: Installs additional utilities and extensions.

2. After installation, the PostgreSQL service will start automatically. You can verify that it is running with:

```bash
sudo systemctl status postgresql
```

The output should show that PostgreSQL is active and running.

## Step 3: Switch to the PostgreSQL User

PostgreSQL uses its own user (postgres) to manage the database. Switch to this user to interact with the database.

```bash
sudo -i -u postgres
```

## Step 4: Access PostgreSQL

tart the PostgreSQL interactive terminal (psql) to interact with your database.

```bash
psql
```

You should now be in the PostgreSQL prompt where you can run SQL queries.

## Step 5: Create a New Database and User

**1. Create a new database:**

```sql
CREATE DATABASE yourdbname;
```

Replace `yourdbname` with the name you want for your database.

**2. Create a new user:**

```sql
CREATE USER yourdbuser WITH ENCRYPTED PASSWORD 'yourpassword';
```

Replace `yourdbuser` with the username you want and `yourpassword` with a strong password.

**3. Grant privileges to the user:**

```sql
GRANT ALL PRIVILEGES ON DATABASE yourdbname TO yourdbuser;
```

This gives the user full privileges over the database.

**4. Exit the PostgreSQL prompt:**

```sql
\q
```

ou are now back to the shell prompt.

## Step 6: Allow Remote Connections (Optional)

If you want your PostgreSQL database to be accessible from external machines (like your Next.js app), you'll need to configure PostgreSQL to listen for remote connections.

1. Edit the PostgreSQL configuration file:

```bash
sudo nano /etc/postgresql/12/main/postgresql.conf
```

Replace `12` with your PostgreSQL version.

Find the `listen_addresses` setting and change it to allow connections from any IP address:

```bash
listen_addresses = '*'
```

2. Edit `pg_hba.conf` to allow remote connections:

Open the `pg_hba.conf` file:

```bash
sudo nano /etc/postgresql/12/main/pg_hba.conf
```

Replace `12` with your PostgreSQL version.

Add the following line at the end of the file, which allows connections from any IP:

```bash
host    all             all             0.0.0.0/0            md5
```

Alternatively, you can restrict access to specific IP ranges for added security.

3. Restart PostgreSQL for the changes to take effect:

```bash
sudo systemctl restart postgresql
```

## Step 7: Connect to PostgreSQL from Your Next.js App (For this Case Prisma)

To interact with PostgreSQL in your Next.js app, we'll use Prisma, which provides a modern ORM (Object-Relational Mapping) for Node.js.

**1. Install Prisma:**
Start by installing Prisma and the PostgreSQL driver in your Next.js application:

```bash
npm install prisma @prisma/client
```

**2. Initialize Prisma**
Initialize Prisma in your Next.js app by running the following command:

```bash
npx prisma init
```

This will create a new directory called `prisma` with the necessary configuration files. (`.env`, `schema.prisma`)

**3. Configure the Database Connection**
In the `prisma/schema.prisma` file, configure the `datasource` block to use PostgreSQL. Replace the `DATABASE_URL` in the .env file with your actual database connection string.

In your `.env` file, add the connection details for your PostgreSQL database:

```bash
DATABASE_URL="postgresql://yourdbuser:yourpassword@your_vps_ip:5432/yourdbname?schema=public"
```

- Replace `yourdbuser`, `yourpassword`, `your_vps_ip`, and `yourdbname` with the respective values.
- You can also adjust the `?schema=public` parameter depending on the schema you're using.

**4. Define Your Prisma Model**

In `prisma/schema.prisma`, define your models based on the structure of your PostgreSQL database.

For example:

```prisma
model User {
  id    Int    @id @default(autoincrement())
  name  String
  email String @unique
  age   Int?
}
```

**5. Run Prisma Migrations**
After defining your models, run the following command to generate the migrations and apply them to your PostgreSQL database:

```bash
npx prisma migrate dev --name init
```

OR

```bash
npx prisma db push
```

**6. Generate Prisma Client**

```bash
npx prisma generate
```

You can now start using prisma in your application.

## Step 8: View Your Data with pgAdmin

pgAdmin is a graphical user interface (GUI) for managing PostgreSQL databases. You can use it to easily view, manage, and interact with your database.

### 1. Install pgAdmin

To use pgAdmin on your local machine or VPS:

- Download and install pgAdmin from the official website: https://www.pgadmin.org/download/

### 2. Connect pgAdmin to PostgreSQL

Once you’ve installed pgAdmin, you need to connect it to your PostgreSQL database.

- Open pgAdmin and log in (use the credentials you created for your PostgreSQL installation).

- Click on "Add New Server" in the **pgAdmin dashboard**.

- In the Create - Server dialog, configure the following:

  - **General Tab:** Name your connection (e.g., `My VPS Database`).

  - Connection Tab:

    - **Host name/address:** Enter the IP address of your VPS (e.g., your_vps_ip).
    - **Port:** Default PostgreSQL port is `5432`.
    - **Maintenance database:** Set this to `postgres` (default database).
    - **Username:** Enter your PostgreSQL user (e.g., `yourdbuser`).
    - **Password:** Enter the password you set for `yourdbuser`.

- **Save:** Click “Save” to connect to your PostgreSQL instance.

## 3. Browse Data in pgAdmin

Once connected, you can:

- Browse and manage databases, tables, and other objects.
- Use the Query Tool to run SQL queries directly on your database.
- View your tables and data, perform CRUD operations (Create, Read, Update, Delete), and manage database settings.

For example, to view data in a table, you can:

- In the left panel, expand your connected server and navigate to the desired database.
- Expand **Schemas** > **public** > **Tables**.
- Right-click on a table and select **View/Edit** Data > **All Rows**.
- This will display the data in the selected table and allow you to perform various operations on it.

## Step 9: Backup and Restore PostgreSQL Data

### 1. Backup Your Database

You can create a backup of your PostgreSQL database using the pg_dump command. This is helpful for migrating `data` or creating backups for disaster recovery.

```bash
pg_dump yourdbname > /path/to/backup/yourdbname_backup.sql
```

- Replace `yourdbname` with your database name and provide a valid path for the backup file.

### 2. Restore Your Database

If you need to restore a database backup, use the `psql` command:

```bash
psql yourdbname < /path/to/backup/yourdbname_backup.sql
```
