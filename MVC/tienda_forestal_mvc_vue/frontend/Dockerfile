# ---------------------------
# ETAPA 1: BUILD DE VITE
# ---------------------------
FROM node:20-alpine AS build

WORKDIR /app

# Copiamos solo archivos esenciales primero (para cache de npm)
COPY package*.json ./

RUN npm install

# Copiamos el resto del código
COPY . .

# Construimos la aplicación
RUN npm run build


# ---------------------------
# ETAPA 2: SERVIDOR ESTÁTICO NGINX
# ---------------------------
FROM nginx:alpine

#WORKDIR /app

# Importamos solo la carpeta dist generada en la etapa 1
COPY --from=build /app/dist /usr/share/nginx/html

#Copiamos nuestra configuración personalizada de Nginx
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
