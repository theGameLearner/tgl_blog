---
{"dg-publish":true,"permalink":"/all-published-notes/unity/networking/photon/pun/pun-basic/"}
---

## PUN
Proton's Pun is no longer supported. Use the updated options if possible.

### Config
Pun Wizard -> Locate Photon Server Settings -> set up app id and chat ID if applicable

### Connecting to Server

1. connection is established by `PhotonNetwork.ConnectUsingSettings();` and it will trigger a callback from photon server to any class that inherits from `MonoBehaviourPunCallbacks` in methods `OnConnectedToMaster()`. We can then join a room and start multiplayer experience.
2. if you loose connection you get a callback in `OnDisconnected(DisconnectCause cause)` with the reason, we need to handle this.
3. when you join a room, `PhotonNetwork.LocalPlayer.ActorNumber` tells us what number did we join and is constant as long as we do not loose the connection. (on reconnecting, this might change)
4. when other player joins a room we get callback on `OnPlayerEnteredRoom()`

### Lobby Data

1. When you join a room, some properties can be shown in Lobby so that others can use it to find rooms to join, these are created by `RoomOptions.CustomRoomPropertiesForLobby` when creating room and set by `RoomOptions.CustomRoomProperties` as a Hash-table.
2. When you want all connected Users ID to be visible outside in lobby, you can set `RoomOptions.PublishUserId` to true, and in c#, you can access this user ID using `Player.UserId`.

### Ways to Sync Data

There are 3 ways to sync data :
1. object sync(transform, animation, component ) mostly via PhotonView
2. Remote Procedure Sync (method calls from client which is forwarded to all other clients in a room) - RPC calls defined in photon project settings scriptable objects under RPC List
3. Player custom Properties
4. we can also raise an event in the network so that our own client and all other client get the event data without making extra RPC calls, this is achieved by `IOnEventCallback` class and we can raise events by `PhotonNetwork.RaiseEvent()` method.
	1. PhotonNetwork.RaiseEvent takes a byte variable as a event code, so we can define it in constants.
	2. All data is converted into array of object (object[]) and passed as such
	3. PhotonNetwork.RaiseEvent() also takes in `RaiseEventOptions` and `SendOptions` to determine who receives this event and what method of sending the event do we employ.
	4. The data is received by all classes that are marked as callback targets by using `PhotonNetwork.AddCallbackTarget()` method. (can use `PhotonNetwork.RemoveCallbackTarget` to remove this)
	5. on being marked as callback target on a network, the events are auto received by `OnEvent()` method. we extract and compare the event code to determine how to process this data.

### Syncing data

1. to sync scene data, we set `PhotonNetwork.AutomaticallySyncScene` to true
2. If we add a `PhotonView` to a gameObject, photon identifies it as a network object to sync between players.
3. `RPC` calls are handled by PhotonView and not by classes which have RPC methods defined in them
4. methods that can be called by other clients have an attribute called `[PunRPC]`, this helps us recognise a method which may be called by the network
5. calls to RPC methods are made by `photonView.RPC()` method, where we pass name of the method, the receivers of the call and the data
6. If we instantiate a `PhotonView` gameObject, photon will instantiate it for all other clients in the network. Hence, any instantiate code should be written as if we will instantiate only that one object without considering other objects.
	1. similarly, do not create a enemy client object in games as they will be generated automatically.
	2. the actual instantiation can be done by `PhotonNetwork.Instantiate()` method.
	3. Instantiated object should be destroyed by using `PhotonNetwork.Destroy()`.
7. when we leave a room, all of our data will be destroyed from the room, we can disable or enable this as per our game logic in `RoomOptions.CleanupCacheOnLeave`.
8. When you want to load the game scene, load the scene by using `PhotonNetwork.LoadLevel()` after ensuring the `PhotonNetwork.AutomaticallySyncScene` is set to true, else objects may be destroyed while changing the scene.