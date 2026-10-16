what if the chrome is set to listen on http://localhost:9222

Set Up configuration with "Debug Javascript"

set the url to: http://localhost:9222/DungeonChampion/index.html

Edit the chrome configuration with pencil, Set command line option: --remote-debugging-port=9222


Use custom user directory: /private/tmp/chrome_dev_test

open -na "Google Chrome" --args --remote-debugging-port=9222 --user-data-dir="/tmp/chrome_dev_test" &
