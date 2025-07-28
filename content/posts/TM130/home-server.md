---
title: "Nixos, ACME, Tailscale, and Nginx?"
date: "2025-07-15"
tags: ["vim"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "everything just works (kinda)"
disableHLJS: true
disableShare: true
hideSummary: false
searchHidden: false
ShowReadingTime: false
ShowBreadCrumbs: false
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: false
UseHugoToc: false
---

## Why?

Now, why would I want to run a home server and have HTTPS even though Tailscale is peer-to-peer encrypted. I'm a **goober**. Realistically, this is all for practice. You do **_not_** have to do this on your network to feel safe. Tailscale and MagicDNS will get you everywhere you need, and want, to go. But let's say you would want to set up HTTPS, how would you do that, especially on NixOS.

## How?

### Prerequisite

I will do my best to provide adequate context to each code snippet contained in this post. But I'm sorry if I miss anything crucial. If something appears inncorect, please leave an issue on the website [repo](https://github.com/Sheepheerd/Sheepheerd.github.io).

### Domain

First, obtain a domain. I obtained my domain through [porkbun](porkbun.com), but there are countless others that can be found by a Google search.

The **.dev** domains are special in that they are managed by Google. They come pre-registered on the HSTS and are enforced to have HTTPS. Porkbun makes it easy to generate the certificate through Let's Encrypt, and we can then move our dns management over to cloudflare. Go to cloudflare's website, generate an account, go through it's walkthrough.

This is all very early steps that I am assuming would be done by now if you are at the point to where you want to get https working with tailscale.

### Post Cloudflare/Cert

#### Tailscale

Let's make sure we enable the Tailscale service.

```nix
    services.tailscale.enable = true;
    networking = {
      firewall = {
        trustedInterfaces = [ config.services.tailscale.interfaceName ];
        allowedUDPPorts = [ config.services.tailscale.port ];
        checkReversePath = "loose";
      };
      networkmanager.unmanaged = [ "tailscale0" ];
    };
```

These settings can be kept the same.

#### ACME

Once you have set up Cloudflare, or some other manager, we need to set up ACME on Nixos. [ACME](https://nixos.wiki/wiki/ACME) is the Automatic domain validation and certificate retrieval and renewal. We can follow the steps on the NixOS docs by enabling the module.

```nix
    security.acme = {
      acceptTerms = true;
      preliminarySelfsigned = false;
      defaults = {
        email = "shepard@heerd.dev";
        dnsProvider = "cloudflare";
        environmentFile = "${config.age.secrets.acme-api.path}";
      };
    };
```

The things to pay attention to here are that you will need an **email**, your **dnsProvider** and an **environmentFile**.

The email, can be whatever you want so long as it is in the correct email form. This is important so that you do not dox your email. You must set a default email, but it can be **_fake_** and overwritten by the file. What is in the file is in the form of

```bash
CLOUDFLARE_API_KEY=
CLOUDFLARE_EMAIL=
CLOUDFLARE_ZONE_API_TOKEN=
```

If you do not have those api tokens, you must generate them on the cloudflare website. I am simply using the global api token and a generate zone token.

#### Nginx

ACME should be set up correctly now. Next, we must set up Nginx.

```nix
    users.users.nginx.extraGroups = [ "acme" ];
    services.nginx = {
      enable = true;
      statusPage = true;
      recommendedProxySettings = true;
      recommendedTlsSettings = true;
      recommendedOptimisation = true;
      recommendedGzipSettings = true;
    };
```

These settings can stay the same for your configuration, but make sure that acce the Nginx user to the acme group.

#### Blocky

We need blocky so that when we connect to other Tailscale nodes, we will be able to route them to the correct host. However, we will also need to go to the Tailscale dashboard and set the dns to our blocky host. This will be under Nameservers in the dns tab on the Tailscale dashboard.

### Nginx Routes

I will give different examples of how I have set up my Nginx routes so that you can derive from them

#### Firefly

```nix
    # Firefly III
    services.firefly-iii = {
      enable = true;
      dataDir = "/vault/data/firefly-iii";
      enableNginx = true;
      virtualHost = domain;
      settings = {
        TZ = "America/New_York";
        APP_ENV = "production";
        APP_KEY_FILE = config.age.secrets.firefly-app-key.path;
        DB_CONNECTION = "sqlite";
        APP_URL = "https://${domain}";
      };
    };
    services.nginx.virtualHosts.${domain} = {
      enableACME = true;
      forceSSL = true;
    };
```

Take note of the **_APP_URL_**. This must be the name of the subdomain you have setup for it.

#### Jellyfin

```nix
    services.jellyfin = {
      enable = true;
      dataDir = "/mnt/two-t-hdd/jellyfin";
      user = "jellyfin";
    };

    services.nginx = {

      virtualHosts = {
        "movies.heerd.dev" = {
          enableACME = true;
          forceSSL = true;
          acmeRoot = null;
          locations = {
            "/" = {
              proxyPass = "http://127.0.0.1:8096";
              extraConfig = ''
                client_max_body_size 0;
              '';
            };
          };
        };
      };
    };
```

Take note that acmeRoot is null. This will break when set.

#### Others

Most services will follow the jellyfin setup, however, please check the service's docs on NixOS for anything special.

## Conclusion

This will allow you to access subdomains through the Tailscale network giving you HTTPS. This is not more secure. It just looks pretty and is fun to practice.
