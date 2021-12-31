# Vanosilla - server-translations

`./server-translations` contains all in-game server-side translations.

Available languages:
- EN
- ES
- FR
- IT
- DE
- CZ
- PL
- TR

Each language directory contains `.yaml` files (if you don't know what .yaml is, [wiki](https://en.wikipedia.org/wiki/YAML) is your friend).

Each `.yaml` file stores data in key-value pair, for example:

```
HELLO_WORLD: Hello World!
```

`HELLO_WORLD` is the **key** and `Hello World!` is the **value** of this key.

___

### ❗ Note

You can use `'Hello World!'` quote or even double quotes for value: `"Hello World!"`.

You **can't** use white spaces for naming the key:

✔️ - `HELLO_WORLD`

❌ - `HELLO WORLD`

___

For better key formatting, I recommend using the naming syntax:

`WHAT-WHERE-OTHER_INFO`:
- `WHAT` - what does it belong to, examples: `SPECIALIST`, `PARTNER`, `FAMILY`, `EQUIPMENT`, `MINILAND`
- `WHERE` - where it appears, examples: `CHATMESSAGE`, `SHOUTMESSAGE`, `INFO`, `DIALOG`
- `OTHER_INFO` - short information on what it is for, examples: `LEVEL_TOO_HIGH`, `ENABLED`, `DISABLED`, `NOT_ALLOWED_WITH_RAID_MEMBER`

Let's take the example of `TIMESPACE_SHOUTMESSAGE_MEMBER_NOT_IN_CLASSIC_MAP`:
- `TIMESPACE` - this message belongs to Time-Spaces
- `SHOUTMESSAGE` - it will appear on the top of the screen
- `MEMBER_NOT_IN_CLASSIC_MAP` - information that member is not in classic map

Here are some types of the `WHERE` tag used in translations:
- `CHATMESSAGE` - appears in the chat
- `SHOUTMESSAGE` - appears on the top of the screen
- `INFO` - appears in small info box
- `MESSAGE` - appears in chat and on the top of the screen
- `DIALOG` or `QNA` - appears in ask dialogs

# Game Dialog Keys

The main one is `game-dialog-key.yaml`.

Game Dialog Keys store all in-game translations for info boxes, chat messages, messages on the top and much more.

## Creating a new key

Let's create a new key. The first step will be creating a new key in `GameDialogKey.cs` enum.

Go to the end of the file and create our new key - for demonstration purposes I will create a key that will say hello to the player: `WELCOME_CHATMESSAGE_HELLO_PLAYER`.

It should look like that:

![](https://i.imgur.com/HBShNkl.png)

Great! Now, save the file and let's build the solution.

After that, let's use the script that generates our new key for each language to `game-dialog-key.yaml`, so you don't have to paste it to each language manually. 

You can do it in two ways:

- PowerShell (if you have permissions to execute .ps1 files):
    - Go to the `./server` directory and open PowerShell terminal.
    - Type `.\scripts\translations-update.ps1` and click [ENTER] key.
    - Final output should looks like this: ![](https://i.imgur.com/EMP6ucY.png)
- Command Prompt / Windows Terminal:
    - Go to the `./server` directory, then enter `./dist/toolkit` directory.
    - Type `Toolkit.exe translations -i ../../../server-translations -o ../../../server-translations` and press [ENTER] key.
    - Final output should look like this: ![](https://i.imgur.com/xrP7paU.png)

___

### ❗ Note

If you get error while using the script `'Toolkit.exe' is not recognized as an internal or external command` that means you didn't build the Toolkit project. Go to the your IDE, choose Toolkit project and click `Build project` and try to use the script again:

![](https://i.imgur.com/rJujpfA.png)

___

Okay, if everything was done successfully, we should see our new key in all `game-dialog-key.yaml` files.
Now we will mainly focus on the English file as our script will update all other languages if they have not been translated.

Now, open `.en/game-dialog-key.yaml` file by using text edior (I recommend using Notepad++ or Visual Studio Code for that) and find our new key (my key will be at the end of file, but it can be everywhere so you should find it by using `CTRL + F` key).

![](https://i.imgur.com/Ta1deph.png)

As you can see the new key has been generated. The default value of the key will be always `'#<KEY>'`. Now, let's change our value to some proper message:

![](https://i.imgur.com/jFjFrPn.png)

Now save the file and let's use the script again - it will convert all non-English non-translated keys to English. 

For example if English key is transleted (so it doesn't start with `'#<KEY>'`) and German is not (so it does start with `'#<KEY>'`), it will replace not-translated key into English, so to visualize:

Translated English key `.en/game-dialog-key.yaml`:

`WELCOME_CHATMESSAGE_HELLO_PLAYER: "Hello player in our server!"`

Not-translated German key `.de/game-dialog-key.yaml`:

`WELCOME_CHATMESSAGE_HELLO_PLAYER: '#WELCOME_CHATMESSAGE_HELLO_PLAYER'`

After running a script in `.de/game-dialog-key.yaml` it will be:

`WELCOME_CHATMESSAGE_HELLO_PLAYER: "Hello player in our server!"`

... because it has not been translated.

___

## Using a new key in emulator

⚠️ To implement a new key, you need understand some basics of C# ⚠️

To use a new key, you have to use `GetLanguage` method from `IClientSession`, where the first parameter of the method is `GameDialogKey` enum.

So it should look like that:
```csharp
session.GetLanguage(GameDialogKey.WELCOME_CHATMESSAGE_HELLO_PLAYER)
```

The `GetLanguage` method returns the string of the translated GameDialogKey depending on player's language.

If you want to get a translated key in other language, you can use `IGameLanguageService` interface that contains all translations, and use `GetLanguage` method.

First, lets implement the language service by using [Depedency Injection](https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection) in the constructor of `MyClass`:

```csharp
private readonly IGameLanguageService _language;

public MyClass(IGameLanguageService language)
{
    _language = langauge;
}
```

After implementation, let's use it - this time I want to know how the key is translated to French, so I will use:

```csharp
_language.GetLanguage(GameDialogKey.WELCOME_CHATMESSAGE_HELLO_PLAYER, RegionLanguageType.FR)
```

In this example, I will use a key in `CharacterLoadEventHandler.cs` file - it's an event handler when player chooses his character in character selection.

First, I need to find `TryToSendSomeWarningMessages` method that will send some messages at the very end.

Then, as my key says, it will send chat message to the player, so I'm using `SendChatMessage` method with the color green.

```csharp
session.SendChatMessage(session.GetLanguage(GameDialogKey.WELCOME_CHATMESSAGE_HELLO_PLAYER), ChatMessageColorType.Green)
```

... and done! Congratulations, you created a new key!

___

## Creating a new key with arguments

Now, let's say that you want to create a key that holds some data - in our previous example, we're saying hello to the player after character selection... but what if we want to say hello with the player name?

To do this, you have to modify the key that contains `{0}` as argument in value - you can even create a key with many arguments, but with increasing number in `{}` bracket, so `{0}`, `{1}`, `{2}` and so on. Let's modify our previous key to handle player name.

The modified key will look like this:

`WELCOME_CHATMESSAGE_HELLO_PLAYER: "Hello {0} in our server!"`

___

### ❗ Note

If the value of the key stores arguments, you have to **always** use quotes or double quotes!

✔️ - `WELCOME_CHATMESSAGE_HELLO_PLAYER: "Hello {0} in our server!"`

✔️ - `WELCOME_CHATMESSAGE_HELLO_PLAYER: 'Hello {0} in our server!'`

❌ - `WELCOME_CHATMESSAGE_HELLO_PLAYER: Hello {0} in our server!`

The bad version will throw error while parsing.

___

Like previously, let's enter the `TryToSendSomeWarningMessages` method and modify our key.

First of all, we have to change the method from `GetLanguage` to `GetLanguageFormat` to allow accepting arguments of the key.

```csharp
session.GetLanguageFormat(GameDialogKey.WELCOME_CHATMESSAGE_HELLO_PLAYER)
```

Practically nothing has changed, but we still need to somehow send the player name to the method. Maybe you have noticed that `GetLanguageFormat` method accepts [params](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/params) of object (so basically everything from number to text). That means the method accepts any number of parameters.

On `GetLanguageFormat` method execution, it will create an array of objects with the parameters you created. Let's add a player name into the method:

```csharp
session.GetLanguageFormat(GameDialogKey.WELCOME_CHATMESSAGE_HELLO_PLAYER, session.PlayerEntity.Name)
```

So the first index of the array `formatParams` will store the name of the player:

```csharp
formatParams[0] = session.PlayerEntity.Name
```

The final result of the message will look like this (the player name is Jacob):

`Hello Jacob in our server!`

If you want to add more arguments into value, the indexes will increase - this time, I will add a level of the player:

`WELCOME_CHATMESSAGE_HELLO_PLAYER: "Hello {0} in our server! Your level is: {1}Lv."`

`{0}` will store a player's name

`{1}` will store a player's level

Let's change our method to accept player's level (let's say the player's level is 52):

```csharp
session.GetLanguageFormat(GameDialogKey.WELCOME_CHATMESSAGE_HELLO_PLAYER, session.PlayerEntity.Name, session.PlayerEntity.Level)
```

Our new `formatParams` array will look like this:
```csharp
formatParams[0] = session.PlayerEntity.Name (Jacob)
formatParams[1] = session.PlayerEntity.Level (52)
```

The final result of the message:

`Hello Jacob in our server! Your level is: 52Lv.`

___

# Storing translations in other files

If you want to create translations in other files, you have to create a file with the `.yaml` extension.

___

### ❗ Note

After file creation, you have to copy the created file to each language directory - Toolkit doesn't do it for you.

___

For demonstration purposes, I will create translations for arena. Let's create a new file `arena.yaml` in `./en` directory and add some keys into it:

- `ARENA_OF_TALENTS_NAME: Arena of Talents`
- `ARENA_OF_MASTERS_NAME: Arena of Masters`

As you can see, we didn't add new keys into `GameDialogKey` enum, because it doesn't belongs there. This time, instead of calling `GameDialogKey` enum inside `GetLanguage` or `GetLanguageFormat` method, we have to use a string of the key:

```csharp
session.GetLanguage("ARENA_OF_TALENTS_NAME")
```

Well, storing keys like that doesn't look good - for better storing keys, I recommend using static class with const strings of the key:

```csharp
public static class ArenaKeysConsts
{
    public const string ARENA_OF_TALENTS_NAME = "ARENA_OF_TALENTS_NAME";
    public const string ARENA_OF_MASTERS_NAME = "ARENA_OF_MASTERS_NAME";
}
```

... and use it in method:

```csharp
session.GetLanguage(ArenaKeysConsts.ARENA_OF_TALENTS_NAME)
```

That looks so much better.