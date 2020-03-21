Examples
======
### Design and Layout
Let's say a quest had ID `1`, and required `Required Item 1 x 10` and `Required Item 2 x 5` which were dropped by `Monster 1` and `Monster 2` respectively in a room called `example`. The reward for the quest is called `Reward Item`. A typical script to complete this quest once would look like this:

```csharp
using RBot;

public class Script
{
	public void ScriptMain(ScriptInterface bot)
	{
        // Setup options.
        // Recommend that these two options are always enabled.
        bot.Options.SafeTimings = true;
        bot.Options.RestPackets = true;
        
        // Setup skills
        // This can also be done in the skills UI, but bot.Skills.StartTimer() must still be called.
        bot.Skills.LoadSkills("Skills/VoidHighlord.xml");
        bot.Skills.StartTimer();

        // Load the bank.
        bot.Player.LoadBank();
        // Saving inventory space, this also gives the bank time to load.
        bot.Inventory.BankAllCoinItems();

        // Move any items that may be required for the quest into inventory.
        bot.Bank.ToInventory("Required Item 1");
        bot.Bank.ToInventory("Required Item 2");

        // Accept the quest. Here id 1 is used.
        bot.Quests.EnsureAccept(1);

        // Join room with required monsters.
        bot.Player.Join("example");
        
        // Hunt monsters for quest items.
        bot.Player.HuntForItem("Monster 1", "Required Item 1", 10);
        bot.Player.HuntForItem("Monster 2", "Required Item 2", 5);

        // Turn in quest and get reward.
        bot.Player.EnsureComplete(1);
        bot.Wait.ForDrop("Reward Item");
        bot.Player.Pickup("Reward Item");
	}
}
```

#### Repeating the Quest
To repeat the quest and complete it indefinitely, we can wrap the quest part of the script in a while loop:

```csharp
while(!bot.ShouldExit()){
    bot.Quests.EnsureAccept(1);

    bot.Player.Join("example");

    //... hunting code

    bot.Player.EnsureComplete(1);
    bot.Wait.ForDrop("Reward Item");
    bot.Player.Pickup("Reward Item");
}
```

This will complete the quest until RBot is closed. The condition in the while loop can be changed to anything, such as checking wheter a player has a specified quantity of item.

#### Hunting Multiple Monsters
The easiest way to hunt multiple monsters for an item is to separate the monster names with a `"|"` character and pass it as the monster's name to `HuntForItem`. For example, if multiple monsters `Monster 1`, `Monster 2` and `Monster 3` drop the item `Item 1`, and these monsters exist in the same room (can be in different cells), you can hunt them to get `Item 1 x 10` like this:

```csharp
bot.Player.HuntForItem("Monster 1|Monster 2|Monster 3", "Item 1", 10);
```

If `Item 1` is a temp item, you can simply pass another argument after the quantity, `true` indicating this. This argument defaults to false:

```csharp
bot.Player.HuntForItem("Monster 1|Monster 2|Monster 3", "Item 1", 10, true);
```

#### Hunting for Multiple Items
If you want to hunt a monster or multiple monsters for multiple items, you can use `HuntForItems`. Multiple monsters are passed to this method in the same way as for `HuntForItem` (using a `"|"` separator). For example, if you want `Item 1 x 10` and `Item 2 x 5`, and all three of of the monsters drop these 2 items, you would use this:

```csharp
bot.Player.HuntForItems("Monster 1|Monster 2|Monster 3", new string[] { "Item 1", "Item 2" }, new int[] { 10, 5 });
```

Sometimes the script compiler doesn't like arrays so you have to use lists and convert them to arrays like this:

```csharp
bot.Player.HuntForItems("Monster 1|Monster 2|Monster 3", (new List<string>() { "Item 1", "Item 2" }).ToArray(), (new List<int>() { 10, 5 }).ToArray());
```

This is a glitch in the scripting library RBot uses and this is the only workaround I know for now.

#### Joining Glitched Rooms
Typically, on well populated servers, any room which is commonly farmed at for quests will be glitched at room 9999. If you would like to join these rooms you can use `JoinGlitched` instead of `Join`:

```csharp
bot.Player.JoinGlitched("judgement");
```

#### Setting Up Relogin
I would recommend you setup the auto relogin in the UI as it is easier, although you can do it in your script if you want. Add this code to where you setup your options:

```csharp
bot.Options.AutoRelogin = true;
bot.Options.LoginServer = ServerList.Servers.Find(x => x.Name == "Artix");
```

You can change Artix to whatever server you want to relogin to.

#### Legion Fealty 1
Here I will build a script to complete the `Legion Fealty 1` quest until the player has `Revenant's Spellscroll x 20`.

Looking at the [wiki](http://aqwwiki.wikidot.com/legion-revenant-s-quests) page for this quest, we require [Aeacus Empowered](http://aqwwiki.wikidot.com/aeacus-empowered) x 50, [Tethered Soul](http://aqwwiki.wikidot.com/tethered-soul) x 300, [Darkened Essence](http://aqwwiki.wikidot.com/darkened-essence) x 500, and [Dracolich Contract](http://aqwwiki.wikidot.com/dracolich-contract) x 1000.

The first thing to do is initialize the script as before and move these items, as well as the quest reward into the player's inventory:

```csharp
public void ScriptMain(ScriptInterface bot)
{
    bot.Options.SafeTimings = true;
    bot.Options.RestPackets = true;

    bot.Skills.LoadSkills("Skills/VoidHighlord.xml");
    bot.Skills.StartTimer();

    bot.Player.LoadBank();
    bot.Inventory.BankAllCoinItems();

    bot.Bank.ToInventory("Aeacus Empowered");
    bot.Bank.ToInventory("Tethered Soul");
    bot.Bank.ToInventory("Darkened Essence");
    bot.Bank.ToInventory("Dracolich Contract");
    bot.Bank.ToInventory("Revenant's Spellscroll");

    // ... next snippet continues from here
}
```

Now we want to repeat the quest until the player has `Revenant's Spellscroll x 20` in their inventory, so we write the while loop:

```csharp
while(!bot.Inventory.Contains("Revenant's Spellscroll", 20))
{
    // ... quest completion code will go here
}
```

Inside the loop, the requirements for the quest will be fullfilled by the bot. First we accept the quest, then we get all the required items, then we turn in the quest and pickup the reward as before:

```csharp
bot.Quests.EnsureAccept(6897);
			
bot.Player.JoinGlitched("judgement");
bot.Player.HuntForItem("Ultra Aeacus", "Aeacus Empowered", 50);

bot.Sleep(2000);

// A dark caster class is required to join this room.
// This one only costs 2000 legion tokens.
bot.Bank.ToInventory("Infinite Legion Dark Caster");
bot.Player.JoinGlitched("revenant");
bot.Player.HuntForItem("Forgotten Soul", "Tethered Soul", 300);

bot.Sleep(2000);

bot.Player.JoinGlitched("shadowrealm");
bot.Player.HuntForItem("Pure Shadowscythe|Shadow Guardian|Shadow Warrior", "Darkened Essence", 500);

bot.Sleep(2000);

bot.Player.JoinGlitched("necrodungeon");
bot.Sleep(1000);
// Here I enable Aggro Monsters mid script as it drastically increases the speed of getting the contracts, especially in a glitched room.
bot.Options.AggroMonsters = true;
bot.Player.HuntForItem("5 Headed Dracolich", "Dracolich Contract", 1000);
bot.Options.AggroMonsters = false;

bot.Quests.EnsureComplete(6897);
bot.Wait.ForDrop("Revenant's Spellscroll");
bot.Player.Pickup("Revenant's Spellscroll");
```

I have added 2 second sleeps between each map join to improve stability, although it usually is not necessary to do this. I have also used `JoinGlitched` instead of `Join` as this is a very popular quest and will almost always have glitched rooms on populated servers (usually Artix is the most populated).

My final Legion Fealty 1 script can be found in the release under `Scripts/LF1.cs`. I have also made LF2 and 3 scripts although they are untested. There are many more script examples in the `Scripts` directory that comes with the bot.