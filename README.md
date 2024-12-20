Ansible Role to set up Bluesky PDS
=========

This role helps you set up a [Personal Data Server](https://github.com/bluesky-social/pds) for Bluesky with Ansible. With it, you can deploy PDS easily and consistently on your server(s).

> [!NOTE]  
> This Ansible role is not officially supported by Bluesky.

Requirements
------------

This package uses Docker (to deploy the server), OpenSSL (to generate secrets) and curl (to download an administration script). You will need your own reverse proxy/web server to deploy the PDS yourself. PDS works best with [Caddy](https://caddyserver.com/), so this is what the role recommends you to use.

Role Variables
--------------

| Variable | Purpose | Needs to be changed? |
| -------- | ------- | -------- |
| bluesky_pds_version | Allows you to change what version of Bluesky PDS you want to download. [Check this page to see what options are supported](https://github.com/bluesky-social/pds/pkgs/container/pds). | No |
| bluesky_pds_folder | Allows you to change where on the host system you want to store the files for the PDS. The default value of /pds is recommended, because the ``pdsadmin`` script doesn't really work without it. | No | 
| bluesky_pds_port | The port that the Bluesky PDS container will be forwarded to. By default, this is 3000, which should work in most cases, but if you have another service running on this port, you should specify another port. | No |
| bluesky_pds_hostname | The domain the PDS is being run on. If you are running the PDS server on ``example.com``, you would set this variable to ``example.com``. | Yes |
| bluesky_pds_admin_password | The admin password the PDS will use. I highly recommend using something like [Ansible Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html) to encrypt the admin password inside your playbook. | Yes |
| bluesky_pds_upload_limit | Upload limit for files. This is in bytes. The default is 52428800 bytes (around 52 MB) | No |
| bluesky_pds_plc_url | What PLC directory to use. The default is enough in most usecases. | No |
| bluesky_pds_app_view_url | Bluesky API url. The default is enough in most usecases. | No |
| bluesky_pds_app_view_did | Same as above, but in [DID](https://www.w3.org/TR/did-core/) format. The default is enough in most usecases. | No |
| bluesky_pds_report_service_url | The server that reports should be sent to. The default is enough in most usecases. | No |
| bluesky_pds_report_service_did | Same as above, but in DID format. The default is enough in most usecases. | No |
| bluesky_pds_crawlers | What relay instance to use. The default is "bsky.network", which is enough in most usecases. | No |
| bluesky_pds_watchtower_enabled | If you don't want to include [Watchtower](https://containrrr.dev/watchtower/), a service that automatically updates Docker container upgrades in the Compose file, set this to false. This is enabled by default. | No |

Example Deployment
----------------

### Playbook
Here's how a basic playbook using this role could look like.

```yaml
    - hosts: servers
      roles:
         - role: stilktf.ansible_bluesky_pds
           bluesky_pds_port: 8080 # If you want to change the forwarded port (what the port for the PDS will be on your host system) use this var
           bluesky_pds_hostname: changeme.xyz # Needs to be changed. This is the domain your PDS will be run on. This isn't automatically fixed for you; bring your own web server.
           bluesky_pds_admin_password: changeme1234 # Again, this needs to be changed. You should generate a secure, hard to crack password and store it using Ansible Vault. 
           bluesky_pds_secret: changeme # PDS secret. You can generate a secret using for example "openssl rand --hex 16". Ansible Vault is again a very nice tool for making sure nobody knows the secret but you.
           bluesky_pds_plc_rotation_key: changeme # PLC rotation key. Again, generate the secret using "openssl ecparam --name secp256k1 --genkey --noout --outform DER | tail --bytes=+8 | head --bytes=32 | xxd --plain --cols 32". Ansible Vault is useful for keeping secrets secure.
```

### Web server
This assumes you're using the Caddy web server.

```caddyfile
changeme.xyz, *.changeme.xyz {
	reverse_proxy :8080 # bsky
}
```

You could also look at the Caddyfile the [PDS installer script](https://github.com/bluesky-social/pds/blob/main/installer.sh#L308) generates. 

### Networking
You would need to set your servers public IP address to the root of ``changeme.xyz`` and ``*.changeme.xyz`` in your domain's DNS settings. Subdomains and deeper subdomains are supported by Bluesky, but just as a heads up, deployments like ``social.changeme.xyz`` and ``*.social.changeme.xyz`` on Cloudflare would need you to pay a fee: multi-domain subdomains are not supported by "Universial SSL certificates". See [here](https://developers.cloudflare.com/ssl/troubleshooting/version-cipher-mismatch/#multi-level-subdomains) for more details. 

Assuming you're using a web server like Caddy, you only need to open port 80 and 443 from your device to the Internet, and ensure that these ports should be accessible on your servers public IP address. You can use a service like [yougetsignal.com](https://www.yougetsignal.com/tools/open-ports/) to check if ports are open.

License
-------

Apache 2.0

Author Information
------------------

Website: [stilk.tf](https://stilk.tf)
Bluesky: [@stilk.tf](https://bsky.app/profile/stilk.tf)
