# Deploy Rails App on DigitalOcean: A Complete Walkthrough From Droplet to Production — Kamal, Puma+Nginx, App Platform, Which Path Should You Pick? (With Full Pricing Breakdown and Free Credit Tips)

So you've built a Rails app. It works on your laptop. You type `rails s`, open `localhost:3000`, and there it is — your little corner of the internet, except it's not really on the internet yet. It's on your machine, where nobody else can see it. That's the moment most Rails developers start Googling something like "deploy rails app on digitalocean," and that's exactly the rabbit hole this article is going to walk you through.

There's no shortage of ways to push a Rails app into production these days. Heroku used to be the default answer, then it killed its free tier and a lot of people started looking around. Render, Fly.io, Hatchbox, Railway — they all want a piece of the Rails crowd. But DigitalOcean keeps showing up in these conversations for a reason that's hard to argue with: predictable pricing, a control panel that doesn't make you feel like you need a certification to use it, and a $200 credit for new accounts that lets you actually try things before you commit. This piece is about how to deploy Rails app on DigitalOcean in a way that makes sense for a solo dev or a small team — not a 500-word listicle, but the actual decisions you'll face and what they cost.

## Why People Pick DigitalOcean for Rails in the First Place

Before we get into the how, let's talk about the why, because the "why" shapes every choice that follows.

DigitalOcean's pitch is simple: a cloud provider that doesn't require a PhD in AWS to operate. You spin up a Droplet (their word for a virtual machine), you SSH in, you install what you need, you run your app. That's the IaaS path. Or you use App Platform, their Heroku-like PaaS, and let them handle the runtime. Or you grab the Ruby on Rails 1-Click App from their Marketplace and start from a pre-baked image with Rails, PostgreSQL, and Nginx already configured. Three doors, same building.

For Rails specifically, the appeal usually comes down to a few things:

- **Price visibility.** A Basic Droplet with 1 vCPU and 1 GiB RAM is $6/month. The same-ish footprint on the big three clouds costs more once you factor in bandwidth, and DigitalOcean bundles a generous transfer allowance (1,000 GiB on that $6 plan) with no surprise egress fees for normal traffic.
- **A Rails-shaped starting point.** The Ruby on Rails 1-Click App ships Rails 8.1.1 on Ubuntu 24.04 with PostgreSQL and Nginx pre-installed. You're not starting from a bare Ubuntu box and hand-installing Ruby.
- **A real free credit.** New accounts referred through a referral link get $200 in credit valid for 60 days. (Heads up: as of July 15, 2026, the default signup offer shifted to $5 for 90 days — the $200-for-60-days deal is tied to referral links, which is why the link at the bottom of this article matters if you want the bigger credit.)
- **Kamal-friendly.** Rails 8 ships with Kamal pre-installed, and Kamal is happy on any VPS with root access. DigitalOcean is one of the hosts Kamal's own documentation points people to.

None of this means DigitalOcean is the right answer for every Rails app. If you want zero-ops, never-touch-a-server deployment, a pure PaaS like Render or Railway will feel smoother. If you're running a giant multi-region setup, AWS or GCP give you more knobs. But for the 80% case — a single web app, maybe a background worker, a Postgres database, and a domain with SSL — DigitalOcean hits a sweet spot, and the rest of this article assumes that's roughly where you are.

## The Three Real Paths to Deploy Rails App on DigitalOcean

When you actually sit down to deploy, you're choosing between three workflows. They're not mutually exclusive — plenty of people start on one and migrate to another — but they have very different day-to-day experiences.

**Path 1: The 1-Click Rails Image on a Droplet.** You go to the Marketplace, pick the Ruby on Rails 1-Click App, choose a Droplet size, and a couple of minutes later you have a server with Rails, PostgreSQL, and Nginx already installed. You SSH in, drop your code, configure your secrets, and you're live. This is the fastest path to "it works," and it's the one most old-school Rails tutorials assume.

**Path 2: Bare Ubuntu Droplet + Kamal.** This is the modern Rails 8 way. You spin up a plain Ubuntu LTS Droplet, install Docker, point Kamal at the server's IP, run `kamal setup`, and Kamal handles building your image, pushing it to a registry, pulling it onto the server, booting the container, and provisioning a free Let's Encrypt SSL certificate. Zero-downtime deploys come for free. This is the path the Rails core team and most current Rails tutorials recommend.

**Path 3: App Platform (PaaS).** You connect your GitHub repo, DigitalOcean detects the Ruby buildpack, builds your app, runs it in a container, gives you HTTPS and a domain, and handles scaling. No SSH, no server to patch. It's the closest thing to Heroku on DigitalOcean, and it starts at $5/month for a small shared container.

Each path has a different cost shape, a different ops burden, and a different ceiling. Let's look at them in turn, then compare the pricing so you can pick with your wallet in mind.

## Path 1: The Ruby on Rails 1-Click App, Start to Finish

This is the path for someone who wants a server they fully control but doesn't want to spend an afternoon installing Ruby and compiling Postgres.

### Step 1 — Create the Droplet from the Rails Image

From the DigitalOcean control panel, go to **Create → Droplet**, then under **Marketplace** choose the **Ruby on Rails** application. The image is Rails 8.1.1 on Ubuntu 24.04. Pick a region close to your users (NYC for North America, Amsterdam for Europe, Bangalore for India, Singapore for Southeast Asia — they have 14 regions). Pick a size — for a small app, the $6/month Basic Droplet (1 vCPU, 1 GiB RAM, 25 GiB SSD, 1,000 GiB transfer) is enough to get going. Add your SSH key. Click Create.

Two to three minutes later you'll have a public IPv4 address and a running server. 👉 [Grab the $200 new-account credit and spin up your Rails Droplet here](https://bit.ly/DigitaLocean).

### Step 2 — SSH In and Look Around

bash
ssh root@your_droplet_ip


The 1-Click image drops you into a server where Rails, PostgreSQL, and Nginx are already installed. The Rails app lives in `/home/rails/` and Nginx is pre-configured as a reverse proxy. You'll want to:

- Change the default PostgreSQL password (the image sets one and tells you in the MOTD).
- Move your own app code into place (or clone your repo).
- Update `config/database.yml` with your production database credentials.
- Set `RAILS_MASTER_KEY` and any other secrets in the environment.
- Run `rails db:migrate RAILS_ENV=production` and `rails assets:precompile RAILS_ENV=production`.
- Restart Puma (the image uses Puma as the app server) and reload Nginx.

### Step 3 — Point Your Domain and Get SSL

Add an A record from your domain to the Droplet's IP. The 1-Click image ships with Certbot, so:

bash
certbot --nginx -d yourdomain.com -d www.yourdomain.com


Certbot edits your Nginx config, fetches a Let's Encrypt certificate, and sets up auto-renewal. Visit your domain and you should see the Rails welcome page or your app, over HTTPS.

### Where This Path Shines and Where It Hurts

The 1-Click path is the fastest way to a working Rails server, and it teaches you how the pieces fit together — Nginx in front, Puma behind, Postgres on the same box. The downside is that updates are on you. When a new Rails comes out, when a CVE hits Nginx, when Ubuntu ships a kernel patch — you're the one SSH-ing in at 2am. For a hobby project that's fine. For anything with users, you'll want either a deployment tool (Capistrano used to be the answer; Kamal is the current one) or a move to Path 2.

## Path 2: Bare Ubuntu + Kamal (The Rails 8 Way)

If Path 1 is "I want a server," Path 2 is "I want a deployment process." Kamal is the tool Rails 8 ships with, and it's worth understanding even if you start on Path 1, because eventually you'll want something like it.

### What Kamal Actually Does

Kamal builds a Docker image of your app on your machine (or a remote builder), pushes it to a registry (Docker Hub by default, or DigitalOcean's Container Registry), SSHes into your server, pulls the image, boots a new container, waits for it to be healthy, switches the traffic over, and shuts down the old container. The result is a zero-downtime deploy. It also handles accessory services — you can declare a Postgres container or a Redis container in `config/deploy.yml` and Kamal will run them on the server alongside your app. SSL is automatic via Let's Encrypt through Kamal's built-in proxy.

### Step 1 — Spin Up a Plain Ubuntu Droplet

This time you don't need the Rails image. Just **Create → Droplet**, pick **Ubuntu 24.04 LTS**, choose a size (1 GiB RAM is the realistic floor for a Rails app with Postgres in a container; 2 GiB is more comfortable), add your SSH key, create. 👉 [Start your Kamal-ready Ubuntu Droplet with the new-user credit here](https://bit.ly/DigitaLocean).

### Step 2 — Install Docker on the Server

bash
ssh root@your_droplet_ip
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh


That's it. Kamal uses Docker on the server to run your app's container.

### Step 3 — Configure Kamal Locally

New Rails 8 apps come with Kamal already in the Gemfile and a `config/deploy.yml` scaffolded. If you're on an older app:

bash
bundle add kamal && kamal init


Edit `config/deploy.yml` with your app name, image name, server IP, and domain:

yaml
service: my-app
image: my-user/my-app

servers:
  web:
    - 192.168.0.1

proxy:
  ssl: true
  host: app.example.com


Put your Docker registry token and `RAILS_MASTER_KEY` in `.kamal/secrets`. If you want Postgres as an accessory on the same server:

yaml
accessories:
  postgres:
    image: postgres:18
    host: 192.168.0.1
    port: 5432:5432
    env:
      clear:
        POSTGRES_USER: postgres
        POSTGRES_DB: my_app_production
      secret:
        - POSTGRES_PASSWORD
    directories:
      - data:/var/lib/postgresql/data


### Step 4 — Deploy

bash
kamal setup


The first run provisions the server, boots your app, and pulls the SSL certificate. Open your domain — your app is live. From now on, every change is:

bash
kamal deploy


That's the whole loop. Edit code, commit, `kamal deploy`, watch the zero-downtime rollout, done.

### When to Pick Kamal

Kamal is the right answer when you want deploys to be a one-liner but you still want to own the server. It's what the Rails core team uses, it's what most current Rails deployment tutorials walk through, and it pairs naturally with DigitalOcean because all Kamal needs is a Linux box with Docker and root access. The trade-off is that you're still on the hook for OS-level patching, monitoring, and backups — Kamal deploys your app, it doesn't run your server for you.

## Path 3: App Platform (The Heroku-like Option)

App Platform is DigitalOcean's answer to "I don't want to SSH into anything." You connect a GitHub or GitLab repo, DigitalOcean detects the Ruby buildpack, builds your app, runs it in a container, gives you HTTPS and a `*.ondigitalocean.app` domain (and you can bring your own), and handles scaling and OS patching. There's a Free Tier for static sites, and paid containers start at $5/month.

### How It Works for Rails

Push your Rails app to a GitHub repo with a `Gemfile` and a `config/database.yml`. In App Platform, create a new app from that repo. The Ruby buildpack picks it up. You add a database — either a development database ($7/month, 512 MiB, Postgres-only, lives with the app) or a full Managed Database (starts at $15/month). You set your environment variables (`RAILS_MASTER_KEY`, `DATABASE_URL`, etc.) in the App Platform UI. You click Deploy. A few minutes later you have a live app on a `*.ondigitalocean.app` URL with HTTPS.

### Where App Platform Wins and Loses

It wins when you want zero server touch — no SSH, no Docker, no Nginx config, no Certbot. It loses when you need something the platform doesn't expose: a custom Nginx tweak, a system package, a background daemon that isn't a worker. It's also more expensive per unit of compute than a Droplet — a $5/month shared container is 512 MiB RAM and 50 GiB transfer, while a $6/month Basic Droplet is 1 GiB RAM and 1,000 GiB transfer. You're paying for the managed experience. 👉 [Try App Platform with the new-account credit here](https://bit.ly/DigitaLocean).

## How the Three Paths Compare

| Dimension | 1-Click Rails Droplet | Bare Droplet + Kamal | App Platform |
| --- | --- | --- | --- |
| Time to first deploy | ~10 min | ~30 min | ~15 min |
| Ops burden | High (you patch everything) | Medium (you patch the OS) | Low (DigitalOcean patches) |
| Zero-downtime deploys | No (need Capistrano or Kamal on top) | Yes (built in) | Yes (built in) |
| Custom Nginx/system packages | Full control | Full control | No |
| SSL | Manual Certbot | Automatic via Kamal | Automatic |
| Cheapest realistic cost | $6/mo (1 vCPU, 1 GiB) | $6/mo (1 vCPU, 1 GiB) | $5/mo shared + $7 dev DB = $12/mo |
| Best for | Learning, single small app | Production apps you want to control | Teams that want to skip ops |

There's no single right answer. If you're deploying your first Rails app and want to understand how the pieces fit, Path 1 is the most educational. If you're shipping a real product and want a sane deploy loop, Path 2 is what the Rails community has converged on. If you're a small team that would rather pay a bit more to never touch a server, Path 3 is the low-friction option.

## The Full DigitalOcean Pricing Picture (So You Can Budget Honestly)

One of the things that makes DigitalOcean easy to recommend for Rails is that the pricing is actually readable. Here's everything you'd plausibly touch when you deploy Rails app on DigitalOcean, with the numbers as they stand now.

### Droplets (Virtual Machines)

Droplets are billed per second, with a 60-second minimum and a monthly cap, so you never pay more than the monthly price even if you leave a Droplet running all month. As of January 1, 2026, per-second billing applies to all Droplets.

**Basic Droplets** — shared CPU, best for bursty apps that don't need dedicated threads:

| Memory | vCPU | Transfer | SSD | $/hr | $/mo |
| --- | --- | --- | --- | --- | --- |
| 512 MiB | 1 | 500 GiB | 10 GiB | $0.00595 | $4.00 |
| 1 GiB | 1 | 1,000 GiB | 25 GiB | $0.00893 | $6.00 |
| 2 GiB | 1 | 2,000 GiB | 50 GiB | $0.01786 | $12.00 |
| 2 GiB | 2 | 3,000 GiB | 60 GiB | $0.02679 | $18.00 |
| 4 GiB | 2 | 4,000 GiB | 80 GiB | $0.03571 | $24.00 |
| 8 GiB | 4 | 5,000 GiB | 160 GiB | $0.07143 | $48.00 |
| 16 GiB | 8 | 6,000 GiB | 320 GiB | $0.14286 | $96.00 |

👉 [Deploy on the $6/mo Basic Droplet with the new-account credit](https://bit.ly/DigitaLocean)

For most Rails apps, the $6 (1 GiB) or $12 (2 GiB) Basic Droplet is the sweet spot. A Rails app with Postgres in a container on the same box will be happier on the $12 plan — 1 GiB is tight once you factor in the Ruby runtime, Puma workers, and Postgres shared buffers.

**CPU-Optimized Droplets** — dedicated 2.6GHz+ vCPUs, 2:1 memory-to-CPU ratio, best for CPU-bound workloads:

| Memory | vCPU | Transfer | SSD | $/mo |
| --- | --- | --- | --- | --- |
| 4 GiB | 2 | 4,000 GiB | 25 GiB | $42.00 |
| 8 GiB | 4 | 5,000 GiB | 50 GiB | $84.00 |
| 16 GiB | 8 | 6,000 GiB | 100 GiB | $168.00 |
| 32 GiB | 16 | 7,000 GiB | 200 GiB | $336.00 |
| 64 GiB | 32 | 9,000 GiB | 400 GiB | $672.00 |
| 96 GiB | 48 | 11,000 GiB | 600 GiB | $1,008.00 |

👉 [Spin up a CPU-Optimized Droplet for a CPU-heavy Rails app](https://bit.ly/DigitaLocean)

**General Purpose Droplets** — balanced memory-to-dedicated-CPU, the default for production web apps:

| Memory | vCPU | Transfer | SSD | $/mo |
| --- | --- | --- | --- | --- |
| 8 GiB | 2 | 4,000 GiB | 25 GiB | $63.00 |
| 16 GiB | 4 | 5,000 GiB | 50 GiB | $126.00 |
| 32 GiB | 8 | 6,000 GiB | 100 GiB | $252.00 |
| 64 GiB | 16 | 7,000 GiB | 200 GiB | $504.00 |
| 128 GiB | 32 | 8,000 GiB | 400 GiB | $1,008.00 |
| 160 GiB | 40 | 9,000 GiB | 500 GiB | $1,260.00 |

👉 [Start a General Purpose Droplet for a production Rails workload](https://bit.ly/DigitaLocean)

**Memory-Optimized Droplets** — 8 GiB RAM per vCPU, NVMe SSDs, for memory-hungry jobs:

| Memory | vCPU | Transfer | SSD | $/mo |
| --- | --- | --- | --- | --- |
| 16 GiB | 2 | 4,000 GiB | 50 GiB | $84.00 |
| 32 GiB | 4 | 6,000 GiB | 100 GiB | $168.00 |
| 64 GiB | 8 | 7,000 GiB | 200 GiB | $336.00 |
| 128 GiB | 16 | 8,000 GiB | 400 GiB | $672.00 |
| 192 GiB | 24 | 9,000 GiB | 600 GiB | $1,008.00 |
| 256 GiB | 32 | 10,000 GiB | 800 GiB | $1,344.00 |

👉 [Pick a Memory-Optimized Droplet for large in-memory Rails jobs](https://bit.ly/DigitaLocean)

**Storage-Optimized Droplets** — NVMe SSDs for disk-heavy workloads:

| Memory | vCPU | Transfer | SSD | $/mo |
| --- | --- | --- | --- | --- |
| 16 GiB | 2 | 4,000 GiB | 300 GiB | $131.00 |
| 32 GiB | 4 | 6,000 GiB | 600 GiB | $262.00 |
| 64 GiB | 8 | 7,000 GiB | 1,170 GiB | $524.00 |
| 128 GiB | 16 | 8,000 GiB | 2,340 GiB | $1,048.00 |
| 192 GiB | 24 | 9,000 GiB | 3,520 GiB | $1,572.00 |
| 256 GiB | 32 | 10,000 GiB | 4,690 GiB | $2,096.00 |

👉 [Use a Storage-Optimized Droplet for disk-intensive Rails workloads](https://bit.ly/DigitaLocean)

### App Platform (PaaS)

App Platform's pricing was simplified — the old Basic/Professional tiers are gone. Now you pick container instances à la carte.

| CPU Type | vCPU | Memory | Transfer | Autoscaling | $/mo |
| --- | --- | --- | --- | --- | --- |
| Free tier | — | — | 1 GiB | No | $0 |
| Shared (Fixed) | 1 | 512 MiB | 50 GiB | No | $5.00 |
| Shared (Fixed) | 1 | 1 GiB | 100 GiB | No | $10.00 |
| Shared | 1 | 1 GiB | 150 GiB | No | $12.00 |
| Shared | 1 | 2 GiB | 200 GiB | No | $25.00 |
| Shared | 2 | 4 GiB | 250 GiB | No | $50.00 |

The Free Tier covers up to 3 static-site apps with 1 GiB transfer each — not enough for a real Rails app, but useful for a marketing page in front of your app. A real Rails app on App Platform starts at the $5 Shared (Fixed) container plus a $7 development database, so $12/month is the realistic floor. Autoscaling is only available on dedicated instances (not shown in the shared table above), so if you want horizontal scaling you're moving up the price ladder.

👉 [Try App Platform with the new-account credit](https://bit.ly/DigitaLocean)

### Managed Databases (PostgreSQL)

If you don't want to run Postgres in a container on your Droplet, DigitalOcean's Managed Databases start at $15/month for a single-node 1 GiB / 1 vCPU PostgreSQL cluster. High-availability clusters (standby replica) start at $30/month. Backups are included, encryption at rest is on by default, and you get automated failover on the HA tier. For a Rails app where the database is the thing you really don't want to lose, this is the upgrade from "Postgres in a Kamal accessory on the same Droplet."

👉 [Add a Managed PostgreSQL database to your Rails deployment](https://bit.ly/DigitaLocean)

### Other Things You'll Probably Touch

- **Backups** — percentage-based (20% weekly or 30% daily of Droplet cost) or usage-based from $0.01/GiB/month.
- **Snapshots** — $0.06/GB/month. Useful for keeping a known-good image of your Rails Droplet before a big change.
- **Container Registry** — if you go the Kamal route and want to keep your images on DigitalOcean instead of Docker Hub, the registry is free for one private repo and cheap after that.
- **Floating IPs** — free while attached to a Droplet, useful for keeping a stable IP when you rebuild your server.
- **Additional outbound transfer** — $0.01/GiB on Droplets, $0.02/GiB on App Platform, beyond the included allowance.

## The Free Credit Situation, Explained Honestly

Here's the part that's genuinely useful and that most articles gloss over. DigitalOcean's signup offer has two flavors, and which one you get depends on how you sign up:

- **Default signup (no referral):** As of July 15, 2026, new accounts get a $5 credit valid for 90 days.
- **Referral signup (through a referral link):** New accounts get $200 in credit valid for 60 days. The credit is auto-applied — no coupon code to type in.

The $200-for-60-days offer is the one you want if you're going to actually try deploying a Rails app, because $200 covers a $6/month Droplet for over two years of wall-clock time (or, more realistically, lets you run a $24/month General Purpose Droplet plus a $15 Managed Database for a couple of months while you build and test). The catch is the 60-day window — once it expires, any remaining credit is gone. So it's a "try before you buy" credit, not a "free server forever" credit.

The link at the bottom of this article is a referral link, which is why it's there — if you're going to sign up anyway, using it gets you the bigger credit. 👉 [Claim the $200 / 60-day credit and start your Rails deployment](https://bit.ly/DigitaLocean).

## Common Questions People Actually Ask

**"Do I need Nginx if I'm using Kamal?"** No. Kamal ships its own proxy (Traefik under the hood) that handles SSL via Let's Encrypt and request routing. If you're on the 1-Click image or a manual Puma setup, you do want Nginx (or another reverse proxy) in front of Puma for SSL termination, static asset serving, and buffering.

**"Can I run Postgres on the same Droplet as my Rails app?"** Yes, and most small setups do. A 2 GiB Droplet can comfortably run a Rails app with Puma and a small Postgres. The trade-off is that if the app and the database fight for memory under load, they fight on the same box. Moving Postgres to a Managed Database (or a second Droplet) is the standard upgrade when you outgrow the single-server setup.

**"How does this compare to Heroku?"** Heroku's free tier is gone, and its paid tiers start higher than DigitalOcean for comparable resources. A Heroku Basic dyno is roughly $7/month for 512 MiB with no database included; a DigitalOcean $6 Droplet is 1 GiB with 1,000 GiB transfer and you can run your own Postgres. Heroku wins on ops (zero), DigitalOcean wins on price and control. App Platform is the middle ground — Heroku-like DX on DigitalOcean pricing.

**"What about Hatchbox?"** Hatchbox is a Rails-specific deployment tool that runs on top of a VPS (often DigitalOcean). It costs $10/month/server on top of the VPS cost and gives you a nice UI for deploys. If you want Kamal-like deploys without writing Kamal config by hand, Hatchbox is worth a look. It's not free, but it's cheaper than Heroku for the same footprint.

**"Is the $200 credit really $200?"** Yes, in the sense that you can spend up to $200 of DigitalOcean services within 60 days without being charged. No, in the sense that it expires. Read it as "60 days of free experimentation," not "a free Droplet for two years."

**"What's the cheapest realistic setup for a small Rails app?"** A $6/month Basic Droplet with the 1-Click Rails image (or a plain Ubuntu Droplet with Kamal) and Postgres on the same box. That's $6/month all-in for a small app with a few hundred users. Add $0.06/GB/month for snapshots if you want backups, or use the percentage-based backup option for ~$1.20/month on a $6 Droplet.

## A Suggested Starting Configuration

If you just want a recommendation and not a menu: start with a **$12/month Basic Droplet (2 GiB RAM, 1 vCPU, 50 GiB SSD, 2,000 GiB transfer)** running **Ubuntu 24.04 LTS**, deploy with **Kamal**, run **Postgres as a Kamal accessory on the same Droplet** for now, add **percentage-based weekly backups** (~$2.40/month), and point your domain at the Droplet with an A record. Total: about $14.40/month, and you can deploy new code with `kamal deploy` in one command. When you outgrow it — usually when traffic picks up or when you start caring about the database being a single point of failure — move Postgres to a $15/month Managed Database and keep the Droplet for the app. 👉 [Get started with this exact setup using the $200 credit](https://bit.ly/DigitaLocean).

## Wrapping Up

Deploying a Rails app on DigitalOcean isn't hard, but the number of options is what trips people up. The 1-Click Rails image is the fastest way to a working server. Kamal on a plain Ubuntu Droplet is the modern, repeatable way to ship a real product. App Platform is the low-ops way for teams that would rather not touch a server. The pricing is readable, the $200 new-account credit (via referral) is genuinely the most generous trial offer in this corner of the cloud market, and the Rails 1-Click image means you're not starting from a blank Ubuntu box unless you want to.

The honest summary: if you searched "deploy rails app on digitalocean" and landed here, the path that will serve you best in the long run is a plain Ubuntu Droplet with Kamal, Postgres as an accessory (or a Managed Database once you care), and the $200 credit to cover your first two months of figuring it out. Start there, and you'll have a deployment story that scales with you instead of one you have to rip out and replace.

👉 [Sign up with the $200 / 60-day credit and deploy your Rails app on DigitalOcean](https://bit.ly/DigitaLocean)
