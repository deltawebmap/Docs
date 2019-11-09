# How to set up DWM locally
Before we begin, I ask that you let me know before you set up any production servers using my code. You're free (and would be greatly appreciated) if you want to contribute to the project, however! That's what this is for!

Here are some things you'll need before we get started:
* .NET SDK
* A [MongoDB](https://www.mongodb.com/) server

## Understanding the parts of the project
* [Lib Delta System](https://github.com/deltawebmap/LibDeltaSystem) is the heart of the platform. It contains functions for connecting to the database, authenticating users, and much much more. It's required for all backend servers.
* The [DWM Master Server](https://github.com/deltawebmap/Delta-Backend/tree/master/backend/ArkWebMapMasterServer) is the main API. It's used to provide user information and write user prefs.
* The [Dynamic Tiles Server](https://github.com/deltawebmap/Delta-Backend/tree/master/backend/ArkWebMapDynamicTiles) is used to provide map tiles for user generated content, such as ARK structures. This is not a required part of the service.
* The [DWM RPC](https://github.com/deltawebmap/RPC/) is used to push events to users. Backend servers connect to the RPC to send messages to frontend clients via a WebSocket.
* [Charlie](https://github.com/deltawebmap/Charlie) is used to process all of the ARK assets and convert them to JSON format. Files are published to https://packages.deltamap.net/index.json
* [Echo](https://github.com/deltawebmap/Echo) is the main content distributor for user data. It serves clients with dinosaurs, items, and all other saved tribe data. This is an important part of the platform.

There's also a couple of frontend repos
* [Web](https://github.com/deltawebmap/Delta-Web-Map-Frontend-JS)
* [Android](https://github.com/deltawebmap/Delta-Android)

## Downloading everything you'll need
You'll need to clone at least the [DWM Master Server](https://github.com/deltawebmap/Delta-Backend/tree/master/backend/ArkWebMapMasterServer), the [DWM Gateway](https://github.com/deltawebmap/Delta-Backend/tree/master/backend/ArkWebMapGateway), and [Echo](https://github.com/deltawebmap/Echo).

You'll also need to clone [Lib Delta System](https://github.com/deltawebmap/LibDeltaSystem) and make sure it is correctly referenced by all of the servers.

## Config
### Database Config File
Every backend server needs access to a database config file. It looks like this:

```json
{
    "env": "staging",
    "rpc_key":"[Base-64 encoded 64 byte key for rpc that you create and keep private]",
    "rpc_port":[RPC server send port],
    "rpc_ip":"[RPC server IP]",
    "server_ip": "[Mongo DB server IP]",
    "server_port": [Mongo DB server port],
    "steam_api_token": "[your Steam api token]",
    "steam_cache_expire_minutes": [How long you wish to keep Steam data, 120 on prod],
    "user":"[MongoDB username]",
    "key":"[MongoDB key that you keep private]",
    "firebase_config":"[Pathname to your Firebase config file]"
}```

Some items on the list are pretty self-explanatory. Here's a few that you need to set:

* ``steam_api_token`` - You'll need to [register an API token with Steam](https://steamcommunity.com/dev/apikey) and paste it here. Keep it a secret!
* ``user`` - This is the username to your MongoDB database
* ``key`` - This is the password to your MongoDB database
* ``server_ip`` - This is the IP address of your MongoDB database (use ``0.0.0.0`` for localhost)
* ``server_port`` - This is the port of your MongoDB database
* ``firebase_config`` - The pathname to a Google Firebase config file. You'll need to obtain this from Google Firebase.
* ``rpc_key`` - This is a unique key that you create to protect communications between RPC and other servers. It MUST be a base 64 encoded string and SHOULD be 64 bytes.
* ``rpc_port`` - The port of your RPC server.
* ``rpc_ip`` - The IP address of your RPC server.

### Map Config File
The map config file defines ARK maps that are supported. You should use mine and update it periodically. You can download it here https://packages.deltamap.net/map_config.json.

You'll need to link to this config file later.

### Master Config File
The master server also requires it's own config file. You should pass the path to this as an argument when starting the program. The file looks like this:

```json
{
    "github_api_key":"[GitHub API Key]",
    "database_config_path":"database_config.json",
    "map_config_path":"map_config.json",
    "listen_port":[Server Port],
    "endpoint_this":"https://deltamap.net/api",
    "endpoint_echo":"https://echo-content.deltamap.net"
}
```

You'll need to set the following values:

* ``github_api_key`` - **[Optional]** Set this if you'd like to be able to allow users to report issues. You shouldn't set this.
* ``database_config_path`` - Set this to the pathname of your database config file you created earlier.
* ``map_config_path`` - Set this to the pathname of the map config file you created earlier.
* ``listen_port`` - The port that your HTTP server should listen on. You should put it behind a proxy, such as apache2.
* ``endpoint_this`` - The URL (without ending slash) that would load the root of this server.
* ``endpoint_echo`` - The URL (without ending slash) that would load the root of the Echo server.

### Dynamic Tiles Config File
TODO!

### Echo Config File
The echo config file requires that you link it to the two other global config files you created earlier. Set the paths here as appropriate.
```json
{
    "database_config_file":"database_config.json",
    "map_config_file":"map_config.json"
}
```

### RPC Config File
You'll also need the config file for the RPC server. It looks like this

```json
{
   "database_config":"database_config.json",
   "key":"[RPC Key you created when making database config]",
   "port":[Port of the HTTP server],
   "buffer_size":8192,
   "timeout_seconds":120
}
```

You'll need to set: 

* ``database_config`` - Set this to the pathname of the database config file you created earlier.
* ``key`` - Set this to the RPC key you created in the database config earlier.
* ``port`` - Set this to the **HTTP server port**, **not** the RPC port.
* ``buffer_size`` - You should keep this at 8192. You may raise it, but please do not lower it.
* ``timeout_seconds`` - How long it would take for a client to be timed out. Feel free to adjust it as needed.

## Launch
Launch all of the services and you should be able to make API requests. To test in the web interface, you'll need to choose a new config file in the frontend.

To this, open ``/assets/app/app.js`` and change the ``CONFIG_URL`` constant near the beginning of the file. Point this to your own copy of the config file.

To create your own copy of the config file, download https://config.deltamap.net/prod/app_config.json and open it in your text editor. You'll need to change *at least* the following

* ``rpc_host`` - Set this to the hostname of your RPC server
* ``api`` - Set this to the root URL of your master server

Some of the other options, such as ``allowed_domains``, aren't enforced now but may in the future.
