# Quadlet Deployment (Bazzite / rootless Podman)

## What these files are

- `cli-proxy-api.container`: Quadlet unit for the service.
- `cliproxy.env.example`: Environment variable template for optional remote stores and management password.

## One-time setup

1. Create runtime directories:

```bash
mkdir -p ~/.config/cliproxy ~/.local/share/cliproxy/{auths,state}
```

2. Copy config and env templates:

```bash
cp ./config.example.yaml ~/.config/cliproxy/config.yaml
cp ./quadlet/cliproxy.env.example ~/.config/cliproxy/cliproxy.env
chmod 600 ~/.config/cliproxy/cliproxy.env
```

3. Install the Quadlet file:

```bash
mkdir -p ~/.config/containers/systemd
cp ./quadlet/cli-proxy-api.container ~/.config/containers/systemd/
```

4. Reload and start user service:

```bash
systemctl --user daemon-reload
systemctl --user start cli-proxy-api.service
loginctl enable-linger "$USER"
```

## Operations

```bash
systemctl --user status cli-proxy-api.service
systemctl --user restart cli-proxy-api.service
journalctl --user -u cli-proxy-api.service -f
podman ps --filter name=cli-proxy-api
```

## Notes

- Ports are bound to `127.0.0.1` by default for safer local use.
- To expose on LAN, edit `PublishPort=` lines and replace `127.0.0.1` with `0.0.0.0`.
- OAuth callback ports (`8085`, `1455`, `54545`, `51121`, `11451`) are included because this repo uses local callback listeners during provider login flows.
- Image is digest-pinned; update by editing the `Image=` line in `cli-proxy-api.container` to a new digest.
- Hardening enabled in unit: dropped capabilities, `NoNewPrivileges`, read-only rootfs, tmpfs for `/tmp`, and conservative service limits (`MemoryMax=768M`, `CPUQuota=100%`, `TasksMax=256`).
- If `systemctl --user enable ...service` says the unit is transient/generated, that is expected with Quadlet-generated `.service` units. Keep `[Install]` in the `.container` file and use `start`/`restart` after `daemon-reload`.
