# BPCDMS Backend (Node.js + Express + SQLite)

Yeh backend aapke Canva-generated **BPCDMS** frontend ko live server pe run karne ke liye hai.
Original frontend Canva ke internal `window.dataSdk` (Canva Sheet) pe depend karta tha — jo sirf Canva ke andar hi kaam karta hai. Isliye `public/index.html` mein sirf **ek hi cheez add ki gayi hai**: `window.dataSdk` ka apna implementation jo humare Express API se baat karta hai. Baaki poora UI, saari forms, saari logic (case stages, IO management, messaging, leave approval, charts, everything) **bilkul waisa hi hai jaisa Canva ne banaya tha**.

## Folder Structure

```
bpcdms-backend/
├── server.js          # Express + SQLite backend (4 API endpoints)
├── package.json
├── public/
│   └── index.html      # Aapka original frontend + dataSdk shim
└── bpcdms.db            # SQLite database (auto-create hoti hai jab server chalega)
```

## Kaise chalayein (Local)

```bash
cd bpcdms-backend
npm install
npm start
```

Fir browser mein kholiye: **http://localhost:3000**

Server start hote hi `bpcdms.db` naam ki SQLite file khud-ba-khud ban jayegi — koi alag database install karne ki zaroorat nahi.

## API Endpoints (agar kabhi zaroorat pade)

| Method | Endpoint             | Kaam                              |
|--------|-----------------------|-----------------------------------|
| GET    | `/api/records`         | Saare records fetch karna         |
| POST   | `/api/records`         | Naya record create karna          |
| PUT    | `/api/records/:id`     | Existing record update karna      |
| DELETE | `/api/records/:id`     | Record delete karna               |
| GET    | `/api/health`          | Server chal raha hai ya nahi check|

Har record ek generic JSON object hai jisme `record_type` field hoti hai: `case`, `io`, `message`, `notification`, ya `leave` — bilkul waisa hi jaisa original Canva version mein tha.

## Live Server Pe Deploy Kaise Karein

Kisi bhi Node.js hosting pe chal jayega. Sabse aasan options:

### Option 1 — Render.com (free tier available)
1. Is poore folder ko GitHub repo mein push kar dijiye
2. Render.com pe "New Web Service" banayein, apna GitHub repo connect karein
3. Build Command: `npm install`
4. Start Command: `npm start`
5. Deploy karte hi aapko ek live URL mil jayega (jaise `https://bpcdms.onrender.com`)

### Option 2 — Railway.app
1. GitHub repo connect karein
2. Railway khud detect kar lega ki yeh Node.js app hai
3. Deploy → live URL mil jayega

### Option 3 — Apna VPS (Hostinger, DigitalOcean, AWS EC2, etc.)
```bash
npm install -g pm2
cd bpcdms-backend
npm install
pm2 start server.js --name bpcdms
pm2 save
```
Fir Nginx se reverse-proxy laga kar apna domain point kar dijiye.

> ⚠️ Important: Jahan bhi deploy karein, `bpcdms.db` file ko **persistent disk/volume** pe rakhna zaroori hai — kuch free hosting platforms (jaise Render free tier) disk ko restart pe reset kar dete hain. Agar aisा hota hai to Render ka "Persistent Disk" add-on use karein, ya database ko Railway/VPS pe rakhein.

## Real-time Sync Kaise Kaam Karta Hai

- Jab bhi koi user (State HQ / District / IO / Complainant) koi action karta hai (naya case, message, leave, etc.), woh turant server pe save ho jata hai aur us user ki screen turant update ho jati hai.
- Baaki sab logged-in users ki screens har **4 seconds** mein automatically refresh hoti hain (polling) — taaki sabko latest data dikhe, bina manually reload kiye.

## Image Uploads

FIR/Case registration mein jo image upload hoti hai, woh base64 format mein seedha database mein store hoti hai (jaise original Canva Sheet version mein hoti thi). Agar aage chal kar bahut zyada images/cases ho jayein (hazaron), to future mein images ko alag se file-storage (jaise AWS S3 / Cloudinary) mein rakhna better rahega — abhi ke liye yeh approach directly kaam karega.

## Security Note

Yeh ek functional starter backend hai. Agar isse asli production/government use ke liye deploy karna hai, to yeh zaroor add karein:
- HTTPS (Render/Railway automatically deta hai)
- Proper authentication (District/IO login abhi bhi frontend mein hardcoded hai, jaisa original Canva version mein tha)
- Rate limiting aur input validation
- Regular database backups
