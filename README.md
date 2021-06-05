# discord.py-with-message-components
The Original [discord.py](https://github.com/Rapptz/discord.py) Libary made by [Rapptz](https://github.com/Rapptz) with implementation of the [Discord-Message-Components](https://discord.com/developers/docs/interactions/message-components) by [mccoderpy](https://github.com/mccoderpy/)

([discord](https://discord.com/channels/@me)-User `mccuber04#2960`)

## Installation:

Replace the [discord](file:///\C:/%APPDATA%/Local/Programs/Python/Python39/Lib/site-packages/discord) folder in the site-packages with the discord folder contained in this repository or yust clone it with git.


## Questions or ideas?

Send me a direct-message on [Discord](https://discord.com/channels/@me): `mccuber04#2960`

--------------------------------------------------------

# Examples:

```py
import discord
from discord.ext import commands
from discord import ActionRow, Button, ButtonColor

client = commands.Bot(command_prefix=commands.when_mentioned_or('.!'), intents=discord.Intents.all(), case_insensitive=True)


# an Command that sends you an Message and edit it if you click an Button


@client.command(name='buttons', description='sends you some nice Buttons')
async def buttons(ctx: commands.Context):
    components = [ActionRow(Button(label='Option Nr.1',
                                   custom_id='option1',
                                   emoji='1️1️⃣2️',
                                   style=ButtonColor.green
                                   ),
                            Button(label='Option Nr.2',
                                   custom_id='option2',
                                   emoji='2️⃣',
                                   style=ButtonColor.blurple)),
                  ActionRow(Button(label='A Other Row',
                                   custom_id='sec_row_1st option',
                                   emoji='😀'),
                            Button(url='https://www.youtube.com/watch?v=dQw4w9WgXcQ',
                                   label="This is an Link",
                                   emoji='🎬'))
                  ]
    an_embed = discord.Embed(title='Here are some Button\'s', description='Choose an option', color=discord.Color.random())
    msg = await ctx.send(embed=an_embed, components=components)

    def _check(i: discord.RawInteractionCreateEvent):
        return i.message == msg and i.member == ctx.author

    interaction: discord.RawInteractionCreateEvent = await client.wait_for('interaction_create', check=_check)
    button_id = interaction.button.custom_id

    # This sends the Discord-API that the interaction has been received and is being "processed" (if this is not used, Discord will indicate that the interaction failed).
    await interaction.defer()

    if button_id == "option1":
        await interaction.message.edit(embed=an_embed.add_field(name='Choose', value=f'Your Choose was `{button_id}`'), components=[components[0].edit_obj(0, disabled=True), components[1]])

    elif button_id == "option2":
        await interaction.message.edit(embed=an_embed.add_field(name='Choose', value=f'Your Choose was `{button_id}`'), components=[components[0].edit_obj(1, disabled=True), components[1]])

    elif button_id == 'sec_row_1st option':
        await interaction.message.edit(embed=an_embed.add_field(name='Choose', value=f'Your Choose was `{button_id}`'),
                                       components=[components[0], components[1].edit_obj(0, disabled=True)])

    # The Discord API doesn't send an event when you press a link button so we can't "receive" that.

client.run('Your Bot-Token here')
```
----------------------------------------------------------------------------------------------------
```py

# Another command where a small embed is sent where you can move the small white ⬜ with the buttons.

pointers = []


class Pointer:
    def __init__(self, guild: discord.Guild):
        self.guild = guild
        self._possition_x = 0
        self._possition_y = 0

    @property
    def possition_x(self):
        return self._possition_x

    def set_x(self, x: int):
        self._possition_x += x
        return self._possition_x

    @property
    def possition_y(self):
        return self._possition_y

    def set_y(self, y: int):
        self._possition_y += y
        return self._possition_y


def get_pointer(obj: typing.Union[discord.Guild, int]):
    if isinstance(obj, discord.Guild):
        for p in pointers:
            if p.guild.id == obj.id:
                return p
        return get_pointer(obj)

    elif isinstance(obj, int):
        for p in pointers:
            if p.guild.id == obj:
                return p
        guild = client.get_guild(obj)
        if guild:
            pointers.append(Pointer(guild))
            return get_pointer(guild)
        return None


def display(x: int, y: int):
    base = [
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    ]
    base[y].__setitem__(x, 1)
    base.reverse()
    return ''.join(f"\n{''.join([str(base[i][w]) for w in range(len(base[i]))]).replace('0', '⬛').replace('1', '⬜')}" for i in range(len(base)))


empty_button = discord.Button(style=discord.ButtonStyle.Secondary, label=" ", custom_id="empty", disabled=True)


@property
def arrow_button():
    return discord.Button(style=discord.ButtonStyle.Primary)


async def on_raw_interaction_create(interaction: discord.RawInteractionCreateEvent):
    await interaction.defer()
    pointer: Pointer = get_pointer(interaction.guild)
    if not (message := interaction.message):
        message: discord.Message = await interaction.channel.fetch_message(interaction.message_id)
    if interaction.button.custom_id == "links":
        await message.edit(embed=discord.Embed(title="Du Hast Links gewählt"), components=[discord.ActionRow(discord.Button(label='Links', custom_id='links', style=discord.ButtonStyle.Secondary, disabled=True), discord.Button(label='Rechts', custom_id='rechts', style=discord.ButtonStyle.Danger))])
    elif interaction.button.custom_id == "rechts":
        await message.edit(embed=discord.Embed(title="Du Hast Rechts gewählt"), components=[discord.ActionRow(discord.Button(label='Links', custom_id='links', style=discord.ButtonStyle.Danger), discord.Button(label='Rechts', custom_id='rechts', style=discord.ButtonStyle.Secondary, disabled=True))])
    elif interaction.button.custom_id == "up":
        pointer.set_y(1)
        await message.edit(embed=discord.Embed(title="Little Game",
                                               description=display(x=pointer.possition_x, y=pointer.possition_y)),
                           components=[discord.ActionRow(empty_button, arrow_button.set_label('↑').set_custom_id('up').disable_if(pointer.possition_y >= 9), empty_button),
                                       discord.ActionRow(arrow_button.set_label('←').set_custom_id('left').disable_if(pointer.possition_x <= 0), arrow_button.set_label('↓').set_custom_id('down'), arrow_button.set_label('→').set_custom_id('right').disable_if(pointer.possition_x >= 9))]
                           )
    elif interaction.button.custom_id == "down":
        pointer.set_y(-1)
        await message.edit(embed=discord.Embed(title="Little Game",
                                               description=display(x=pointer.possition_x, y=pointer.possition_y)),
                           components=[discord.ActionRow(empty_button, arrow_button.set_label('↑').set_custom_id('up'), empty_button),
                                       discord.ActionRow(arrow_button.set_label('←').set_custom_id('left').disable_if(pointer.possition_x <= 0), arrow_button.set_label('↓').set_custom_id('down').disable_if(pointer.possition_y <= 0), arrow_button.set_label('→').set_custom_id('right').disable_if(pointer.possition_x >= 9))]
                           )
    elif interaction.button.custom_id == "right":
        pointer.set_x(1)
        await message.edit(embed=discord.Embed(title="Little Game",
                                               description=display(x=pointer.possition_x, y=pointer.possition_y)),
                           components=[discord.ActionRow(empty_button, arrow_button.set_label('↑').set_custom_id('up'), empty_button),
                                       discord.ActionRow(arrow_button.set_label('←').set_custom_id('left'), arrow_button.set_label('↓').set_custom_id('down'), arrow_button.set_label('→').set_custom_id('right').disable_if(pointer.possition_x >= 9))]
                           )
    elif interaction.button.custom_id == "left":
        pointer.set_x(-1)
        await message.edit(embed=discord.Embed(title="Little Game",
                                               description=display(x=pointer.possition_x, y=pointer.possition_y)),
                           components=[discord.ActionRow(empty_button, arrow_button.set_label('↑').set_custom_id('up'), empty_button),
                                       discord.ActionRow(arrow_button.set_label('←').set_custom_id('left').disable_if(pointer.possition_x <= 0), arrow_button.set_label('↓').set_custom_id('down'), arrow_button.set_label('→').set_custom_id('right'))]
                           )

```