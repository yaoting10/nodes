### Nginx



docker run --name nginx -v /Users/tim/Sites/nginx.conf:/etc/nginx/conf.d/default.conf:ro -v /Users/tim/Sites/dragon_portal:/usr/share/nginx/html:ro -p 80:80 -d nginx