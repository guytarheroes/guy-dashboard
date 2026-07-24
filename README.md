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

Pi อยู่หลังเราเตอร์บ้าน ไม่มี public IP เลยไม่เปิด port ออกเน็ต เส้นทางที่มีอยู่แล้วคือ

```
Cloudflare edge → cloudflared (tunnel) → Nginx Proxy Manager → guytarheroes-web:80
```

tunnel และ route ของ `guytarheroes.com` **ตั้งไว้อยู่แล้ว** ไม่ต้องสร้างใหม่ — ที่ขาดคือ proxy host ใน NPM

NPM admin → **Hosts → Proxy Hosts → Add Proxy Host**

| ช่อง | ค่า |
|---|---|
| Domain Names | `guytarheroes.com`, `www.guytarheroes.com` |
| Scheme | `http` |
| Forward Hostname / IP | `guytarheroes-web` |
| Forward Port | `80` |
| Block Common Exploits | เปิด |

ไม่ต้องออก cert ในแท็บ SSL — TLS จบที่ Cloudflare แล้ว ช่วงนี้เป็นวงในทั้งหมด

ทุก container (`cloudflared_tunnel_1/2`, NPM, `guytarheroes-web`) อยู่ network `server_default`
จึงเรียกกันด้วยชื่อ container ได้ตรง ๆ — `docker-compose.yml` เลยประกาศ network นั้นเป็น `external`

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
