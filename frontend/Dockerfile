FROM node:18-alpine AS build
WORKDIR /app
COPY package.json ./
COPY package-lock.json* ./
RUN npm install --legacy-peer-deps
COPY . .
RUN CI=true npm test
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]