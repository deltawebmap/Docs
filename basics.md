# How Delta Web Map works
Delta Web Map works to keep ARK data in sync with a number of servers.

## Writing
Here's the basics on how the ARK server writes data to our database. More docs on this will be added in the future.

```[ARK Server -> DWM Mod] --[HTTP]--> [Delta Sync Server] --[MongoDB]--> [Database]```

More specifically, the mod begins when the ARK server is started by sending a request to ``http://sync-nossl-prod.deltamap.net/config`` to establish options. This endpoint will return a configuration file, but also returns an access token that the mod should use. The access token is saved to the ARK save file. If no access token is set, this endpoint will create a new server in our database.

From there, the mod periodically sends data to the following endpoints using the access token we downloaded with the config file. The config file also sets how frequently this data is sent:

* ``/v1/dinos`` This endpoint is used to transmit dinosaur data, such as tame names, classnames, and stats. This is sent in chunks of 6 to avoid server lag.
* ``/v1/structures`` This endpoint is used to transmit player created structures, such as walls and foundations. This is sent in chunks of 30 to avoid server lag.
* ``/v1/profiles`` This endpoint is used to submit connected players and tribes. This is used to create the "online status" of players, and to give them access to the server using their Steam ID.
* ``/v1/update_revision_id`` This endpoint is used whenever a new batch of dinos or structures is sent. This endpoint removes the data of an old batch. This is used to clean up killed dinosaurs or destroyed structures.

## Reading
Delta Web Map has a few servers used for reading content. With all of these servers, we use the Steam ID of the DWM user they used to sign in and search for profiles using that Steam ID sent by the ARK server mod in the above section. 

When we find a matching profile, we can extract the tribe ID of the user from it. Using that tribe ID, we can load additional tribe content from the database.

More info will be added in the future about this.
