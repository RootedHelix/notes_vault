Related to [[Hermes Agent]],  
#setup_agent #hermes

# Hermes Agent on Arch Linux — Separate User, Local RTX 5080 Model, and Desktop Output

**Target host:** Arch Linux (`archboxx`)  
**Personal account:** `helix`  
**Agent account:** `hermes`  
**GPU:** NVIDIA MSI RTX 5080, 16 GB VRAM  
**Inference:** Ollama on the host, accessible through loopback  
**Agent:** Hermes Agent, running under `hermes`  
**File handoff:** `/home/helix/Desktop/HermesOutput`

> Run commands under the account named in each heading. Review shell commands before executing them. These instructions separate users on one Linux installation; they do not provide VM-level isolation. **Do not grant the agent sudo, access to your personal SSH keys, or membership in the rootful `docker` group.**

## Part 1 — Create a dedicated Linux user

### 1. Check your current account and GPU (`helix`)

```
whoami
nvidia-smi
```

Expected username: `helix`. Ensure `nvidia-smi` reports the RTX 5080 correctly. Fix any NVIDIA NVML driver/library mismatch before proceeding.

### 2. Create the user (`helix`)

```
sudo useradd --create-home --shell /bin/bash hermes
sudo passwd hermes
sudo chmod 700 /home/hermes
id hermes
```

Choose a unique password. **Do not** add `hermes` to `wheel`, `sudo`, or `docker`.

Verify its groups:

```
id -nG hermes
```

### 3. Inspect and preserve the personal-home ACL (`helix`)

```
ls -ld /home/helix /home/helix/.ssh 2>/dev/null
getfacl /home/helix
```

**Observed on this host:**

```
# owner: helix
# group: helix
user::rwx
user:http:--x
group::---
mask::--x
other::---
```

The existing `http` ACL allows traversal only (not directory listing). **Leave this ACL in place. Do not blindly run `chmod 700 /home/helix`**: that could change the ACL mask and disable the existing service-user access. General access by other users is already denied. Later, add a named `hermes` ACL entry specifically for desktop file handoff, preserving `http`.

The `.ssh` directory was reported as `drwx------`; leave its permissions unchanged.

### 4. Create a workspace (`helix`)

```
sudo install -d -o hermes -g hermes -m 700 /home/hermes/workspace
```

### 5. Test account separation (`helix`)

```
sudo -iu hermes
whoami
printf '%s\n' "$HOME"
ls /home/helix  # Expected: Permission denied
exit
```

If `ls /home/helix` succeeds, inspect group memberships, directory permissions and ACLs before proceeding.

---

## Part 2 — Install Ollama for your RTX 5080

### 6. Install the Arch packages (`helix`)

```
sudo pacman -Syu ollama ollama-cuda
sudo systemctl enable --now ollama.service
systemctl status ollama.service --no-pager
```

Arch packages and service names can change; see the current [ArchWiki Ollama page](https://wiki.archlinux.org/title/Ollama).

### 7. Verify local access (`helix`)

```
curl -fsS http://127.0.0.1:11434/api/tags
```

The Ollama API should stay on `127.0.0.1:11434`. Do not expose its unauthenticated interface to the LAN or Internet.

### 8. Download and test GPT-OSS 20B (`helix`)

```
ollama pull gpt-oss:20b
ollama run gpt-oss:20b
```

Type a short test prompt. Leave the Ollama interactive prompt with `/bye`. Verify GPU placement in another terminal:

```
ollama ps
nvidia-smi
```

**Verified on this host:** `gpt-oss:20b` was reported at `100% GPU` on the RTX 5080, using the `llamacpp` runner.

### 9. Configure and verify the 64K context window (`helix`)

Since `systemctl edit` had no default editor configured, explicitly use Neovim:

```
sudo SYSTEMD_EDITOR=nvim systemctl edit ollama.service
```

Put the following in the **editable override section**, above the comment indicating edits below will be discarded:

```
[Service]
Environment="OLLAMA_CONTEXT_LENGTH=65536"
```

Do **not** overwrite the packaged unit or remove its existing `Environment="HOME=/var/lib/ollama"` entry. Save Neovim with `Esc`, `:wq`, Enter.

```
sudo systemctl restart ollama.service
systemctl show ollama.service -p Environment
ollama run gpt-oss:20b
```

In another terminal:

```
ollama ps
nvidia-smi
```

**Observed result on this machine:**

```
NAME           SIZE     PROCESSOR    CONTEXT    RUNNER
gpt-oss:20b    12 GB    100% GPU     65536      llamacpp
```

`nvidia-smi` showed **14,068 MiB / 16,303 MiB** in use, leaving **2,235 MiB** free at that moment. A 64K allocation is verified, though this alone does not benchmark prompts filled to 64K tokens. The host has **30 GiB usable RAM** (approximately 32 GB installed) and no swap.

---

## Part 3 — Install and configure Hermes as its own user

### 10. Enter the agent account (`helix`)

```
sudo -iu hermes
```

All steps in this section run **as `hermes`** unless explicitly stated otherwise.

### 11. Install Hermes (`hermes`)

Use the official installation instructions from [Hermes Agent](https://hermes-agent.nousresearch.com/). The published bootstrap command is:

```
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

**Security:** This executes a remote script under `hermes`. For greater assurance, download and inspect the script before executing it. If `hermes` is not found after installation, open a new login shell or follow the installer's PATH instructions.

```
hermes --help
hermes doctor
```

### 12. Connect Hermes to local Ollama (`hermes`)

During Hermes **Full Setup**, select:

|Prompt|Selected setting|
|---|---|
|Model provider|**Custom endpoint (enter URL manually)**|
|Base URL|`http://127.0.0.1:11434/v1`|
|API compatibility|**2. Chat Completions**|
|Detected model|`gpt-oss:20b` (confirmed)|
|Context length|`65536`|
|Display name|Any descriptive label, e.g. `GPT-OSS 20B (Local RTX 5080)`|
|Reasoning effort|**medium**|
|Terminal backend|**Docker / Podman** after rootless Docker verification in Part 5|
|Egress credential firewall|Skipped during initial setup|
|Messaging platforms|Skipped|
|Gateway startup/autostart|**No** / **No**|
|Shared metrics|**No thanks**|

The messaging-platform step was cancelled without changing the earlier sections. The CLI subsequently launched as **Hermes Agent v0.22.2**, displaying `gpt-oss:20b` and a 65.5K context window. No gateway is required for direct terminal use.

To inspect saved settings, use:

```
ls -la ~/.hermes
nvim ~/.hermes/config.yaml
```

Preserve the setup wizard's model configuration rather than replacing it with guessed YAML keys.

### 13. Smoke-test the local agent (`hermes`)

```
hermes
```

Ask it to identify its configured model, explain a small C function, and create a harmless file in its own workspace. Use `hermes doctor` if it cannot connect.

**Important distinction:** The **Hermes process** runs as the `hermes` user. When its terminal backend is set to `local`, shell commands executed by the agent also run as `hermes`. This gives user-level protection but is **not sandboxing**.

---

## Part 4 — Prepare desktop output permissions (optional; not yet verified)

Commands in this section run as **`helix`** in a separate terminal or after exiting the `hermes` shell.

### 14. Install ACL tools and create output folder (`helix`)

```
sudo pacman -S acl
mkdir -p /home/helix/Desktop/HermesOutput
```

If the Desktop folder does not exist under Hyprland, `mkdir -p` creates it; a Wayland compositor does not necessarily render files as desktop icons.

### 15. Grant only necessary traversal/write rights (`helix`)

```
sudo setfacl -m u:hermes:--x /home/helix
setfacl -m u:hermes:--x /home/helix/Desktop
setfacl -m u:hermes:rwx /home/helix/Desktop/HermesOutput
setfacl -d -m u:hermes:rwx /home/helix/Desktop/HermesOutput
```

`--x` allows traversal of a directory when the exact path is known, not listing its contents. The ACL on `HermesOutput` grants access **only inside that folder**, assuming no other permissions or mounts grant more access. If the folder already contains files, existing file ACLs are not automatically changed.

### 16. Test both write and isolation (`helix`)

```
sudo -u hermes sh -c 'printf "Hermes output test\n" > /home/helix/Desktop/HermesOutput/test.txt'
cat /home/helix/Desktop/HermesOutput/test.txt
sudo -u hermes ls /home/helix/Documents  # Should fail, assuming Documents is private
```

Review the ACLs:

```
getfacl /home/helix /home/helix/Desktop /home/helix/Desktop/HermesOutput
```

**Note on ownership:** New files normally belong to `hermes` (not `helix`), although `helix` should have access through folder permissions and inherited ACLs. If you need `helix` to modify files created by Hermes, test that explicitly; some creation modes or application behavior can affect inherited ACLs.

### 17. Tell the agent where to save outputs

With the **local terminal backend** (not the Docker backend selected in this setup), you can instruct Hermes:

> Save final deliverables into `/home/helix/Desktop/HermesOutput/`. Do not write anywhere else under `/home/helix`.

The instruction is a workflow convention, not a security boundary. The ACLs enforce host-user access limits. **For the selected Docker backend, use `/output` after completing the optional mount in Part 5.**

---

## Part 5 — Rootless Docker for Hermes (completed and verified)

This is the **actual workflow used on `archboxx`**, not an optional untested alternative. Commands are labeled by account. Rootless Docker limits the daemon's host privileges, but is **not equivalent to a VM**.

### 18. Install Docker on Arch (`helix`)

```
sudo pacman -Syu docker
```

On this Arch installation, `docker-rootless-extras` was **not available from the official Pacman repositories**:

```
error: target not found: docker-rootless-extras
```

Install the rootless extras package from the **AUR**, for example (if `yay` is installed):

```
yay -S docker-rootless-extras
```

Review AUR build scripts before installation. If it is already installed, do not reinstall it. Check the tools and the user-service unit:

```
docker --version
command -v dockerd-rootless.sh
command -v newuidmap
command -v newgidmap
ls /usr/lib/systemd/user/docker.service
```

The user-service unit was found on this machine at `/usr/lib/systemd/user/docker.service`.

**Do not** add `hermes` to the `docker` group or enable the rootful `/var/run/docker.sock` daemon for agent use.

### 19. Verify subordinate UID/GID mappings and enable lingering (`helix`)

```
grep '^hermes:' /etc/subuid /etc/subgid
sudo loginctl enable-linger hermes
```

The `hermes` user needs distinct, nonoverlapping subordinate UID and GID ranges (commonly 65,536 IDs each). If missing, allocate free ranges using `usermod --add-subuids START-END` and `--add-subgids START-END` as root; **do not copy an arbitrary example range**. Lingering allows the user-level service to remain active after the interactive session closes.

### 20. Open a proper Hermes systemd session (`helix` → `hermes`)

`systemd-container` is **not a Pacman package on this Arch system**; the attempt returned `target not found`. Instead, the available `machinectl` command was used:

```
command -v machinectl
sudo machinectl shell hermes@
```

Inside the new session:

```
whoami
echo "$HOME"
echo "$XDG_RUNTIME_DIR"
```

**Observed:** `hermes` and `/run/user/1001` for the runtime directory. An existing `sudo -iu hermes` shell and a `machinectl` shell are both the same user and share files, but the latter established the intended systemd user-service environment.

### 21. Start and verify the rootless daemon (`hermes`, machinectl session)

```
systemctl --user enable --now docker.service
systemctl --user status docker.service --no-pager
```

Check the rootless socket explicitly:

```
ls -l "$XDG_RUNTIME_DIR/docker.sock"
DOCKER_HOST="unix://$XDG_RUNTIME_DIR/docker.sock" docker info
```

**Observed socket:** `/run/user/1001/docker.sock`, owned by `hermes`. The numeric group identifier shown in its listing is not, by itself, evidence of a problem.

### 22. Create and use a named Docker context (`hermes`)

```
docker context create hermes-rootless \
  --docker "host=unix://$XDG_RUNTIME_DIR/docker.sock"
docker context use hermes-rootless
```

Do not recreate the context if it already exists. Verify:

```
docker context show
docker context inspect hermes-rootless --format '{{.Endpoints.docker.Host}}'
docker info --format '{{json .SecurityOptions}}'
```

**Actual confirmed output:**

```
hermes-rootless
unix:///run/user/1001/docker.sock
["name=seccomp,profile=builtin","name=rootless","name=cgroupns"]
```

This proves the selected daemon is rootless. As `hermes`, the following also succeeded:

```
docker run --rm hello-world
```

### 23. Configure Hermes's Docker backend (`hermes`)

In Hermes Full Setup, **Docker / Podman** was selected and detected `/usr/bin/docker`. The egress credential firewall was skipped. This was an explicit local choice, not evidence that outbound network access is blocked.

If checking or changing the configuration later:

```
nvim ~/.hermes/config.yaml
```

Merge these settings into any existing `terminal:` section (do not duplicate YAML keys):

```
terminal:
  backend: docker
  docker_forward_env: []
  container_persistent: true
```

Prefer launching the agent from the `machinectl` Hermes session so its Docker CLI uses the correct user context:

```
docker context show
hermes
```

### 24. Verify the actual running Hermes sandbox (`hermes`, host shell)

From the **host's `hermes` shell**, not from the Hermes chat prompt:

```
docker ps -a
```

**Observed container:**

```
IMAGE                                 COMMAND            STATUS    NAME
nousresearch/hermes-sandbox:desktop   "sleep infinity"   Up        hermes-0d41b8b8
```

The `sleep infinity` command keeps the persistent sandbox container alive. Verify command execution from the **host**:

```
docker exec hermes-0d41b8b8 whoami
```

**Observed output:** `root`. In rootless Docker, root inside the container does **not** mean host root. When `docker exec ...` was sent to the **Hermes chat interface**, it failed with exit 127 because the Docker CLI is not installed inside the sandbox. That is expected; the host controls Docker, not the agent container.

### 25. Inspect container mounts and isolation (`hermes`, host shell)

```
docker inspect hermes-0d41b8b8 --format '{{json .Mounts}}'
```

The observed bind mounts all originated under `/home/hermes/.hermes/...`:

|Host source|Container path|Access|
|---|---|---|
|`.hermes/sandboxes/docker/default/home`|`/root`|RW|
|`.hermes/sandboxes/docker/default/workspace`|`/workspace`|RW|
|`.hermes/attachments`, `composer-pastes`, `images`, `skills`|Under `/root/.hermes/`|RO|
|`.hermes/cache/{audio,delegation,documents,generated,images,screenshots,spillover,videos,web}`|Under `/root/.hermes/cache/`|RO|

**No `/home/helix` bind mount was present** in the inspected configuration. The list is evidence about mounts, not proof against every possible path of access. Optional additional checks:

```
docker exec hermes-0d41b8b8 ls -ld /home/helix
docker exec hermes-0d41b8b8 ls -l /var/run/docker.sock
```

Both should fail if those paths are not exposed. Do not mount the host Docker socket into the sandbox.

### 26. Important troubleshooting distinction (`helix` versus `hermes`)

From `helix`, plain `docker ps -a` initially failed trying to access `/var/run/docker.sock`, because the rootless daemon is **not** the host's default rootful daemon. Run Docker management commands from the `hermes` machinectl session with `hermes-rootless` selected.

If a terminal-tool request stalls, inspect the container on the host using:

```
docker ps -a
docker images
journalctl --user -u docker.service -n 30 --no-pager
```

A slow AI response to `whoami` does not imply the command itself is slow; model reasoning and agent orchestration add latency. Verify the command independently via `docker exec` on the host.

### 27. Optional desktop handoff mount (not yet verified)

The earlier Part 4 grants the **host** user `hermes` narrowly scoped ACL access to `/home/helix/Desktop/HermesOutput`. With Docker, an **additional bind mount** is required. Only after creating the folder and testing the ACLs, add a volume entry to the existing Hermes `terminal:` section:

```
terminal:
  backend: docker
  docker_forward_env: []
  container_persistent: true
  docker_volumes:
    - "/home/helix/Desktop/HermesOutput:/output"
```

A rootless daemon uses user namespaces, so **successful host-user ACL tests do not guarantee that the container can write the mounted directory**. Recreate the sandbox if necessary for new volumes to take effect, then test from inside Hermes:

```
printf 'sandbox write test\n' > /output/from-hermes.txt
```

As `helix`:

```
cat /home/helix/Desktop/HermesOutput/from-hermes.txt
```

If access fails, inspect UID mapping and ACLs; never solve it by exposing your entire home, making the container privileged, or sharing the rootful Docker socket. Keep `/output` limited to agent deliverables. **This mount/test was planned but not demonstrated in the session recorded here.**

---

## Part 6 — Permissions, maintenance, and rollback

### Revoke the desktop output access (`helix`)

Stop Hermes and its running container first; otherwise an existing container can retain access to an established mount. **The desktop bind mount had not been verified during the recorded setup.**

```
setfacl -x u:hermes /home/helix/Desktop/HermesOutput
setfacl -d -x u:hermes /home/helix/Desktop/HermesOutput
setfacl -x u:hermes /home/helix/Desktop
sudo setfacl -x u:hermes /home/helix
```

Only the Hermes-specific ACLs are removed; other default ACL entries remain intact. Revocation of access to _existing files_ should be checked separately if you granted permissions recursively in the future.

### Verify the agent never acquired privileged groups (`helix`)

```
id hermes
sudo -l -U hermes
```

If the sudo check shows no permitted commands and `id` shows no privileged groups, that is the intended baseline. Distribution-wide sudo configuration should still be reviewed if changed later.

### Common problems

|Symptom|Check|
|---|---|
|`nvidia-smi` fails|NVIDIA kernel module and userspace versions; reboot after upgrades if needed|
|Ollama uses CPU|`ollama ps`, `nvidia-smi`, CUDA backend package and driver|
|Hermes cannot connect|`curl http://127.0.0.1:11434/v1/models` as `hermes`, `hermes doctor`|
|Slow performance or out-of-memory|Lower context length; use 9B model; inspect GPU vs CPU allocation|
|Can't save to Desktop|Directory traversal ACLs, output directory ACL, container mount|
|Docker can't start|Rootless daemon status and `docker info` under `hermes`|
|Docker writes wrong file ownership|Rootless UID mapping, `docker_run_as_host_user`, bind-mount ACLs|

## Recommended steady-state layout

```
Arch Linux
├── user: helix
│   ├── private development environment
│   └── ~/Desktop/HermesOutput/   (explicit agent write access)
├── user: hermes                (unprivileged)
│   ├── ~/.hermes/              (config, sessions, memory)
│   └── ~/workspace/            (agent-owned work)
├── system service: ollama      (loopback API, shared RTX 5080)
└── rootless Docker under hermes (verified)
    └── /output  →  /home/helix/Desktop/HermesOutput
```

## References

- [Hermes Agent installation and official site](https://hermes-agent.nousresearch.com/)
- [Hermes Agent — LLM and Model Providers](https://hermes-agent.nousresearch.com/docs/integrations/providers)
- [Hermes Agent — Configuration / Docker terminal backend](https://hermes-agent.nousresearch.com/docs/user-guide/configuration/)
- [ArchWiki — Ollama](https://wiki.archlinux.org/title/Ollama)
- [Docker — Rootless mode](https://docs.docker.com/engine/security/rootless/)
- [ArchWiki — Docker](https://wiki.archlinux.org/title/Docker)
- [Docker — Rootless mode tips and setup](https://docs.docker.com/engine/security/rootless/)