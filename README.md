# chat2
An advanced chat solution for JC3MP

![](https://media.giphy.com/media/ftdALo80vqCZT45oT8/giphy.gif)
![](https://i.imgur.com/CJc7Znd.png)
![](https://media.giphy.com/media/7NDDKWoQmYcoRvKziX/giphy.gif)
![](https://i.imgur.com/X85vBM7.png)

*Much of this readme is from the default chat readme*

## Transitioning from default chat to chat2
It's pretty simple, except there are a few changes and a little bit of configuration you'll need to do. These are some basic steps to help you get started.
1. Delete the old chat package.
2. Install the chat2 package.
3. Inside `config.js`, modify it to include the channels that you want, in `default_channels`. If a channel with the name `Local` is specified, then it will be used as a local chat channel where you can only talk to players who are nearby. This local chat distance is specified by `local_distance` in `config.js`.
4. Admins won't have their stars anymore (though it is possible to readd them, but that will not be covered here), and if you implemented custom tags, you'll need to do a little bit of work to get tags to appear again. See the [Tags](#tags) section on more information about how you can implement nametags.
5. Continue on to the next section to finish up implementing chat2.

## Using the Package as Dependency
When you want to use the chat as a dependency for your package, make sure to add it to your `jcmp_dependencies` array. You'll probably want to replace all instance of `chat` with `chat2` in dependencies of your packages. If you don't want to do that, you can change the name of this package to `chat` in the `package.json` in both this directory and within the `client_package` directory. You can then use the following code snippet to get the chat object:

```javascript
const chat = jcmp.events.Call('get_chat')[0];
```

The returned object provides these functions:

### `chat.send(target: Player, message: string, color: RGB = new RGB(255, 255, 255), args: object)`
* target: target Player
* message: message string
* color (optional): rgb color for the message
* args (optional): additional arguments for the message
    * **Available arguments include:**
    * timeout (integer): number of seconds until the message should be removed
    * channel (string): what channel the message should be sent on. If no channel is given, it will be sent to the player's currently open channel
    * style (string): a valid `font-style` for the message, such as `italic`
    * use_name (boolean): set this to true if the message contains a player's name 

Example:
```javascript
jcmp.events.Add('PlayerDeath', player =>
  chat.send(player, "You died.");
});
```

Example 2:
```javascript
jcmp.events.Add('PlayerDeath', player =>
  chat.send(player, "You died.", new RGB(255,0,0), {channel: 'CoolChannel', timeout: 10});
});
```

### `chat.broadcast(message: string, color: RGB = new RGB(255, 255, 255), args: object)`
* message: message string
* color (optional): rgb color for the message
* args (optional): additional arguments for the message
    * **Available arguments include:**
    * timeout (integer): number of seconds until the message should be removed
    * channel (string): what channel the message should be sent on. If no channel is given, it will be sent to the player's currently open channel
    * style (string): a valid `font-style` for the message, such as `italic`
    * use_name (boolean): set this to true if the message contains a player's name 

Example:
```
jcmp.events.Add('PlayerDeath', () =>
  chat.broadcast("Someone died!");
});
```

## Formatting Messages
Your messages can include any html, except if `use_name` is true in `args`.
If you want your message to include multiple colors, you can do that by using color tags in this format: `[#RRGGBB]`

Example:
```
jcmp.events.Add('PlayerDeath', player =>
  chat.send(player, "[#ff0000]You [#ffffff]died.");
  // You will be red, the rest will be white
});
```

## Tags
This chat supports neat looking tags. To enable this feature, you must give a player a tag. This can be done like so:
```javascript
player.tag = 
{
    name: 'Admin',
    color: '#FF0000'
}
```

If you want to replace the default admin star with an Admin tag, here are some steps on doing that.

1. Navigate to `freeroam/events/player.js`.
2. Inside the `PlayerCreated` event, near line 34, add this line:
```javascript
player.tag = 
{
    name: 'Admin',
    color: '#FF0000'
}
```

3. The entire `PlayerCreated` event should now look like this:
```javascript
jcmp.events.Add("PlayerCreated", player => {
    player.escapedNametagName = player.name.replace(/</g, '&lt;').replace(/>/g, '&gt;').substring(0, 40);
    console.log(`${player.escapedNametagName} has joined.`);
    freeroam.chat.broadcast(`** ${player.escapedNametagName} has joined.`, freeroam.config.colours.connection);

    const colour = freeroam.colours.randomColor();
    player.freeroam = {
        colour: colour,
        colour_rgb: freeroam.utils.hexToRGB(colour),
        kills: 0,
        deaths: 0,
        custom_time_set: false,
        vehicle_nitro_toggled: false,
        vehicle_turbojump_toggled: false
    };

    player.group = null;
    player.groupInvite = null;

    const data = {
        id: player.networkId,
        name: player.escapedNametagName,
        colour: player.freeroam.colour,
        isAdmin: freeroam.utils.isAdmin(player),
    };

    player.tag = 
    {
        name: 'Admin',
        color: '#FF0000'
    }

    jcmp.events.CallRemote("freeroam_player_created", null, JSON.stringify(data));
});
```

If you did everything correctly, any admins registered in the freeroam package should have Admin tags in chat.

## Mentioning
You can mention players, similar to Discord.

**Mentioning a specific player**

1. Start typing your message: `hello`
2. Start typing the @ part of the message: `hello @ri`
3. Once you have part of the player's name, press TAB to autocomplete: `hello @"rico"`
4. If it isn't the right player, you can hit TAB again to cycle through players.
5. **You __must__ use TAB and autocomplete to mention a player, otherwise it will fail.**

**Mentioning Everyone**

In order to mention everyone, the player must have the `Admin` tag. If the message is sent by the server (through `send` or `broadcast`), it can mention everyone as well.

Player:
1. Type your message, and make sure to add `@everyone` somewhere: `hello @everyone!`
2. Send your message

Server:
1. Make sure the message includes `@everyone`: `chat.broadcast("@everyone Hello!");`
2. You can also mention specific players by name with `@"PlayerName"`

## Events
All events from the default chat are included, such as `chat_command` and `chat_message`. However, these have an additional argument, which is the channel that the message was sent on.

`jcmp.events.Add('LocalPlayerChat', (msg) => {})`
Client event that fires when the client sends a message. Return false to cancel sending.


## Built In Commands & Keys
`/clear`
Clears the current channel of all chat messages.

`F5`
Hides/shows the chat window.

`Up or Down Key while input is open`
Go through previously sent messages to easily the same message again.

## Logging
Chat logs are recorded in `chat2/logs` daily. They include player names and steam ids.