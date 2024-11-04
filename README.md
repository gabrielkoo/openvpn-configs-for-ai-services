# openvpn-configs-for-ai-services

This repository provides sample OpenVPN configuration files designed to facilitate access to AI services such as **Apple Intelligence**, **Google Gemini/NotebookLM**and **OpenAI** from *regions where these services are not restricted*, with OpenVPN configuration profiles utilizing Include Tunneling. These configuration files can help users securely connect and access AI resources through **split tunneling**.

You may wonder - *I could have just used a normal OpenVPN config profile, why bother include tunneling?* 

If you are turning on a always-on VPN just for the sake of securely connecting to AI resources via *an unrestricted region*, it might slow down your entire web browsing experience as a whole, also it might affect your access to security-aware services like e-banking or your company resources, which are sensitive to VPN IP addresses. If you are using a split tunnel instead, you are only routing the traffic to the VPN only when it connects to the AI service - this essentially reduces the traffic load on your VPN server, and minimizes the interruption to your other normal browsing activities.

## How It Works

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
OpenAI | ChatGPT.com, platform.openai.com | By Trial and Error.

**Disclaimer**: The use of these configuration files is at your own risk. This repository is intended only for users whose home country is **permitted** to access these AI services. The author takes no responsibility for any misuse or legal implications that may arise from the use of these files.
