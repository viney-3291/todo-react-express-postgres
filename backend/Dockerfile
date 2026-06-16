FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json .
COPY prisma ./prisma
RUN npm install
COPY . .
RUN npm run build

FROM node:20-alpine AS production
WORKDIR /app
RUN apk add --no-cache openssl
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/prisma ./prisma
COPY package.json .
EXPOSE 8080
CMD ["node", "dist/index.js"]