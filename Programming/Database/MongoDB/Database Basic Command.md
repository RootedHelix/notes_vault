1) Prerequisites
•
Homebrew installed: brew -v
•
Update Homebrew:
`brew update`
2) Add MongoDB’s official tap
`brew tap mongodb/brew`
3) Install the latest MongoDB Community server
•
Latest (as of 2025, 8.0 series):
`brew install mongodb-community@8.0`
•
Or install a specific version (example 7.0):
brew install mongodb-community@7.0
4) Install the MongoDB Shell (mongosh)
brew install mongosh
5) Start MongoDB as a background service

• **Start now and on login:**
brew services start mongodb-community@8.0

- **Stop or restart later**:

`brew services stop mongodb-community@8.0`
`brew services restart mongodb-community@8.0`

6) Verify it’s running
`brew services list`
You should see mongodb-community@8.0 started.
7) Connect with mongosh
mongosh
Default URI: mongodb://127.0.0.1:27017.
Paths and data locations
•
Apple Silicon (M1/M2/M3): /opt/homebrew/var/mongodb
•
Intel macOS: /usr/local/var/mongodb
•
Logs: typically in /opt/homebrew/var/log/mongodb (ARM) or /usr/local/var/log/mongodb (Intel)
Uninstall / Cleanup
brew services stop mongodb-community@8.0
brew uninstall mongodb-community@8.0
brew untap mongodb/brew   # optional
Remove data (optional and destructive):
rm -rf /opt/homebrew/var/mongodb  # ARM
rm -rf /usr/local/var/mongodb     # Intel
Common issues
•
If mongod won’t start, ensure the data dir exists and is writable:
mkdir -p /opt/homebrew/var/mongodb && chmod 700 /opt/homebrew/var/mongodb

(Use /usr/local/var/mongodb on Intel.)
•
If you installed a different version, use that exact suffix in commands (e.g., mongodb-community@7.0).
That’s it—after brew services start ..., you can use mongosh to interact with MongoDB locally.
