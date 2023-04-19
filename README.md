# Shareable Tailscale Docker Container

This configuration will,

- Start and authenticate [Tailscale](https://tailscale.com) with [Userspace Networking](https://tailscale.com/kb/1112/userspace-networking/)
- Start nginx listening on port 80 and 443 sharing the local directory on /
- Create a [Docker volume](https://docs.docker.com/storage/volumes/) called `varlib` to store Tailscale state and Let's Encrypt Certificates

## Prerequisites

This guide assumes the following,

- A Tailscale account is setup
- [MagicDNS](https://tailscale.com/kb/1081/magicdns/) is enabled
- [HTTPS in Tailscale](https://tailscale.com/kb/1153/enabling-https/) is enabled

## Required Hostname Changes

Update the hostname and Tailnet in the following files to match your own,

- `server_name` in `default.conf`
- `ssl_certificate` and `ssl_certificate_key` file locations in `shared.conf`
- `hostname` in `docker-compose.yml`

## Tailscale Authentication

Generate a [Auth Key](https://tailscale.com/kb/1085/auth-keys/) to use for the container. This will

- Automatically authenticate node on start
- Replace/reuse the existing IP after registration

Generate the key with,

- Reusable: No
- Expiration: 90 Days
- Ephemeral: No
- Pre-approved: Yes

Tailscale state is stored in the `varlib` volume mounted on `/var/lib`, this keeps the hostname and IP if the  container is restarted.

## Running Container

On first run, start up the containers with the Auth Key from above. This is only needed the first time docker compose is run.

```
TS_AUTHKEY=tskey-auth-XXXXXXXX docker compose up
```

Nginx will fail to start due to missing the Let's Encrypt certificates.

While the containers are still running, generate the Let's Encrypt certs for the hostname and domain in the tailscale container. This will store them in the shared `varlib` volume so the Nginx container will have access.

```
docker exec -it tailscale sh -c 'cd /var/lib ; tailscale cert HOSTNAME.tailnet-XXXX.ts.net'
```

Bring the containers down and back up and nginx will successfully start using the certificatres.

## Verify Access

Open up the host in a web browser and it will show the current directoyr listing over https.
