docker network create --driver bridge reverse-proxy

docker rm -f nginx
docker run -d -p 80:80 -p 443:443 \
	--name=nginx \
	--restart=always \
	--network=reverse-proxy \
	-v /home/virtual/nginx/certs:/etc/nginx/certs:ro \
	-v /home/virtual/nginx/conf.d:/etc/nginx/conf.d \
	-v /home/virtual/nginx/vhost.d:/etc/nginx/vhost.d \
	-v /home/virtual/nginx/html:/usr/share/nginx/html \
	-v /var/run/docker.sock:/tmp/docker.sock:ro \
	-e NGINX_PROXY_CONTAINER=nginx \
	--label com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy=true \
	jwilder/nginx-proxy

docker rm -f nginx-letsencrypt
docker run -d \
    --name nginx-letsencrypt \
    --restart=always \
    --network=reverse-proxy \
    --volumes-from nginx \
    -v /home/virtual/nginx/certs:/etc/nginx/certs:rw \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    jrcs/letsencrypt-nginx-proxy-companion
