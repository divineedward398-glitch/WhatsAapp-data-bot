# 📱 WhatsApp Data Bot

A full-stack WhatsApp data-selling bot system for Nigeria with an admin dashboard. Users buy MTN, Airtel, Glo, and 9mobile data via WhatsApp; payments go through Paystack directly to your account.

---

## 🗂️ Project Structure

```
whatsapp-data-bot/
├── backend/
│   ├── server.js              ← Main Express server
│   ├── db.js                  ← JSON file database
│   ├── utils.js               ← Helper functions
│   ├── data.json              ← Auto-created: stores orders & settings
│   ├── controllers/
│   │   ├── botController.js   ← WhatsApp conversation logic
│   │   ├── whatsappController.js ← WhatsApp Cloud API
│   │   ├── paymentController.js  ← Paystack integration
│   │   └── adminController.js    ← Admin login & stats
│   ├── routes/
│   │   ├── whatsapp.js
│   │   ├── payment.js
│   │   ├── admin.js
│   │   └── orders.js
│   └── middleware/
│       └── auth.js            ← JWT token verification
├── frontend/
│   └── pages/
│       ├── login.html         ← Admin login page
│       └── dashboard.html     ← Admin panel
├── .env.example               ← Copy to .env and fill in
├── package.json
└── README.md
```

---

## 🚀 Quick Start

### 1. Install dependencies
```bash
npm install
```

### 2. Set up environment variables
```bash
cp .env.example .env
# Then edit .env with your real values
```

### 3. Start the server
```bash
# Development (auto-restarts)
npm run dev

# Production
npm start
```

Your server runs at: `http://localhost:3000`
Admin panel: `http://localhost:3000/pages/dashboard.html`

---

## ⚙️ Configuration

### WhatsApp Cloud API (Meta)
1. Go to [developers.facebook.com](https://developers.facebook.com)
2. Create an app → Add "WhatsApp" product
3. Get your **Phone Number ID** and **Access Token**
4. Set your **Webhook URL**: `https://yourdomain.com/webhook/whatsapp`
5. Set **Verify Token**: match what you put in `.env` as `WHATSAPP_VERIFY_TOKEN`
6. Subscribe to the `messages` webhook field

### Paystack
1. Go to [dashboard.paystack.com](https://dashboard.paystack.com)
2. Settings → API Keys & Webhooks
3. Copy your **Secret Key** → paste in `.env` as `PAYSTACK_SECRET_KEY`
4. Add Webhook URL: `https://yourdomain.com/payment/webhook`

### Deploying (Render, Railway, etc.)
1. Push code to GitHub
2. Create a new Web Service on [render.com](https://render.com) (free tier)
3. Set all environment variables in the dashboard
4. Deploy — your app gets a public URL for webhooks

---

## 💬 WhatsApp Bot Flow

```
User sends "Hi"
  → Shows main menu
    1. Buy Data → Select Network → Select Plan → Enter Phone → Confirm → Payment Link
    2. Check Prices → Shows price list
    3. Contact Support → Shows support number
```

The bot tracks conversation state per user. Users can type **menu** anytime to restart.

---

## 🔌 Plugging in a Real VTU API

In `backend/controllers/paymentController.js`, find the `simulateDataDelivery` function and replace the simulation with a real API call:

```js
// Example: VTpass
async function simulateDataDelivery(order) {
  const response = await fetch('https://vtpass.com/api/pay', {
    method: 'POST',
    headers: { 'api-key': process.env.VTPASS_API_KEY, ... },
    body: JSON.stringify({
      request_id: order.id,
      serviceID: networkToServiceId(order.network),
      billersCode: order.recipientPhone,
      variation_code: planToVariation(order.plan),
      amount: order.amount,
      phone: order.buyerPhone,
    })
  });
  // Handle response ...
}
```

VTU API providers to look at:
- [VTpass](https://vtpass.com/documentation)
- [Clubkonnect](https://www.clubkonnect.com/apidoc)
- [Datasubly](https://datasubly.com/api-docs)

---

## 🛡️ Security

- All payment webhooks verified with HMAC-SHA512 signature
- Admin routes protected by JWT token (expires in 12 hours)
- Input validation on all phone numbers and prices
- API keys stored in environment variables only

---

## 📊 Admin Dashboard Features

| Feature | Description |
|---------|-------------|
| Stats | Total earnings, order counts by status |
| Orders table | View all orders with filters (pending/paid/delivered) |
| Mark Delivered | Manually confirm data delivery |
| Price settings | Update prices per network and plan |
| Auto-refresh | Dashboard updates every 30 seconds |

---

## 🔧 Customization

**Add more plans:** Edit `DEFAULT_DB.settings.prices` in `db.js`

**Change bot messages:** Edit `botController.js` — all messages are plain strings

**Add more networks:** Add to the `NETWORKS` array and add prices in `db.js`

**Switch to MongoDB:** Replace `db.js` functions with Mongoose calls — same interface

---

## ❓ Troubleshooting

| Problem | Fix |
|---------|-----|
| Webhook not receiving messages | Make sure your server is publicly accessible (use ngrok for local dev) |
| Payment not confirming | Check Paystack webhook URL is set correctly in dashboard |
| "Invalid signature" on webhook | Ensure `PAYSTACK_SECRET_KEY` in `.env` matches your Paystack dashboard |
| Can't log in to admin | Check `ADMIN_USERNAME` and `ADMIN_PASSWORD` in `.env` |

For local development with webhooks, use [ngrok](https://ngrok.com):
```bash
ngrok http 3000
# Use the https URL as your APP_URL and webhook base
```
