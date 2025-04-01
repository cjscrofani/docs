# Cloudflare Zero Trust DNS over HTTPS (DoH) Setup on Ubiquiti/UniFi

This guide explains how to configure Cloudflare Zero Trust DNS over HTTPS (DoH) on your Ubiquiti/UniFi network using a DNS stamp. The instructions use example values so that no identifying information is included.

---

## Introduction

DNS over HTTPS (DoH) encrypts your DNS queries by sending them over HTTPS. This prevents eavesdropping and tampering with your DNS traffic. When combined with Cloudflare Zero Trust, DoH not only secures your DNS lookups but also enables advanced filtering and logging for enhanced security and privacy.

---

## Why Use DoH?

- **Enhanced Privacy:** Encrypts DNS queries to hide them from ISPs, network administrators, and attackers.
- **Improved Security:** Reduces the risk of DNS spoofing and man-in-the-middle attacks.
- **Bypass Restrictions:** Uses standard HTTPS ports, making it harder for restrictive networks to block DNS traffic.
- **Centralized Management:** Cloudflare Zero Trust provides detailed query logs, filtering rules, and analytics.

---

## Cloudflare Zero Trust Setup

1. **Log In to Cloudflare Zero Trust**  
   Access your [Cloudflare Zero Trust dashboard](https://dash.teams.cloudflare.com) (formerly Cloudflare for Teams).

2. **Create a DNS Location**  
   Under the **Gateway** section, add a new DNS location and enable DNS over HTTPS (DoH).  
   For example, you might receive a unique DoH endpoint like:  
   ```
   https://example-doh.example.com/dns-query
   ```

---

## Generating Your DNS Stamp

A DNS stamp encodes the DoH configuration details. Use an online DNS stamp calculator (e.g., [dnscrypt.info/stamps](https://dnscrypt.info/stamps/)) with the following parameters:

- **Protocol:** DNS-over-HTTPS (DoH)
- **Host name (vhost+SNI):**  
  `example-doh.example.com`
- **Path:**  
  `/dns-query`
- **IP Address / Hashes:** Leave these fields blank

The calculator will generate a stamp similar to:

```
sdns://AgcAAAAAAAAAAAAhaDexample-doh.example.comCi9kbnMtcXVlcnk
```

---

## Configuring UniFi DNS Shield

1. **Access Your UniFi Controller**  
   Log in to your UniFi Console.

2. **Navigate to the Encrypted DNS Settings**  
   Go to:  
   `Settings > Security > Protection > Encrypted DNS > Custom`

3. **Configure the Custom Rule**  
   - **Name:** Enter a descriptive name (e.g., `CFZT`).
   - **DNS Stamp:** Paste the generated DNS stamp:
     ```
     sdns://AgcAAAAAAAAAAAAhaDexample-doh.example.comCi9kbnMtcXVlcnk
     ```

4. **Apply Changes**  
   Save the settings and push the configuration to your network.

---

## Verification and Troubleshooting

- **Verify on Client Devices:**  
  Use tools like `dig` or `nslookup` or visit a DNS leak test website (e.g., [1.1.1.1/help](https://1.1.1.1/help)) to confirm that DNS queries are encrypted.

- **Check the Cloudflare Dashboard:**  
  Verify that your query logs and filtering analytics are updating correctly.

- **Review UniFi Settings:**  
  Ensure that your DHCP settings are distributing the correct DNS settings and that no conflicting configurations exist.

---

## Summary

By following this guide, you have set up Cloudflare Zero Trust DNS over HTTPS on your UniFi network using a DNS stamp. This method secures your DNS queries and leverages Cloudflare’s advanced filtering and logging capabilities—all without needing a local proxy.

For further assistance or troubleshooting, please refer to the official Cloudflare and UniFi documentation.

--- 
