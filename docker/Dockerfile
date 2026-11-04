FROM nginxinc/nginx-unprivileged:alpine

WORKDIR /usr/share/nginx/html

COPY app/ /usr/share/nginx/html/

EXPOSE 8080

CMD ["nginx","-g","daemon off;"]
