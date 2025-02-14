# MKE/MKI Pre-Authorisation Backend

## Overview
The **MKE/MKI Pre-Authorisation Backend** is a Node.js-based payment pre-authorisation service that interfaces with a front-end payment form. It supports currency conversion and integrates with a payment gateway (e.g., Stripe) to handle transactions securely.

## Features
- Secure pre-authorisation of transactions
- Supports **USD, EUR, GHS** currency conversion
- Validates **email, amount, currency, card number, expiry month/year, and pre-authorisation code**
- Uses **Express.js** as a backend framework
- Implements **CORS** for secure cross-origin requests
- **PM2 process management** for stability
- **CI/CD setup** for automated deployments on **Hostinger VPS**

## Tech Stack
- **Node.js** (Backend framework)
- **Express.js** (API development)
- **Stripe API** (Payment gateway integration)
- **MongoDB** (Database for storing transactions - optional)
- **Docker** (Containerized deployment - optional)
- **PM2** (Process management)
- **Nginx** (Reverse proxy)
- **CI/CD** (GitHub Actions for automated deployment)

---

## Getting Started

### Prerequisites
Ensure you have the following installed:
- [Node.js](https://nodejs.org/)
- [npm](https://www.npmjs.com/)
- [PM2](https://pm2.keymetrics.io/)
- [MongoDB](https://www.mongodb.com/) (Optional)

### Installation
1. **Clone the repository**
   ```bash
   git clone https://github.com/your-repo.git
   cd mke-payment-backend
   ```
2. **Install dependencies**
   ```bash
   npm install
   ```
3. **Create a `.env` file**
   ```bash
   nano .env
   ```
   **Add the following environment variables:**
   ```ini
   PORT=5000
   STRIPE_SECRET_KEY=your-stripe-secret-key
   BASE_CURRENCY=USD
   ```

4. **Run the server**
   ```bash
   npm start
   ```
   OR using PM2 (recommended):
   ```bash
   pm2 start server.js --name "mke-payment-backend"
   ```

5. **Access API**
   ```bash
   curl -X POST http://localhost:5000/payment -d '{"amount": "100", "currency": "USD"}'
   ```

---

## Deployment Guide (Hostinger VPS)

### **Step 1: SSH into the VPS**
```bash
ssh your-user@your-vps-ip
```

### **Step 2: Setup Project Folder**
```bash
cd /var/www/
mkdir mke-payment-backend && cd mke-payment-backend
```

### **Step 3: Clone and Install**
```bash
git clone https://github.com/your-repo.git .
npm install
```

### **Step 4: Configure PM2 & Nginx**
```bash
pm install -g pm2
pm2 start server.js --name "mke-payment-backend"
pm2 save
pm2 startup
```
```bash
sudo nano /etc/nginx/sites-available/mke-payment
```
_Add the following:_
```nginx
server {
    listen 80;
    server_name your-domain.com;
    location / {
        proxy_pass http://localhost:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
```bash
sudo ln -s /etc/nginx/sites-available/mke-payment /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### **Step 5: Setup CI/CD (GitHub Actions)**
1. Create `.github/workflows/deploy.yml`
```yaml
name: Deploy to Hostinger VPS

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Deploy to VPS
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            cd /var/www/mke-payment-backend
            git pull origin main
            npm install
            pm2 restart mke-payment-backend
```
2. **Add Secrets in GitHub:**
   - `VPS_HOST` (Your VPS IP)
   - `VPS_USER` (Your VPS username)
   - `VPS_SSH_KEY` (Your private SSH key)

---

## API Endpoints

### **1. Pre-Authorise Payment**
```http
POST /payment
```
**Request Body:**
```json
{
  "email": "user@example.com",
  "amount": 100,
  "currency": "USD",
  "card_number": "4242424242424242",
  "expiry_month": "12",
  "expiry_year": "2025",
  "pre_auth_code": "123456"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Pre-authorisation successful",
  "transaction_id": "txn_123456"
}
```

### **2. Get Transaction Status**
```http
GET /payment/:transaction_id
```
**Response:**
```json
{
  "transaction_id": "txn_123456",
  "status": "pending"
}
```

---

## **Troubleshooting**

1. **Check PM2 Logs:**
   ```bash
   pm2 logs mke-payment-backend
   ```
2. **Restart PM2 App:**
   ```bash
   pm2 restart mke-payment-backend
   ```
3. **Check Nginx Status:**
   ```bash
   sudo systemctl status nginx
   ```

---

## **Contributing**
Feel free to contribute by creating a pull request or reporting issues.

---

## **License**
This project is licensed under the MIT License.

---

## **Author**
Developed by **Felix J. Y. Asubonteng Dwomoh** | Erkenntnis Technologies.

