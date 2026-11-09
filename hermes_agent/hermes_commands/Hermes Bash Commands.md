Related to [[Hermes Agent]]
#hermes #hermes_commands_bash

Start the agent:

```bash
hermes
```

Switch to proper `systemd` user session: 


systemctl --user daemon-reload


```bash
without root
machinectl shell hermes@

with root 
sudo machinectl shell hermes@
```

## TUI 

Move up or down through the output:
``` 
shift + up/down arrow

or 

shift + page up/down
```



**GitHub CLI**

1. Determine your current branch name (e.g. `mybranch`):

   ```bash
   git rev-parse --abbrev-ref HEAD
   ```

2. Change the repo’s default branch:

   ```bash
   gh repo edit --default-branch mybranch
   # or, if you’re not in the repo dir:
   gh repo edit USER/REPO --default-branch mybranch
   ```

**Web UI**

1. In GitHub, go to **Settings → Branches**.  
2. Under **Default branch**, click the pencil icon, select your branch from the drop‑down, and save.

Both methods require that your GitHub authentication has permission to edit repository settings.

❯ I need you to remember those instructions. Also create a create/read/write skill and remember to invoke that skill according to these ins
instructions
