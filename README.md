# Strapi Setup on Ubuntu Server 22.04 (LXC)

This guide covers the complete process of setting up **Strapi in production** on **Ubuntu Server 22.04 inside an LXC container**, behind **NGINX with SSL (Let's Encrypt)**, and using **PM2 for process management**.

---

## **1. Install Required Dependencies**

Ensure the system is up to date:
```bash
sudo apt update && sudo apt upgrade -y
```

### **Install Node.js (LTS Recommended)**
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```
Verify:
```bash
node -v
npm -v
```

### **Install Yarn** (Optional but recommended)
```bash
sudo npm install -g yarn
```

### **Install PM2** (Process manager for Node.js)
```bash
sudo npm install -g pm2
```

Verify PM2 installation:
```bash
pm2 -v
```

---

## **2. Set Up Strapi Project**

Navigate to your desired project location:
```bash
cd /home/smm/Files/
```

Create a new Strapi project:
```bash
npx create-strapi-app strapi-project --quickstart
```

Move into your project folder:
```bash
cd strapi-project
```

---

## **3. Configure Strapi for Production**

Edit the **production config file**:
```bash
nano config/server.js
```

Modify it to match your production settings:
```javascript
module.exports = ({ env }) => ({
  host: env('HOST', '0.0.0.0'),
  port: env.int('PORT', 1337),
  url: 'https://yourdomain.com', // Your domain
  app: {
    keys: env.array('APP_KEYS'),
  },
  webhooks: {
    populateRelations: env.bool('WEBHOOKS_POPULATE_RELATIONS', false),
  },
});
```
Save and exit (`CTRL + X`, then `Y`, then `ENTER`).

### **Set Up Environment Variables**

Create a **.env** file:
```bash
nano .env
```

Add the following:
```env
HOST=0.0.0.0
PORT=1337
APP_KEYS=your_random_keys_here
NODE_ENV=production
DATABASE_CLIENT=sqlite
DATABASE_FILENAME=.tmp/data.db
JWT_SECRET=your_jwt_secret
ADMIN_JWT_SECRET=your_admin_jwt_secret
API_TOKEN_SALT=your_api_token_salt
```
Save and exit.

---

## **4. Build & Run Strapi with PM2**

### **Build the project:**
```bash
yarn build
```
If using npm:
```bash
npm run build
```

### **Start Strapi using PM2:**
```bash
pm2 start npm --name "strapi" -- run start
```

### **Ensure Strapi Starts on Boot**
```bash
pm2 save
pm2 startup
```
Run the command PM2 provides to enable auto-start.

### **Check PM2 Process List**
```bash
pm2 list
```

---

## **5. Configure NGINX as a Reverse Proxy**

Open your NGINX configuration:
```bash
sudo nano /etc/nginx/sites-available/strapi
```

Add the following:
```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:1337;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Save and exit.

Enable the configuration:
```bash
sudo ln -s /etc/nginx/sites-available/strapi /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## **6. Secure with SSL (Let's Encrypt)**

If you haven't already installed Certbot:
```bash
sudo apt install certbot python3-certbot-nginx -y
```

Request an SSL certificate:
```bash
sudo certbot --nginx -d yourdomain.com
```

Test renewal:
```bash
sudo certbot renew --dry-run
```

Set up auto-renewal:
```bash
sudo crontab -e
```
Add this line:
```
0 3 * * * certbot renew --quiet && systemctl restart nginx
```

---

## **7. Access Strapi Admin Panel**

Now, visit:
🔗 **https://yourdomain.com/admin**

Log in and start managing your content! 🚀

---

## **8. Useful PM2 Commands**

Restart Strapi:
```bash
pm2 restart strapi
```

Stop Strapi:
```bash
pm2 stop strapi
```

Check logs:
```bash
pm2 logs strapi
```

Monitor processes:
```bash
pm2 list
```

---

## **✅ Strapi is Now Running in Production!**

With this setup, you have:
- **Strapi running in production** on an Ubuntu LXC container
- **NGINX reverse proxy** handling requests
- **SSL certificate from Let's Encrypt**
- **PM2 managing the process** for reliability

Let me know if you need help with **database setup, API customization, or performance optimizations**! 🚀

