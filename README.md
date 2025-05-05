import discord
from discord.ext import commands

# Bot token
TOKEN = ""

# Channel ID where the ticket creation message will be sent
CHANNEL_ID =   # Replace this with your actual channel ID

# Category ID where tickets will be created
CATEGORY_ID =   # Replace this with your actual category ID

intents = discord.Intents.default()
intents.guilds = True
intents.messages = True
intents.message_content = True  # To read message content
intents.members = True  # To mention members properly

bot = commands.Bot(command_prefix="!", intents=intents)

# Ticket button class
class TicketButton(discord.ui.View):
    def __init__(self):
        super().__init__(timeout=None)

    @discord.ui.button(label="🎟️ Create a Support Ticket", style=discord.ButtonStyle.gray, custom_id="AmiX-VX")
    async def create_ticket(self, interaction: discord.Interaction, button: discord.ui.Button):
        guild = interaction.guild
        user = interaction.user

        # Check if the user already has an open ticket
        existing_channel = discord.utils.get(guild.text_channels, name=f"ticket-{user.name}")
        if existing_channel:
            await interaction.response.send_message("📌 You already have an open ticket!", ephemeral=True)
            return

        # Get the category where the ticket should be created
        category = discord.utils.get(guild.categories, id=CATEGORY_ID)
        if category is None:
            await interaction.response.send_message("⚠️ Ticket category not found. Please check the CATEGORY_ID.", ephemeral=True)
            return

        # Create a new ticket channel inside the category
        overwrites = {
            guild.default_role: discord.PermissionOverwrite(view_channel=False),
            user: discord.PermissionOverwrite(view_channel=True, send_messages=True, read_message_history=True),
            guild.me: discord.PermissionOverwrite(view_channel=True, send_messages=True, read_message_history=True)
        }

        ticket_channel = await guild.create_text_channel(f"ticket-{user.name}", overwrites=overwrites, category=category)

        # Embed message for ticket creation
        embed = discord.Embed(title="Your Ticket is Opened", description=f"{user.mention}, please describe your issue below.", color=0x2b2d31)
        embed.set_footer(text="Ticket | AmiX-VX", icon_url="https://cdn.discordapp.com/icons/123456789012345678/your_icon.png")  # Replace with your actual icon

        await ticket_channel.send(embed=embed, view=CloseTicketButton())

        # Send a message in the main channel tagging the newly created ticket channel
        await interaction.response.send_message(f"✅ Your ticket has been created! {ticket_channel.mention}", ephemeral=True)

# Close ticket button class (with the same design as opening a ticket)
class CloseTicketButton(discord.ui.View):
    def __init__(self):
        super().__init__(timeout=None)

    @discord.ui.button(label="🔒 Close Ticket", style=discord.ButtonStyle.gray, custom_id="close_ticket")
    async def close_ticket(self, interaction: discord.Interaction, button: discord.ui.Button):
        channel = interaction.channel

        # Embed message for ticket closing confirmation
        embed = discord.Embed(title="Close the Ticket", description="Click the button below to close your support ticket.", color=0x2b2d31)
        embed.set_footer(text="Tickety | Tickety.top", icon_url="https://cdn.discordapp.com/icons/123456789012345678/your_icon.png")  # Replace with your actual icon

        await channel.send(embed=embed, view=ConfirmCloseTicketButton())

# Confirmation button before closing the ticket
class ConfirmCloseTicketButton(discord.ui.View):
    def __init__(self):
        super().__init__(timeout=None)

    @discord.ui.button(label="✅ Confirm Close", style=discord.ButtonStyle.danger, custom_id="confirm_close_ticket")
    async def confirm_close_ticket(self, interaction: discord.Interaction, button: discord.ui.Button):
        await interaction.response.send_message("✅ The ticket will be closed shortly.", ephemeral=True)
        await interaction.channel.delete()

@bot.event
async def on_ready():
    print(f"✅ {bot.user} is now online!")

    # Get the specified channel and send the ticket creation message
    channel = bot.get_channel(CHANNEL_ID)
    if channel:
        embed = discord.Embed(title="AmiX Development", description="Please click on the button below to create a support ticket.", color=0x2b2d31)
        embed.set_footer(text="Ticket | AmiX Development", icon_url="https://cdn.discordapp.com/icons/123456789012345678/your_icon.png")  # Replace with your actual icon
        await channel.send(embed=embed, view=TicketButton())

bot.run(TOKEN)
