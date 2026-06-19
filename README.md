# openvpn-configs-for-ai-services

This repository provides sample OpenVPN configuration files designed to facilitate access to AI services such as **Apple Intelligence**, **Google Gemini/NotebookLM**and **OpenAI** from *regions where these services are not restricted*, with OpenVPN configuration profiles utilizing Include Tunneling. These configuration files can help users securely connect and access AI resources through **split tunneling**.

See also:

- **[tailscale-config-for-ai-services](https://github.com/gabrielkoo/tailscale-config-for-ai-services)** — if you'd rather route by **hostname** with a Tailscale [app connector](https://tailscale.com/kb/1281/app-connectors). Recommended when a service lives on a shared CDN, since hostnames keep working even when the underlying IPs rotate.
- **[wireguard-configs-for-ai-services](https://github.com/gabrielkoo/wireguard-configs-for-ai-services)** — the same IP-based approach, for WireGuard instead of OpenVPN.

## Why Bother Use Include Tunneling?

You may have wondered - *I could have just used a normal OpenVPN config profile!*

If you are turning on a always-on VPN just for the sake of securely connecting to AI resources via *an unrestricted region*, **it might slow down your entire web browsing experience as a whole**, also **it might affect your access to security-aware services like e-banking or your company resources**, which are sensitive to VPN IP addresses. If you are using a split tunnel instead, **you are only routing the traffic to the VPN only when it connects to the AI service** - this essentially reduces the traffic load on your VPN server, and **minimizes the interruption to your other normal browsing activities**.

## How It Works

Open your `*.ovpn` OpenVPN configuration file in your favourite text editor.

For each services, research for it's public IP ranges. Try to reduce the list of IP CIDR blocks into broader CIDR blocks.

For IPv4, convert the range `1.2.3.0/24` into the OpenVPN config directive `route 1.2.3.0 255.255.255.0`.

For IPV6, just prefix the IP CIDR range into e.g. `route -6 ffff::/16`.

Lastly, to avoid routing all other normal web traffic against your VPN network for uninterrupted internet speed & performance, add the directive `route-nopull` before all your custom `route` redirective.

That's it!

## AI Services Included in This Repository

AI Service | Sub Services | Details
---|---|---
Apple Intelligence | iCloud Private Relay, Writing Tools (iOS 18.1), ChatGPT Integration (iOS 18.2) | [Apple's `17.0.0.0/8` IPv4 Prefix](https://support.apple.com/en-us/101555#:~:text=Yes-,Firewalls,-If%20your%20firewall) and [Apple Intelligence, Siri, and Search](https://support.apple.com/en-us/101555#:~:text=Yes-,Apple%20Intelligence,-%2C%20Siri%2C%20and%20Search)
Google Gemini | Gemini Web App, NotebookLM | Condensed from <https://www.gstatic.com/ipranges/goog.json>.
OpenAI | ChatGPT.com, platform.openai.com | Resolved hostnames from the Cloudflare Application Library (ChatGPT, App ID `1199`), aggregated to broader CIDR blocks. Auto-refreshed — see [`scripts/update_ips.py`](scripts/update_ips.py).
Anthropic / Claude | claude.ai, api.anthropic.com | `160.79.104.0/21` (Anthropic Inc., AS399230) + dynamically-resolved GCLB front-ends. Auto-refreshed.

## Which services are practical by IP?

Not every AI service can be cleanly pinned down to IP ranges. **The IP approach only works well when a service sits on its own dedicated address space.** Use this to decide whether to reach for this repo or the [Tailscale app-connector approach](https://github.com/gabrielkoo/tailscale-config-for-ai-services) (which routes by hostname instead).

- **Good fit — dedicated address space.** Apple Intelligence (`17.0.0.0/8` is Apple's own block), Anthropic (`160.79.104.0/21`, AS399230), and Google (publishes [`goog.json`](https://www.gstatic.com/ipranges/goog.json)) own large, stable prefixes you can route wholesale.
- **Workable but brittle — shared anycast CDN.** OpenAI's `chatgpt.com` lives behind Cloudflare's shared anycast. You *can* resolve and route the specific blocks it currently answers from, but those addresses are shared with unrelated Cloudflare customers and **drift over time**, so the list needs periodic refreshing (that's what the GitHub Action does).
- **Impractical by IP — route by hostname instead.** Anything that shares a hyperscale front end with thousands of other tenants (generic `*.cloudflare.net`, `*.amazonaws.com`/CloudFront, `*.azureedge.net`, Fastly) can't be isolated by IP without scooping up huge amounts of unrelated traffic. For these, the [Tailscale app connector](https://github.com/gabrielkoo/tailscale-config-for-ai-services) is the right tool — it matches the **hostname**, so the CDN's shared IPs don't matter.

> **The Cloudflare Application Library shortcut.** Cloudflare's Zero Trust **Application Library** catalogs well-known SaaS apps together with the hostnames they've observed. It's the fastest way to discover an AI platform's real domains — then you resolve those hostnames to IPs and aggregate. [`scripts/update_ips.py`](scripts/update_ips.py) automates exactly this (pull hostnames → resolve over DoH → aggregate → rewrite the `route` blocks), and runs on a schedule via [GitHub Actions](.github/workflows/update-ips.yml). See the [Tailscale repo's "How Do I Come Up With the List of Domains?"](https://github.com/gabrielkoo/tailscale-config-for-ai-services#how-do-i-come-up-with-the-list-of-domains) for the full methodology.

## Disclaimer**

The use of these configuration files is at your own risk. This repository is intended only for users whose home country is **permitted** to access these AI services. The author takes no responsibility for any misuse or legal implications that may arise from the use of these files.
