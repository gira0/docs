# Cloudflare remote with cert-auth

## 1. Add cloudflared add-on

First, add the cloudflared app: https://github.com/homeassistant-apps/app-cloudflared/

Check out the repo or click this link here, it's part of the Unofficial Home Assistant Apps (Add-ons) repos, perhaps you already have it:

[![Add Repository to HA][my-ha-badge]][my-ha-url]

[my-ha-url]: https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fhomeassistant-apps%2Frepository
[my-ha-badge]: https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg

## 2. Setup the cloudflared app

We need a Cloudflare account for this, and a domain name. I recommend a .xyz domain (https://gen.xyz/premiums) since there are options where you can get a domain for 0.99 USD a year, For a domain that is entered once and then not really ever seen again, a numeric domain works well (example: 0101010.xyz), You can use any domain you want, just get one and set it up with Cloudflare, Buying directly from Cloudflare reduces setup and avoids extra markup,

After you install the app, ensure the `Start on boot` and `Watchdog` options in the app are enabled so the app is always running, You can also enable `Auto update` if you are comfortable with that,

### Configuration

In the configuration you have the following fields we need to fill in:

- External Home Assistant URL: This is the Domain you want to setup, I recommend choosing a subdomain for this, also gives the option to host more than one service on the same domain, Example: `ha.0101010.xyz`
- Cloudflare Tunnel Name: The name of the cloudflare tunnel, I recommend adding the domain name here as well, example: `ha-0101010-xyz`
- Cloudflare Tunnel Token: This is the token we are going to generate, It is important to keep it secret

### Home Assistant configuration

You need to add the following configuration to your `configuration.yaml` file,

```yaml
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 172.30.33.0/24
```

This is required so that the data from the proxy created by cloudflared is actually trusted and received by Home Assistant, The IP range `172.30.33.0/24` is used by cloudflared and is a private IP range, so there is no security risk in trusting it,

### Setup in cloudflare dashboard

#### Tunnel

In the dashboard where you have your domain already set up, just go to the search (CTRL+K) just search for `Tunnels`, you should end up in the `Protect & Connect` > `Networking` > `Tunnels` area, There, click on the top right `Create Tunnel`,

Set up the following things,

- Tunnel Name: Name of the tunnel, the name you need to enter in the home assistant, This can be whatever you want, I recommend adding the domain name here as well, example: `ha-0101010-xyz`, then press `continue`,
- Setup Environment: Here you get the Cloudflare Tunnel Token that you need to enter in the cloudflared app, You can choose any operating system, copy the text from the `Run the following command` area into a text file, Example command: `cloudflared.exe service install eyJhIjoiZmVlMjUzZDA0MDJlODY2NDc4ODQ0ZTkxMGQ4MWU4ZGIiLCJ0IjoiMTJlMGRiYWItYzczYi00ZmRjLWFkZTQtMDFjZWIxZTFjOTEyIiwicyI6Ik5ERTJPVEJrTW1FdE1XVTVPUzAwTlRjM0xXRXhNamd0TWpJell6bGxOVFJrTkdJeiJ9`, Copy the token part at the end into the Home Assistant app and press save,

You can only continue if the cloudflared app is now running with the hostname, tunnel name and tunnel token, The website will wait and check if the tunnel is up and running, if it is, you can continue and are done with this step,

#### Route

You should still be in the `Tunnels` area, click the tunnel you created, then click the `Add route` button, Here you need to set up the following,

- Route: This is the domain you want to use (example: `ha`), then select the domain (example: `0101010.xyz`), so the full domain will be `ha.0101010.xyz`,
- Service URL: This is the URL the cloudflare tunnel will forward the traffic to on your system, some guides just enter the IP, I recommend the proper URL that is `http://homeassistant.local.hass.io:8123` so you are safe should your IP change,

After this setup, your address should be up and running and you can test it out, You are already done if you trust Home Assistant's security and have set up a strong password, If you want to be extra safe, continue with the next step, which I strongly recommend,

#### Access control with cert-auth

Go to the search again (CTRL+K) and search for `Client Certificates` and choose the domain if asked, Top right, create a new certificate,

Select the following options,

- Certificate Authority: `Cloudflare Origin CA`, it's just simpler,
- Private Key Type: `ECC`, it's more secure than the 2048 bit RSA and also faster, so it's a win win,
- Certificate Validity: `15 years`, you don't want to have to renew this every year, so just go with the maximum,

You can now download the certificate, choose `PEM` for `Key Format`, and copy the `Certificate` and `Private Key` into separate text files,

After this it asks you to add an `Associated Hostname`, enter the domain you've chosen for Home Assistant (example: `ha.0101010.xyz`), then press `Add` and `Save`,

Unfortunately the not-so-fun part follows, you need to convert the certificate to PFX format (recommended with a password for compatibility), On Linux this is quite easy, run the following command in the terminal, making sure to change the file names and password,

```sh
openssl pkcs12 -export -out ha-cert.pfx -inkey key.pem -in cert.pem -password pass:password
```

On windows this should do the trick(untested but hopefully works):

```powershell
$cert = [System.Security.Cryptography.X509Certificates.X509Certificate2]::CreateFromPemFile("cert.pem", "key.pem")
$pfxBytes = $cert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pfx, "password")
[System.IO.File]::WriteAllBytes("$PWD\ha-cert.pfx", $pfxBytes)
```

Copy this certificate to all devices that you want to access the home assistant, How to import the certificate depends on the device, but usually you can just click on the file and it should ask you to import it, Good luck!

##### Security Rules

###### Rule 1

Firewall rules are needed to actually secure the domain, In the `Client Certificates` area, click the `Create mTLS Rule` button and set up the following,

- Enforce mTLS authentication: `Client certificate Verified` set to `true`,
- Hostname: `equals` and then enter the domain you chose (example: `ha.0101010.xyz`),

In the `Choose action` dropdown, select `Skip` and then click on `WAF components to skip` `All remaining custom rules`,
Then `Place at` `Select order` `First` and then `Save`,

##### Rule 2

This rule is to block all traffic that doesn't have a valid client certificate, so it blocks all traffic that doesn't have the right certificate, which is basically all traffic that is not from you or someone you shared the certificate with,

- Hostname: `equals` and then enter the domain you chose (example: `ha.0101010.xyz`),

In the `Choose action` dropdown, select `Block`,

Then `Place at` `Select order` `Last` and then `Save`,

DONE
