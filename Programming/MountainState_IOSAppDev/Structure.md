

## Login Structure


Employees will be emailed account creation link to a webpage. Employees will be prompted on the webpage to create an account with specific username and password parameters. Employees will wait to be approved or denied access entry into the system. 
[Further development of deny case] Once they are approved they will be allowed entry into the system and be emailed a link to to download the app. Once reaching the webpage they will be able to download the app. 

App May Have Not Sign in - If the app has a sign in upon launch there could potentially be issues with user's accessing vital job information. Due to the nature of employee work ie working outside in areas where wifi and cellular data is limited, placing a gate on their tool might make for hinderances. 



or May - Having a sign in on app secures the database more tightly adding a further layer of protection for sensitive data.

**Caching Credentials for Offline Use**:

Instead of removing the sign-in completely, you can allow users to **sign in once when they have connectivity**, and then cache their credentials locally for future offline use. This allows employees to continue using the app even when they have no internet access. The app can securely store a session token or credentials using **iOS Keychain**, and periodically attempt to sync with the server when connectivity is available.
