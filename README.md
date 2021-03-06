# ExileMineLimiter
ExileMineLimiter is a system built specifically for Exile to limit the amount of mines that can be placed by players. Players can also place persistent __*DEFENSIVE*__ territory mines. There is also a working disarm system to counter the mines. Even more exciting, this makes the Mine Detector work with Exile!


<p align="left">
Here is a video of the system in action.

[![Exile Mine Limiter](https://img.youtube.com/vi/tdxi-jPnRjI/0.jpg)](https://www.youtube.com/watch?v=tdxi-jPnRjI "ARMA 3 | Sneak Peek - Mine Disarming")
</p>

# How It Works

ExileMineLimiter is coded by a "weight" system. Each mine type weighs a certain amount, for example an explosive charge might weigh 10, while a satchel would weight 25. This
allows for flexibility for certain amounts and types of mines instead of just having a max amount. This system works best with RHS. 

When a player sets a mine, the system checks if the mine is an allowed type, then sets the player's total weight. If the player reaches the total max weight, he or she will no longer
be allowed to place mines. If the player is inside their own friendly territory, the mine is persistent and will be respawned on restart if not triggered. Players will be given
ownership of the mine if they die or reconnect so it fixes the issue where you lose ownership of the mine after you die or log out. 

Mine Detectors also work with this system! Simply add them to your traders and loot tables, and it will instantly start picking up the mines on the radar. This is because the mines are spawned by the server
instead of by the client.


# Server

#### SQL
1. In your ExileMineLimiter folder, go into the DB folder.
2. Double click the .sql file to open the file in MySQL workbench to insert the changes to your Exile SQL database.
3. Or if you are not using MySQL Workbench, copy/paste the query inside your own SQL table viewer.


### ExileServer
1. Open the ExileServer\addons folder and copy the MineLimiter_Server.pbo into your addons folder in your ExileServer\addons folder.
2. Copy all contents from exile.ini inside the sql_custom folder and paste into your exile.ini near the bottom.

# MPMission

List of files that will need to be merged:
1. init.sqf
2. initPlayerLocal.sqf
3. initServer.sqf
4. config.cpp
5. CfgSounds.hpp
6. CfgNetworkMessages.cpp
7. CfgExileCustomCode.cpp
8. CfgExileDelayedActions.cpp
9. custom\MineLimiter_Client\overrides\ExileClient_action_execute.sqf

### init.sqf

This initiates the server task to check all mines on the map, to compare them against mines that no longer exist. Then sets player mine weights, and calls necessary server functions.


Ensure that you have this line at the top
````
[] execVM "custom\initCustomFile.sqf";
````

### initPlayerLocal.sqf

Defines the function to get closest proximity bomb, as well as initiates the client-sided MineLimiter functions.

````
//Put this near the top
[] execVM "custom\MineLimiter_Client\MineLimiter_Client_init.sqf";


//What this does is it returns the closest mine/bomb in proximity to the player so they can disarm it
fnc_getClosestBomb = {
    private _MineBase = nearestObject [player, "MineBase"];
    private _PipeBombBase = nearestObject [player, "PipeBombBase"];
    private _TimeBombCore = nearestObject [player, "TimeBombCore"];

    private _closestMine = [[_MineBase,_PipeBombBase,_TimeBombCore], [], { player distance _x }, "ASCEND"] call BIS_fnc_sortBy;

    _closestMine select 0
};
````

### initServer.sqf

Initiates the addAction handler for all players.

````
waitUntil { time > 0 };
enableEnvironment [true, false];

[] call ExileServer_MineLimiter_disarmMine_addActionHandler;
````

### config.cpp

Contains the CfgMine class.
Here are the editable settings:

````

	//Max Weight per player
	maxWeight = 150;
	
	//If for whatever reason no weight can be found, use this number instead of null
	defaultWeight = 10;

	
	allowDisarming = true; //Do we want to allow disarming mines with a tool?
	disarmingRequiresTool = true; //Should disarming mines require an item in inventory?
	disarmingTool = "Exile_Item_Pliers"; //Item Id
	disarmActivationDistance = 4; //Distance in meters, play around with it to get the right setting. If you set it too close, the mine might go off.
	disarmRequiresLineOfSight = true; //Pretty self-explanatory

	territoryOnly = false; //Should mines only be allowed to be placed inside territories?

	//Sound ID for disarming
	disarmingSound = "mine_dismantling";

	//Wanna troll some friends and have it blow up automatically??
	BlowUpOnUID[] = {};
````

### CfgExileCustomCode.cpp

These are the current files that are overriden by my system:

```
//Mine limiter - Client
ExileClient_object_player_event_onFired = "custom\MineLimiter_Client\overrides\ExileClient_object_player_event_onFired.sqf";
ExileClient_action_execute = "custom\MineLimiter_Client\overrides\ExileClient_action_execute.sqf";

//Mine limiter - Server
ExileServer_world_initialize = "overrides\ExileServer_world_initialize.sqf";
ExileServer_object_player_createBambi = "overrides\ExileServer_object_player_createBambi.sqf";
ExileServer_object_player_database_load = "overrides\ExileServer_object_player_database_load.sqf";

```

### Feel free to contact me on discord if you have any issues: YouViolateMe#2297
