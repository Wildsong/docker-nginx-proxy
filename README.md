# docker-nginx-proxy

This is an nginx server with several functions.

Note that it's *possible* to accomplish all these things in
one container, but putting the proxy ("proxy_reverse")
and the content server ("content_server") in separate containers
means I have a built-in way of testing the proxy.

   curl -v http://localhost:81/     # test content server
   curl -v http://localhost/        # test redirection
   curl -v https://localhost/       # test reverse

(Use your hostname instead of localhost)

1 Proxy

nginx does a good job at this. I have tried using Varnish/Hitch
(very flexible but long startup time and complicated config) and
Caddy. Switching to Caddy was okay but felt a little weird to me.

I tried the official images that do all the fancy stuff of adding
Let's Encrypt certificates for me but that was just more complexity
at very little return for my small use cases.

2 Landing page

I use this at work not yet here at home.

## Proxy set-up

For most cases, NGINX has to do a MITM proxy, that is, both forward and reverse.
This is because it's doing the HTTPS handling, so all traffic has to go
through it both directions. The backend sees only HTTP traffic.

You can start a copy of the services running for testing with

   docker compose up content_server

but you should probably start everything and watch the logs,

   docker compose up -d
   docker logs --follow

Make config changes and reload with this dandy script.

   ./reload

## Content server set-up

I'm leaving these comments here for now. This is work related.

Static content is in the local folder www_content.
This includes a landing page and sundry files for my simple media server,
for example the "no image was found" image.

Media (aka "photos") are on CIFS filesystems mounted at /media/images
and /media/applications. 
Refer to the /etc/fstab file regarding CIFS services and credentials.

You can start a copy of the services running for testing with

   docker compose up content_server

## Production mode

I don't use a custom Docker image here, so there is no build step. It's just nginx published on port 80/443.
It can be deployed either with compose or as a swarm.

2024-03-29 Today I am using Compose because Swarm is not coming up and I am tired of working on this.

### Compose

   docker compose up -d

### Swarm

   docker stack deploy -c compose.yaml nginx_proxy

### Quick test

   curl http://localhost/
   curl http://localhost/photos/bridges/1002A.jpg

### Test supported URLs

Use unittest.py to do all the testing.
Add more test cases if you add more stuff.

## Resources

There are tons of guides out there, 99% are outdated.
The official documentation is unfortunately pretty sparse. For example, the section
on reverse proxy covers about 5 directives but there are about 40.

NGINX documentation

* [nginx docs](https://nginx.org/en/docs/)
* [list of variables](https://nginx.org/en/docs/varindex.html)
* [index of directives](https://nginx.org/en/docs/dirindex.html)
* [reverse proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)

