# build context คือ root ของ repo (ไม่ใช่ apps/web) เพราะ lockfile ของ npm workspaces อยู่ที่ root
FROM node:22-alpine AS build
WORKDIR /repo

# copy แค่ manifest ก่อน — แก้โค้ดแล้ว layer npm ci ยังใช้ cache เดิมได้
COPY package.json package-lock.json ./
COPY apps/web/package.json apps/web/
RUN npm ci

COPY apps/web apps/web
RUN npm run build -w apps/web

FROM nginx:alpine AS runtime
COPY apps/web/nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /repo/apps/web/dist /usr/share/nginx/html
EXPOSE 80
