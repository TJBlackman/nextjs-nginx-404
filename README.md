# NextJs + Nginx Reverse Proxy 404 Errors

## Description

When using NginX as a reverse proxy to a NextJS app, static files (JS files) come up as 404 not found. This happens when NginX is configured to add caching headers to static files via this config block:

```config
# cache static files for 7 days
location ~* \.(?:css(\.map)?|js(\.map)?|jpe?g|png|gif|ico|cur|heic|webp|tiff?|mp3|m4a|aac|ogg|midi?|wav|mp4|mov|webm|mpe?g|avi|ogv|flv|wmv)$ {
    expires    7d;
    access_log off;
}
```

## Setup

A brief description of how this project is setup. You don't have to do any of this.

- NextJS App created using `npx create-next-app@latest` - v14.4 at the time of making this demo (happens with my NextJS v13.x app as well).
- NextJS App is dockerized following NextJS's docs here: https://github.com/vercel/next.js/tree/canary/examples/with-docker
  - Edit `next.config.js` to include `output: "standalone"`
  - Create Dockerfile according to NextJS Documentation
- The `nginx` folder at the root of this project has all the nginx config.
- A `docker-compose.yaml` file was created to launch both apps.
  - NginX listens on port 80 for incoming traffic, and reverse-proxies traffic to the NextJS app.

## Requirements

- Docker and NodeJS must be installed on your system.

## Reproduce the Issue

- Update `docker-compose.yaml` for correct nginx volume mapping
  - Right click on `nginx.conf` and click `Copy Path` to get the full system path. Paste on line 8 of `docker-compose.yaml`.
  - Do the same with `nginx/settings` on line 9, and `nginx/server-blocks` on line 10.
- Run the app with `docker compose up` from the root directory.
- Go to http://localhost
  - Static files, JS and CSS, are all 404 not found!

## Fix this issue

- Edit the file `nginx/settings/general.conf`
  - Comment out caching declarations; lines 14-17, and 20-24
- Run the app with `docker compose up` from the root directory.
- Go to http://localhost
  - Static files work again!

## How do I get around this issue?

My NginX server serves more than just my NextJS App, including static files on the host machine, so I would like to keep these caching declarations in tact. Some possible options are:

1. Comment out this block in NginX. This is a poor solution, as other web resources benefit from this directive. 2/10 solution.
2. Declare caching directives in their own file, and include them in server blocks that need them. This would work, but is slightly more tedious. 8/10 solution.
3. Host NextJS app static files on host machine instead of inside Docker container, and configure NginX to serve those files. I don't like this solution, as it separates the static files from the server file. Although, it may be more performant, it does add some complexity. 6/10 solution.
   - I'm actually currently doing this, and it was obnoxious to set up, and also obnoxious to wrap my head around it when I came back to it 8 months later.
4. Same as 3, but upload files to CDN instead of hosting on NginX machine. Same pros and cons as solution 3. 7/10 solution.


> Note: Serving static files from a JS server is SLOW! For public production applications, use a dedicated file server (CDN, NginX, etc). Most importantly, know the pros and cons of the solutions above, and choose what's best for your specific use case.
