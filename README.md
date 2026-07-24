# guytarheroes.com

Landing page ท่องเที่ยว — Astro + Tailwind, เสิร์ฟด้วย nginx ใน Docker, ออกเน็ตผ่าน Cloudflare Tunnel

```
apps/web/          Astro site (หน้าเดียว)
docker-compose.yml deploy
```

## Dev

```bash
npm install
npm run dev        # http://localhost:4321
npm run build      # ออกที่ apps/web/dist/
```

## Deploy บน Raspberry Pi

```bash
git clone git@github.com:guytarheroes/guy-dashboard.git
cd guy-dashboard
docker compose up -d --build     # build native arm64 บนเครื่อง ใช้เวลา 2-3 นาทีรอบแรก
curl -I http://127.0.0.1:8080/   # ต้องได้ 200
```

image ที่ได้ ~28MB (nginx:alpine + static files ล้วน ไม่มี node_modules ติดไป)

### ผูกกับ guytarheroes.com

Pi อยู่หลังเราเตอร์บ้าน ไม่มี public IP เลยไม่เปิด port ออกเน็ต — ให้ Cloudflare Tunnel ที่รันอยู่แล้ว
(ตัวเดียวกับที่ใช้ SSH) เป็นทางเข้าเดียว

Cloudflare Zero Trust → Networks → Tunnels → เลือก tunnel เดิม → **Public Hostnames** → Add:

| | |
|---|---|
| Subdomain | (เว้นว่าง) |
| Domain | `guytarheroes.com` |
| Service | `HTTP` → `guytarheroes-web:80` |

`cloudflared` บนเครื่องนี้รันเป็น container (`cloudflared_tunnel_1`, `cloudflared_tunnel_2`) อยู่บน network
`server_default` — `docker-compose.yml` เลยต่อ `web` เข้า network นั้นเป็น external เพื่อให้เรียกกันด้วยชื่อ container ได้

TLS จบที่ Cloudflare ไม่ต้องทำ cert ที่ origin

## Stack

| | | |
|---|---|---|
| Astro 7 | static output | ship JS 0 byte ยกเว้น scroll reveal ~1kB |
| Tailwind v4 | ผ่าน `@tailwindcss/vite` | ไม่มี `tailwind.config.js` |
| animation | CSS + IntersectionObserver | ไม่มี dependency |
| รูป | `astro:assets` | JPEG → WebP + srcset อัตโนมัติ (572kB → 27kB) |

หน้าเว็บรองรับ dark mode ตาม `prefers-color-scheme` และปิด animation ให้เองเมื่อผู้ใช้ตั้ง reduce motion

## Credit

รูปทั้งหมดจาก [Unsplash](https://unsplash.com) (Unsplash License — ใช้เชิงพาณิชย์ได้ ไม่ต้องให้เครดิต)
เก็บไว้ใน `apps/web/src/assets/`
