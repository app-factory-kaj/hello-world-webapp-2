FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm i
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx/nginx.conf /etc/nginx/nginx.conf
COPY nginx/default.conf /etc/nginx/conf.d/default.conf
# Cloud web-application pods run as UID 65534 with a read-only root FS and
# no emptyDir. Pre-create nginx's temp dirs so startup mkdir() is EEXIST
# rather than EROFS; pid/logs go to /dev/null and the runtime fds (nginx.conf).
RUN mkdir -p \
        /var/cache/nginx/client_temp \
        /var/cache/nginx/proxy_temp \
        /var/cache/nginx/fastcgi_temp \
        /var/cache/nginx/uwsgi_temp \
        /var/cache/nginx/scgi_temp \
    && chmod -R a+rX /usr/share/nginx/html /var/cache/nginx
EXPOSE 9090
# Skip the stock nginx entrypoint: it tries to rewrite conf.d on start and
# that write fails under a read-only root. nginx itself is enough.
ENTRYPOINT ["nginx", "-g", "daemon off;"]

