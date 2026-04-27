"""
Fían Bán Discord Bot — Full Administration Build
Complete server management, moderation, and community tools.
"""

import discord
from discord.ext import commands, tasks
from discord import app_commands
import json
import os
from datetime import datetime, date, timedelta
from typing import Optional
import asyncio

# ─── CONFIGURATION ────────────────────────────────────────────────────────────
TOKEN = os.environ.get("DISCORD_TOKEN", "YOUR_BOT_TOKEN_HERE")
GUILD_ID = int(os.environ.get("GUILD_ID", "YOUR_GUILD_ID_HERE"))

RULES_CHANNEL = "code-of-conduct"
ROLE_SELECT_CHANNEL = "role-selection"
WELCOME_CHANNEL = "welcome"
BASE_ROLE = "Fían"
VERIFIED_ROLE = "Sworn"

# ─── DATA FILES ───────────────────────────────────────────────────────────────
PROFILES_FILE = "profiles.json"
WARNINGS_FILE = "warnings.json"
REMINDERS_FILE = "reminders.json"

def load_json(filepath):
    if os.path.exists(filepath):
        with open(filepath, "r") as f:
            return json.load(f)
    return {}

def save_json(filepath, data):
    with open(filepath, "w") as f:
        json.dump(data, f, indent=2)

# ─── IRISH CALENDAR ───────────────────────────────────────────────────────────
IRISH_CALENDAR = [
    {
        "name": "Imbolc",
        "date": (2, 1),
        "description": (
            "**Imbolc — February 1st**\n"
            "Sacred to Brigid. The first stirring of the light half. "
            "Purification, renewal, the return from the dark half's ranging. "
            "The fénnidi were traditionally cleansed and restored to society at this time.\n\n"
            "*Observance details to be added by the community.*"
        )
    },
    {
        "name": "Bealtaine",
        "date": (5, 1),
        "description": (
            "**Bealtaine — May 1st**\n"
            "Opening of the light half. The campaign season begins. "
            "The fénnidi leave the settled lands and range through the summer landscape. "
            "The sovereignty of the land is most active.\n\n"
            "*Observance details to be added by the community.*"
        )
    },
    {
        "name": "Lúnasa",
        "date": (8, 1),
        "description": (
            "**Lúnasa — August 1st**\n"
            "Festival of Lugh. The testing season. The Tailteann Games — "
            "martial contests, athletic competition, legal assembly. "
            "Demonstrate what the training has produced.\n\n"
            "*Observance details to be added by the community.*"
        )
    },
    {
        "name": "Samhain",
        "date": (11, 1),
        "description": (
            "**Samhain — November 1st**\n"
            "The most significant threshold of the year. The boundary between "
            "the living world and the otherworld dissolves. The dark half begins. "
            "The Morrigan moves. The fénnidi return to the settled lands.\n\n"
            "*Observance details to be added by the community.*"
        )
    }
]

# ─── ONBOARDING CONTENT ───────────────────────────────────────────────────────
RULES_TEXT = """
**Fían Bán — Conduct and Oath**

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Before you enter this community you must understand what it is and what it requires.

**This community is grounded in:**
The pre-Christian Irish warrior tradition as found in primary sources. The three pillars of Corp (body), Fís (knowledge), and Fír (truth). The vocational understanding of the warrior as one who protects rather than dominates.

**Conduct requirements:**

**I. Fír — Truth**
Speak honestly. Do not misrepresent your background, your practice, or your intentions. Your word here carries the same weight as your word anywhere else.

**II. Respect for the tradition**
Engage with the source material seriously. This is not an aesthetic community. If you are here for the look rather than the substance, this is not the right place.

**III. No political extremism**
White supremacy, ethnic nationalism, and any ideology that uses the Irish tradition as a vehicle for racial or ethnic exclusivity are explicitly prohibited. The tradition belongs to those who do the work. Full stop.

**IV. The warrior's obligation to protection**
The warrior path is defined by protection, not domination. Aggression without cause, harassment of community members, and the use of warrior identity to intimidate others are prohibited.

**V. The four geasa apply here**
Do not turn away someone who genuinely needs help. Do not act with cruelty toward the vulnerable. Do not abandon those who are standing with you. Do not walk past suffering you have the capacity to address.

**VI. Serious practice**
This community is for people actively working the path — training, studying the sources, developing their practice.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

*By clicking ✅ below you acknowledge these terms and swear to conduct yourself accordingly. This is your oath of entry. Take it seriously or do not take it.*
"""

# ─── BOT SETUP ────────────────────────────────────────────────────────────────
intents = discord.Intents.all()
bot = commands.Bot(command_prefix="!", intents=intents)
tree = bot.tree

# ─── HELPER: ADMIN CHECK ──────────────────────────────────────────────────────
def is_admin():
    async def predicate(interaction: discord.Interaction):
        return (
            interaction.user.guild_permissions.administrator or
            interaction.user.guild_permissions.manage_guild
        )
    return app_commands.check(predicate)

def is_mod():
    async def predicate(interaction: discord.Interaction):
        return (
            interaction.user.guild_permissions.administrator or
            interaction.user.guild_permissions.manage_guild or
            interaction.user.guild_permissions.kick_members or
            interaction.user.guild_permissions.ban_members
        )
    return app_commands.check(predicate)

# ─── EVENTS ───────────────────────────────────────────────────────────────────
@bot.event
async def on_ready():
    print(f"Fían Bán Bot online as {bot.user}")
    try:
        synced = await tree.sync()
        print(f"Synced {len(synced)} commands")
    except Exception as e:
        print(f"Sync error: {e}")
    check_calendar.start()
    check_reminders.start()

@bot.event
async def on_member_join(member: discord.Member):
    guild = member.guild
    fian_role = discord.utils.get(guild.roles, name=BASE_ROLE)
    if not fian_role:
        fian_role = await guild.create_role(
            name=BASE_ROLE,
            color=discord.Color.from_rgb(220, 220, 200)
        )
    await member.add_roles(fian_role)

    welcome_ch = discord.utils.get(guild.text_channels, name=WELCOME_CHANNEL)
    rules_ch = discord.utils.get(guild.text_channels, name=RULES_CHANNEL)

    if welcome_ch:
        embed = discord.Embed(
            title="Welcome to Fían Bán",
            description=(
                f"Fáilte, {member.mention}.\n\n"
                f"You have been given the **{BASE_ROLE}** role.\n\n"
                f"Read and swear the oath in "
                f"{rules_ch.mention if rules_ch else '#rules-and-oath'} "
                f"to gain access to the community.\n\n"
                f"*The Fianna did not admit anyone who had not first sworn. Neither do we.*"
            ),
            color=discord.Color.from_rgb(220, 220, 200)
        )
        embed.set_footer(text="Fían Bán — Corp, Fís, Fír")
        await welcome_ch.send(embed=embed)

@bot.event
async def on_raw_reaction_add(payload: discord.RawReactionActionEvent):
    if payload.user_id == bot.user.id:
        return
    guild = bot.get_guild(payload.guild_id)
    if not guild:
        return
    member = guild.get_member(payload.user_id)
    if not member or member.bot:
        return
    channel = guild.get_channel(payload.channel_id)
    if not channel:
        return
    emoji_str = str(payload.emoji)

    # Rules oath
    if channel.name == RULES_CHANNEL and emoji_str == "✅":
        verified_role = discord.utils.get(guild.roles, name=VERIFIED_ROLE)
        if not verified_role:
            verified_role = await guild.create_role(
                name=VERIFIED_ROLE,
                color=discord.Color.from_rgb(180, 160, 120)
            )
        if verified_role not in member.roles:
            await member.add_roles(verified_role)
            try:
                await member.send(
                    "Your oath has been received. You now have access to Fían Bán.\n"
                    "Head to #role-selection to complete your profile.\n\n"
                    "*Fír — hold to it.*"
                )
            except discord.Forbidden:
                pass
        return

    # Role selection reactions
    EMOJI_ROLE_MAP = {
        "🍀": "Irish Polytheism",
        "🏴󠁧󠁢󠁳󠁣󠁴󠁿": "Scottish Polytheism",
        "🏴󠁧󠁢󠁷󠁬󠁳󠁿": "Welsh Polytheism",
        "⚜️": "Brythonic Polytheism",
        "🔱": "Other Tradition",
        "🪖": "Míliath",
        "🎖️": "Seansaighdiúir",
        "🛡️": "Caomhnóir",
        "🏹": "Sealgaire",
        "📖": "Filidh",
        "🌀": "Draoi",
    }
    if channel.name == ROLE_SELECT_CHANNEL and emoji_str in EMOJI_ROLE_MAP:
        role_name = EMOJI_ROLE_MAP[emoji_str]
        role = discord.utils.get(guild.roles, name=role_name)
        if not role:
            role = await guild.create_role(name=role_name)
        if role not in member.roles:
            await member.add_roles(role)

@bot.event
async def on_raw_reaction_remove(payload: discord.RawReactionActionEvent):
    if payload.user_id == bot.user.id:
        return
    guild = bot.get_guild(payload.guild_id)
    if not guild:
        return
    member = guild.get_member(payload.user_id)
    if not member:
        return
    channel = guild.get_channel(payload.channel_id)
    if not channel or channel.name != ROLE_SELECT_CHANNEL:
        return
    EMOJI_ROLE_MAP = {
        "🍀": "Irish Polytheism",
        "🏴󠁧󠁢󠁳󠁣󠁴󠁿": "Scottish Polytheism",
        "🏴󠁧󠁢󠁷󠁬󠁳󠁿": "Welsh Polytheism",
        "⚜️": "Brythonic Polytheism",
        "🔱": "Other Tradition",
        "🪖": "Míliath",
        "🎖️": "Seansaighdiúir",
        "🛡️": "Caomhnóir",
        "🏹": "Sealgaire",
        "📖": "Filidh",
        "🌀": "Draoi",
    }
    emoji_str = str(payload.emoji)
    if emoji_str in EMOJI_ROLE_MAP:
        role_name = EMOJI_ROLE_MAP[emoji_str]
        role = discord.utils.get(guild.roles, name=role_name)
        if role and role in member.roles:
            await member.remove_roles(role)

# ═══════════════════════════════════════════════════════════════════════════════
# ADMIN COMMANDS — ROLES
# ═══════════════════════════════════════════════════════════════════════════════

@tree.command(name="role_create", description="[Admin] Create a new role")
@app_commands.describe(
    name="Role name",
    color="Hex color e.g. ff0000",
    hoist="Show separately in member list",
    mentionable="Allow @mentioning this role"
)
@is_admin()
async def role_create(
    interaction: discord.Interaction,
    name: str,
    color: Optional[str] = None,
    hoist: Optional[bool] = False,
    mentionable: Optional[bool] = False
):
    role_color = discord.Color.default()
    if color:
        try:
            role_color = discord.Color(int(color.strip("#"), 16))
        except ValueError:
            await interaction.response.send_message("Invalid color format. Use hex e.g. ff0000", ephemeral=True)
            return
    role = await interaction.guild.create_role(
        name=name, color=role_color, hoist=hoist, mentionable=mentionable
    )
    await interaction.response.send_message(f"Role **{role.name}** created.", ephemeral=True)

@tree.command(name="role_edit", description="[Admin] Edit an existing role")
@app_commands.describe(
    role="Role to edit",
    new_name="New name (optional)",
    color="New hex color e.g. ff0000 (optional)",
    hoist="Show separately in member list (optional)",
    mentionable="Allow @mentioning (optional)"
)
@is_admin()
async def role_edit(
    interaction: discord.Interaction,
    role: discord.Role,
    new_name: Optional[str] = None,
    color: Optional[str] = None,
    hoist: Optional[bool] = None,
    mentionable: Optional[bool] = None
):
    kwargs = {}
    if new_name:
        kwargs["name"] = new_name
    if color:
        try:
            kwargs["color"] = discord.Color(int(color.strip("#"), 16))
        except ValueError:
            await interaction.response.send_message("Invalid color format.", ephemeral=True)
            return
    if hoist is not None:
        kwargs["hoist"] = hoist
    if mentionable is not None:
        kwargs["mentionable"] = mentionable
    await role.edit(**kwargs)
    await interaction.response.send_message(f"Role **{role.name}** updated.", ephemeral=True)

@tree.command(name="role_delete", description="[Admin] Delete a role")
@app_commands.describe(role="Role to delete")
@is_admin()
async def role_delete(interaction: discord.Interaction, role: discord.Role):
    name = role.name
    await role.delete()
    await interaction.response.send_message(f"Role **{name}** deleted.", ephemeral=True)

@tree.command(name="role_assign", description="[Admin] Assign a role to a user")
@app_commands.describe(member="Member to assign role to", role="Role to assign")
@is_admin()
async def role_assign(interaction: discord.Interaction, member: discord.Member, role: discord.Role):
    await member.add_roles(role)
    await interaction.response.send_message(
        f"**{role.name}** assigned to {member.mention}.", ephemeral=True
    )

@tree.command(name="role_remove", description="[Admin] Remove a role from a user")
@app_commands.describe(member="Member to remove role from", role="Role to remove")
@is_admin()
async def role_remove(interaction: discord.Interaction, member: discord.Member, role: discord.Role):
    await member.remove_roles(role)
    await interaction.response.send_message(
        f"**{role.name}** removed from {member.mention}.", ephemeral=True
    )

@tree.command(name="role_list", description="[Admin] List all roles in the server")
@is_admin()
async def role_list(interaction: discord.Interaction):
    roles = [r for r in interaction.guild.roles if r.name != "@everyone"]
    roles_sorted = sorted(roles, key=lambda r: r.position, reverse=True)
    text = "\n".join([f"`{r.name}` — {len(r.members)} members" for r in roles_sorted])
    embed = discord.Embed(
        title=f"Roles — {interaction.guild.name}",
        description=text or "No roles found.",
        color=discord.Color.from_rgb(180, 160, 120)
    )
    await interaction.response.send_message(embed=embed, ephemeral=True)

@tree.command(name="role_members", description="[Admin] List all members with a specific role")
@app_commands.describe(role="Role to list members of")
@is_admin()
async def role_members(interaction: discord.Interaction, role: discord.Role):
    members = role.members
    if not members:
        await interaction.response.send_message(f"No members have **{role.name}**.", ephemeral=True)
        return
    text = "\n".join([f"{m.mention} ({m.display_name})" for m in members])
    embed = discord.Embed(
        title=f"Members with {role.name} ({len(members)})",
        description=text,
        color=discord.Color.from_rgb(180, 160, 120)
    )
    await interaction.response.send_message(embed=embed, ephemeral=True)

@tree.command(name="role_grant_sworn", description="[Admin] Manually grant Sworn role to a user")
@app_commands.describe(member="Member to grant Sworn role to")
@is_admin()
async def role_grant_sworn(interaction: discord.Interaction, member: discord.Member):
    verified_role = discord.utils.get(interaction.guild.roles, name=VERIFIED_ROLE)
    if not verified_role:
        verified_role = await interaction.guild.create_role(name=VERIFIED_ROLE)
    await member.add_roles(verified_role)
    await interaction.response.send_message(
        f"**{VERIFIED_ROLE}** granted to {member.mention}.", ephemeral=True
    )

@tree.command(name="role_revoke_sworn", description="[Admin] Revoke Sworn role from a user")
@app_commands.describe(member="Member to revoke Sworn role from")
@is_admin()
async def role_revoke_sworn(interaction: discord.Interaction, member: discord.Member):
    verified_role = discord.utils.get(interaction.guild.roles, name=VERIFIED_ROLE)
    if verified_role and verified_role in member.roles:
        await member.remove_roles(verified_role)
        await interaction.response.send_message(
            f"**{VERIFIED_ROLE}** revoked from {member.mention}.", ephemeral=True
        )
    else:
        await interaction.response.send_message(
            f"{member.mention} does not have the **{VERIFIED_ROLE}** role.", ephemeral=True
        )

# ═══════════════════════════════════════════════════════════════════════════════
# ADMIN COMMANDS — CHANNELS
# ═══════════════════════════════════════════════════════════════════════════════

@tree.command(name="channel_create", description="[Admin] Create a new channel")
@app_commands.describe(
    name="Channel name",
    channel_type="text or voice",
    category="Category to place it in (optional)",
    topic="Channel topic (text channels only)"
)
@is_admin()
async def channel_create(
    interaction: discord.Interaction,
    name: str,
    channel_type: Optional[str] = "text",
    category: Optional[str] = None,
    topic: Optional[str] = None
):
    cat_obj = None
    if category:
        cat_obj = discord.utils.get(interaction.guild.categories, name=category)
        if not cat_obj:
            cat_obj = await interaction.guild.create_category(category)
    if channel_type == "voice":
        ch = await interaction.guild.create_voice_channel(name, category=cat_obj)
    else:
        ch = await interaction.guild.create_text_channel(
            name, category=cat_obj, topic=topic
        )
    await interaction.response.send_message(
        f"Channel {ch.mention} created.", ephemeral=True
    )

@tree.command(name="channel_delete", description="[Admin] Delete a channel")
@app_commands.describe(channel="Channel to delete")
@is_admin()
async def channel_delete(
    interaction: discord.Interaction,
    channel: discord.TextChannel
):
    name = channel.name
    await channel.delete()
    await interaction.response.send_message(f"Channel **#{name}** deleted.", ephemeral=True)

@tree.command(name="channel_rename", description="[Admin] Rename a channel")
@app_commands.describe(channel="Channel to rename", new_name="New channel name")
@is_admin()
async def channel_rename(
    interaction: discord.Interaction,
    channel: discord.TextChannel,
    new_name: str
):
    old_name = channel.name
    await channel.edit(name=new_name)
    await interaction.response.send_message(
        f"**#{old_name}** renamed to **#{new_name}**.", ephemeral=True
    )

@tree.command(name="channel_topic", description="[Admin] Set a channel topic")
@app_commands.describe(channel="Channel to update", topic="New topic text")
@is_admin()
async def channel_topic(
    interaction: discord.Interaction,
    channel: discord.TextChannel,
    topic: str
):
    await channel.edit(topic=topic)
    await interaction.response.send_message(
        f"Topic set for {channel.mention}.", ephemeral=True
    )

@tree.command(name="channel_clone", description="[Admin] Clone a channel")
@app_commands.describe(channel="Channel to clone", new_name="Name for the clone (optional)")
@is_admin()
async def channel_clone(
    interaction: discord.Interaction,
    channel: discord.TextChannel,
    new_name: Optional[str] = None
):
    cloned = await channel.clone(name=new_name or f"{channel.name}-copy")
    await interaction.response.send_message(
        f"Channel cloned as {cloned.mention}.", ephemeral=True
    )

@tree.command(name="channel_lock", description="[Admin] Lock a channel — no one can send messages")
@app_commands.describe(channel="Channel to lock")
@is_admin()
async def channel_lock(
    interaction: discord.Interaction,
    channel: discord.TextChannel
):
    overwrite = channel.overwrites_for(interaction.guild.default_role)
    overwrite.send_messages = False
    await channel.set_permissions(interaction.guild.default_role, overwrite=overwrite)
    await interaction.response.send_message(
        f"{channel.mention} locked.", ephemeral=True
    )
    await channel.send("🔒 This channel has been locked by an administrator.")

@tree.command(name="channel_unlock", description="[Admin] Unlock a channel")
@app_commands.describe(channel="Channel to unlock")
@is_admin()
async def channel_unlock(
    interaction: discord.Interaction,
    channel: discord.TextChannel
):
    overwrite = channel.overwrites_for(interaction.guild.default_role)
    overwrite.send_messages = None
    await channel.set_permissions(interaction.guild.default_role, overwrite=overwrite)
    await interaction.response.send_message(
        f"{channel.mention} unlocked.", ephemeral=True
    )
    await channel.send("🔓 This channel has been unlocked.")

@tree.command(name="channel_readonly", description="[Admin] Make a channel read-only for everyone")
@app_commands.describe(channel="Channel to make read-only")
@is_admin()
async def channel_readonly(
    interaction: discord.Interaction,
    channel: discord.TextChannel
):
    overwrite = channel.overwrites_for(interaction.guild.default_role)
    overwrite.send_messages = False
    overwrite.add_reactions = False
    await channel.set_permissions(interaction.guild.default_role, overwrite=overwrite)
    await interaction.response.send_message(
        f"{channel.mention} set to read-only.", ephemeral=True
    )

@tree.command(name="channel_slowmode", description="[Admin] Set slowmode on a channel")
@app_commands.describe(
    channel="Channel to set slowmode on",
    seconds="Slowmode delay in seconds (0 to disable)"
)
@is_admin()
async def channel_slowmode(
    interaction: discord.Interaction,
    channel: discord.TextChannel,
    seconds: int
):
    await channel.edit(slowmode_delay=seconds)
    if seconds == 0:
        await interaction.response.send_message(
            f"Slowmode disabled on {channel.mention}.", ephemeral=True
        )
    else:
        await interaction.response.send_message(
            f"Slowmode set to **{seconds}s** on {channel.mention}.", ephemeral=True
        )

@tree.command(name="channel_move", description="[Admin] Move a channel to a different category")
@app_commands.describe(channel="Channel to move", category="Target category name")
@is_admin()
async def channel_move(
    interaction: discord.Interaction,
    channel: discord.TextChannel,
    category: str
):
    cat_obj = discord.utils.get(interaction.guild.categories, name=category)
    if not cat_obj:
        await interaction.response.send_message(
            f"Category **{category}** not found.", ephemeral=True
        )
        return
    await channel.edit(category=cat_obj)
    await interaction.response.send_message(
        f"{channel.mention} moved to **{category}**.", ephemeral=True
    )

@tree.command(name="channel_list", description="[Admin] List all channels by category")
@is_admin()
async def channel_list(interaction: discord.Interaction):
    guild = interaction.guild
    text = ""
    for cat in guild.categories:
        text += f"\n**{cat.name}**\n"
        for ch in cat.channels:
            icon = "🔊" if isinstance(ch, discord.VoiceChannel) else "💬"
            text += f"  {icon} #{ch.name}\n"
    uncategorized = [ch for ch in guild.channels if ch.category is None]
    if uncategorized:
        text += "\n**No Category**\n"
        for ch in uncategorized:
            text += f"  💬 #{ch.name}\n"
    embed = discord.Embed(
        title=f"Channels — {guild.name}",
        description=text or "No channels.",
        color=discord.Color.from_rgb(180, 160, 120)
    )
    await interaction.response.send_message(embed=embed, ephemeral=True)

# ═══════════════════════════════════════════════════════════════════════════════
# ADMIN COMMANDS — CATEGORIES
# ═══════════════════════════════════════════════════════════════════════════════

@tree.command(name="category_create", description="[Admin] Create a new category")
@app_commands.describe(name="Category name")
@is_admin()
async def category_create(interaction: discord.Interaction, name: str):
    cat = await interaction.guild.create_category(name)
    await interaction.response.send_message(
        f"Category **{cat.name}** created.", ephemeral=True
    )

@tree.command(name="category_delete", description="[Admin] Delete a category")
@app_commands.describe(name="Category name to delete")
@is_admin()
async def category_delete(interaction: discord.Interaction, name: str):
    cat = discord.utils.get(interaction.guild.categories, name=name)
    if not cat:
        await interaction.response.send_message(f"Category **{name}** not found.", ephemeral=True)
        return
    await cat.delete()
    await interaction.response.send_message(f"Category **{name}** deleted.", ephemeral=True)

@tree.command(name="category_rename", description="[Admin] Rename a category")
@app_commands.describe(current_name="Current category name", new_name="New name")
@is_admin()
async def category_rename(
    interaction: discord.Interaction,
    current_name: str,
    new_name: str
):
    cat = discord.utils.get(interaction.guild.categories, name=current_name)
    if not cat:
        await interaction.response.send_message(
            f"Category **{current_name}** not found.", ephemeral=True
        )
        return
    await cat.edit(name=new_name)
    await interaction.response.send_message(
        f"Category renamed to **{new_name}**.", ephemeral=True
    )

# ═══════════════════════════════════════════════════════════════════════════════
# ADMIN COMMANDS — MODERATION
# ═══════════════════════════════════════════════════════════════════════════════

@tree.command(name="mod_warn", description="[Mod] Warn a user")
@app_commands.describe(member="Member to warn", reason="Reason for the warning")
@is_mod()
async def mod_warn(
    interaction: discord.Interaction,
    member: discord.Member,
    reason: str
):
    warnings = load_json(WARNINGS_FILE)
    uid = str(member.id)
    if uid not in warnings:
        warnings[uid] = []
    warnings[uid].append({
        "reason": reason,
        "issued_by": str(interaction.user.id),
        "timestamp": datetime.utcnow().isoformat()
    })
    save_json(WARNINGS_FILE, warnings)
    count = len(warnings[uid])
    await interaction.response.send_message(
        f"{member.mention} warned. Reason: **{reason}**\nTotal warnings: **{count}**",
        ephemeral=True
    )
    try:
        await member.send(
            f"You have received a warning in **{interaction.guild.name}**.\n"
            f"Reason: {reason}\n"
            f"Total warnings: {count}\n\n"
            f"*Continued violations may result in removal.*"
        )
    except discord.Forbidden:
        pass

@tree.command(name="mod_warnings", description="[Mod] View a user's warnings")
@app_commands.describe(member="Member to check")
@is_mod()
async def mod_warnings(interaction: discord.Interaction, member: discord.Member):
    warnings = load_json(WARNINGS_FILE)
    uid = str(member.id)
    user_warnings = warnings.get(uid, [])
    if not user_warnings:
        await interaction.response.send_message(
            f"{member.mention} has no warnings.", ephemeral=True
        )
        return
    text = ""
    for i, w in enumerate(user_warnings, 1):
        ts = w.get("timestamp", "Unknown")[:10]
        issued_by = interaction.guild.get_member(int(w.get("issued_by", 0)))
        issued_name = issued_by.display_name if issued_by else "Unknown"
        text += f"**{i}.** {w['reason']} — by {issued_name} on {ts}\n"
    embed = discord.Embed(
        title=f"Warnings — {member.display_name} ({len(user_warnings)})",
        description=text,
        color=discord.Color.orange()
    )
    await interaction.response.send_message(embed=embed, ephemeral=True)

@tree.command(name="mod_clearwarnings", description="[Mod] Clear all warnings for a user")
@app_commands.describe(member="Member to clear warnings for")
@is_admin()
async def mod_clearwarnings(interaction: discord.Interaction, member: discord.Member):
    warnings = load_json(WARNINGS_FILE)
    uid = str(member.id)
    if uid in warnings:
        del warnings[uid]
        save_json(WARNINGS_FILE, warnings)
    await interaction.response.send_message(
        f"All warnings cleared for {member.mention}.", ephemeral=True
    )

@tree.command(name="mod_kick", description="[Mod] Kick a user from the server")
@app_commands.describe(member="Member to kick", reason="Reason for kick")
@is_mod()
async def mod_kick(
    interaction: discord.Interaction,
    member: discord.Member,
    reason: Optional[str] = "No reason provided"
):
    try:
        await member.send(
            f"You have been kicked from **{interaction.guild.name}**.\n"
            f"Reason: {reason}"
        )
    except discord.Forbidden:
        pass
    await member.kick(reason=reason)
    await interaction.response.send_message(
        f"{member.mention} kicked. Reason: **{reason}**", ephemeral=True
    )

@tree.command(name="mod_ban", description="[Mod] Ban a user from the server")
@app_commands.describe(
    member="Member to ban",
    reason="Reason for ban",
    delete_days="Days of messages to delete (0-7)"
)
@is_mod()
async def mod_ban(
    interaction: discord.Interaction,
    member: discord.Member,
    reason: Optional[str] = "No reason provided",
    delete_days: Optional[int] = 0
):
    try:
        await member.send(
            f"You have been banned from **{interaction.guild.name}**.\n"
            f"Reason: {reason}"
        )
    except discord.Forbidden:
        pass
    await member.ban(reason=reason, delete_message_days=min(delete_days, 7))
    await interaction.response.send_message(
        f"{member.mention} banned. Reason: **{reason}**", ephemeral=True
    )

@tree.command(name="mod_unban", description="[Mod] Unban a user by ID")
@app_commands.describe(user_id="The Discord user ID to unban")
@is_admin()
async def mod_unban(interaction: discord.Interaction, user_id: str):
    try:
        user = await bot.fetch_user(int(user_id))
        await interaction.guild.unban(user)
        await interaction.response.send_message(
            f"**{user.name}** unbanned.", ephemeral=True
        )
    except Exception as e:
        await interaction.response.send_message(f"Error: {e}", ephemeral=True)

@tree.command(name="mod_timeout", description="[Mod] Timeout a user for a duration")
@app_commands.describe(
    member="Member to timeout",
    minutes="Timeout duration in minutes",
    reason="Reason for timeout"
)
@is_mod()
async def mod_timeout(
    interaction: discord.Interaction,
    member: discord.Member,
    minutes: int,
    reason: Optional[str] = "No reason provided"
):
    duration = timedelta(minutes=minutes)
    await member.timeout(duration, reason=reason)
    await interaction.response.send_message(
        f"{member.mention} timed out for **{minutes} minutes**. Reason: **{reason}**",
        ephemeral=True
    )

@tree.command(name="mod_untimeout", description="[Mod] Remove timeout from a user")
@app_commands.describe(member="Member to remove timeout from")
@is_mod()
async def mod_untimeout(interaction: discord.Interaction, member: discord.Member):
    await member.timeout(None)
    await interaction.response.send_message(
        f"Timeout removed from {member.mention}.", ephemeral=True
    )

@tree.command(name="mod_purge", description="[Mod] Delete a number of messages from a channel")
@app_commands.describe(
    channel="Channel to purge",
    amount="Number of messages to delete (max 100)"
)
@is_mod()
async def mod_purge(
    interaction: discord.Interaction,
    channel: discord.TextChannel,
    amount: int
):
    amount = min(amount, 100)
    await interaction.response.send_message(
        f"Purging {amount} messages from {channel.mention}...", ephemeral=True
    )
    deleted = await channel.purge(limit=amount)
    await interaction.edit_original_response(
        content=f"Deleted **{len(deleted)}** messages from {channel.mention}."
    )

# ═══════════════════════════════════════════════════════════════════════════════
# ADMIN COMMANDS — SERVER INFO
# ═══════════════════════════════════════════════════════════════════════════════

@tree.command(name="server_info", description="Server statistics and info")
@is_admin()
async def server_info(interaction: discord.Interaction):
    guild = interaction.guild
    embed = discord.Embed(
        title=guild.name,
        color=discord.Color.from_rgb(180, 160, 120)
    )
    if guild.icon:
        embed.set_thumbnail(url=guild.icon.url)
    embed.add_field(name="Members", value=guild.member_count, inline=True)
    embed.add_field(name="Roles", value=len(guild.roles), inline=True)
    embed.add_field(name="Channels", value=len(guild.channels), inline=True)
    embed.add_field(name="Categories", value=len(guild.categories), inline=True)
    embed.add_field(name="Created", value=guild.created_at.strftime("%B %d, %Y"), inline=True)
    embed.add_field(name="Owner", value=guild.owner.mention if guild.owner else "Unknown", inline=True)
    bots = sum(1 for m in guild.members if m.bot)
    humans = guild.member_count - bots
    embed.add_field(name="Humans", value=humans, inline=True)
    embed.add_field(name="Bots", value=bots, inline=True)
    embed.set_footer(text="Fían Bán — Corp, Fís, Fír")
    await interaction.response.send_message(embed=embed, ephemeral=True)

@tree.command(name="user_info", description="[Mod] Look up info on a user")
@app_commands.describe(member="Member to look up")
@is_mod()
async def user_info(interaction: discord.Interaction, member: discord.Member):
    warnings = load_json(WARNINGS_FILE)
    warn_count = len(warnings.get(str(member.id), []))
    roles = [r.name for r in member.roles if r.name != "@everyone"]
    embed = discord.Embed(
        title=f"{member.display_name}",
        color=member.color
    )
    embed.set_thumbnail(url=member.display_avatar.url)
    embed.add_field(name="Username", value=str(member), inline=True)
    embed.add_field(name="ID", value=member.id, inline=True)
    embed.add_field(name="Joined Server", value=member.joined_at.strftime("%B %d, %Y"), inline=True)
    embed.add_field(name="Account Created", value=member.created_at.strftime("%B %d, %Y"), inline=True)
    embed.add_field(name="Warnings", value=warn_count, inline=True)
    embed.add_field(name="Timed Out", value="Yes" if member.is_timed_out() else "No", inline=True)
    if roles:
        embed.add_field(name=f"Roles ({len(roles)})", value=", ".join(roles[:20]), inline=False)
    embed.set_footer(text="Fían Bán — Corp, Fís, Fír")
    await interaction.response.send_message(embed=embed, ephemeral=True)

@tree.command(name="bot_status", description="Check if the bot is running correctly")
async def bot_status(interaction: discord.Interaction):
    latency = round(bot.latency * 1000)
    embed = discord.Embed(
        title="Fían Bán Bot — Status",
        description=f"Online. Latency: **{latency}ms**",
        color=discord.Color.green()
    )
    embed.set_footer(text="Fían Bán — Corp, Fís, Fír")
    await interaction.response.send_message(embed=embed, ephemeral=True)

# ═══════════════════════════════════════════════════════════════════════════════
# ADMIN COMMANDS — MESSAGING
# ═══════════════════════════════════════════════════════════════════════════════

@tree.command(name="announce", description="[Admin] Post a formatted announcement as the bot")
@app_commands.describe(
    channel="Channel to post in",
    title="Announcement title",
    message="Announcement body"
)
@is_admin()
async def announce(
    interaction: discord.Interaction,
    channel: discord.TextChannel,
    title: str,
    message: str
):
    embed = discord.Embed(
        title=title,
        description=message,
        color=discord.Color.from_rgb(180, 160, 120)
    )
    embed.set_footer(text="Fían Bán — Corp, Fís, Fír")
    await channel.send(embed=embed)
    await interaction.response.send_message(
        f"Announcement posted in {channel.mention}.", ephemeral=True
    )

@tree.command(name="bot_say", description="[Admin] Send a plain message as the bot to a channel")
@app_commands.describe(channel="Channel to send to", message="Message to send")
@is_admin()
async def bot_say(
    interaction: discord.Interaction,
    channel: discord.TextChannel,
    message: str
):
    await channel.send(message)
    await interaction.response.send_message("Message sent.", ephemeral=True)

@tree.command(name="pin_message", description="[Mod] Pin a message by its ID")
@app_commands.describe(
    channel="Channel containing the message",
    message_id="ID of the message to pin"
)
@is_mod()
async def pin_message(
    interaction: discord.Interaction,
    channel: discord.TextChannel,
    message_id: str
):
    try:
        msg = await channel.fetch_message(int(message_id))
        await msg.pin()
        await interaction.response.send_message("Message pinned.", ephemeral=True)
    except Exception as e:
        await interaction.response.send_message(f"Error: {e}", ephemeral=True)

@tree.command(name="unpin_message", description="[Mod] Unpin a message by its ID")
@app_commands.describe(
    channel="Channel containing the message",
    message_id="ID of the message to unpin"
)
@is_mod()
async def unpin_message(
    interaction: discord.Interaction,
    channel: discord.TextChannel,
    message_id: str
):
    try:
        msg = await channel.fetch_message(int(message_id))
        await msg.unpin()
        await interaction.response.send_message("Message unpinned.", ephemeral=True)
    except Exception as e:
        await interaction.response.send_message(f"Error: {e}", ephemeral=True)

@tree.command(name="poll", description="[Admin] Create a simple poll")
@app_commands.describe(
    channel="Channel to post poll in",
    question="Poll question",
    option1="First option",
    option2="Second option",
    option3="Third option (optional)",
    option4="Fourth option (optional)"
)
@is_admin()
async def poll(
    interaction: discord.Interaction,
    channel: discord.TextChannel,
    question: str,
    option1: str,
    option2: str,
    option3: Optional[str] = None,
    option4: Optional[str] = None
):
    options = [option1, option2]
    if option3:
        options.append(option3)
    if option4:
        options.append(option4)

    number_emojis = ["1️⃣", "2️⃣", "3️⃣", "4️⃣"]
    description = ""
    for i, opt in enumerate(options):
        description += f"{number_emojis[i]} {opt}\n"

    embed = discord.Embed(
        title=f"📊 {question}",
        description=description,
        color=discord.Color.from_rgb(180, 160, 120)
    )
    embed.set_footer(text="Fían Bán Poll — React to vote")
    msg = await channel.send(embed=embed)
    for i in range(len(options)):
        await msg.add_reaction(number_emojis[i])
    await interaction.response.send_message(
        f"Poll posted in {channel.mention}.", ephemeral=True
    )

# ═══════════════════════════════════════════════════════════════════════════════
# ADMIN COMMANDS — ONBOARDING
# ═══════════════════════════════════════════════════════════════════════════════

@tree.command(name="post_rules", description="[Admin] Post the oath message in current channel")
@is_admin()
async def post_rules(interaction: discord.Interaction):
    embed = discord.Embed(
        title="Fían Bán — Conduct and Oath",
        description=RULES_TEXT,
        color=discord.Color.from_rgb(180, 160, 120)
    )
    embed.set_footer(text="React ✅ to swear the oath and gain access to the community.")
    await interaction.response.send_message("Posting oath...", ephemeral=True)
    msg = await interaction.channel.send(embed=embed)
    await msg.add_reaction("✅")

@tree.command(name="post_roles", description="[Admin] Post role selection in current channel")
@is_admin()
async def post_roles(interaction: discord.Interaction):
    await interaction.response.send_message("Posting role selection...", ephemeral=True)

    embed1 = discord.Embed(
        title="🏴 Tradition — Select Your Path",
        description=(
            "Select your tradition. You may select more than one.\n\n"
            "🍀 **Irish** — Gaelic Irish tradition\n"
            "🏴󠁧󠁢󠁳󠁣󠁴󠁿 **Scottish** — Gaelic Scottish tradition\n"
            "🏴󠁧󠁢󠁷󠁬󠁳󠁿 **Welsh** — Welsh/Brittonic tradition\n"
            "⚜️ **Brythonic** — Continental Celtic/Gaulish tradition\n"
            "🔱 **Other** — Other reconstructionist path\n\n"
            "*React to select. React again to remove.*"
        ),
        color=discord.Color.from_rgb(180, 160, 120)
    )
    msg1 = await interaction.channel.send(embed=embed1)
    for emoji in ["🍀", "🏴󠁧󠁢󠁳󠁣󠁴󠁿", "🏴󠁧󠁢󠁷󠁬󠁳󠁿", "⚜️", "🔱"]:
        await msg1.add_reaction(emoji)

    embed2 = discord.Embed(
        title="⚔️ Vocation — Your Path",
        description=(
            "Select your vocation. You may select more than one.\n\n"
            "🪖 **Míliath** (Warrior/Military) — Active duty\n"
            "🎖️ **Seansaighdiúir** (Veteran) — Military veteran\n"
            "🛡️ **Caomhnóir** (Guardian) — Law enforcement / First responder\n"
            "🏹 **Sealgaire** (Hunter) — Civilian practitioner\n"
            "📖 **Filidh** (Poet-Scholar) — Bardic / scholarly focus\n"
            "🌀 **Draoi** (Devotional) — Spiritual / devotional focus\n\n"
            "*React to select. React again to remove.*"
        ),
        color=discord.Color.from_rgb(180, 160, 120)
    )
    msg2 = await interaction.channel.send(embed=embed2)
    for emoji in ["🪖", "🎖️", "🛡️", "🏹", "📖", "🌀"]:
        await msg2.add_reaction(emoji)

    embed3 = discord.Embed(
        title="📋 Profile — Skills and Background",
        description=(
            "Add your background to your profile so the community knows "
            "what skills we have available.\n\n"
            "`/profile_setbranch` — Military branch\n"
            "`/profile_setmos` — MOS, rate, or military job\n"
            "`/profile_setjob` — Civilian occupation\n\n"
            "`/profile` — View your own profile\n"
            "`/profile [member]` — View someone else's profile"
        ),
        color=discord.Color.from_rgb(180, 160, 120)
    )
    await interaction.channel.send(embed=embed3)

@tree.command(name="welcome_resend", description="[Admin] Re-send welcome message for a member")
@app_commands.describe(member="Member to re-welcome")
@is_admin()
async def welcome_resend(interaction: discord.Interaction, member: discord.Member):
    welcome_ch = discord.utils.get(interaction.guild.text_channels, name=WELCOME_CHANNEL)
    rules_ch = discord.utils.get(interaction.guild.text_channels, name=RULES_CHANNEL)
    if not welcome_ch:
        await interaction.response.send_message(
            "No #welcome channel found.", ephemeral=True
        )
        return
    embed = discord.Embed(
        title="Welcome to Fían Bán",
        description=(
            f"Fáilte, {member.mention}.\n\n"
            f"Read and swear the oath in "
            f"{rules_ch.mention if rules_ch else '#rules-and-oath'} "
            f"to gain access to the community.\n\n"
            f"*The Fianna did not admit anyone who had not first sworn. Neither do we.*"
        ),
        color=discord.Color.from_rgb(220, 220, 200)
    )
    embed.set_footer(text="Fían Bán — Corp, Fís, Fír")
    await welcome_ch.send(embed=embed)
    await interaction.response.send_message(
        f"Welcome message re-sent for {member.mention}.", ephemeral=True
    )

# ═══════════════════════════════════════════════════════════════════════════════
# PROFILE COMMANDS
# ═══════════════════════════════════════════════════════════════════════════════

@tree.command(name="profile_setbranch", description="Set your military branch")
@app_commands.choices(branch=[
    app_commands.Choice(name="Army", value="Army"),
    app_commands.Choice(name="Navy", value="Navy"),
    app_commands.Choice(name="Marine Corps", value="Marine Corps"),
    app_commands.Choice(name="Air Force", value="Air Force"),
    app_commands.Choice(name="Coast Guard", value="Coast Guard"),
    app_commands.Choice(name="Space Force", value="Space Force"),
    app_commands.Choice(name="National Guard", value="National Guard"),
    app_commands.Choice(name="Reserves", value="Reserves"),
])
async def profile_setbranch(interaction: discord.Interaction, branch: str):
    profiles = load_json(PROFILES_FILE)
    uid = str(interaction.user.id)
    if uid not in profiles:
        profiles[uid] = {}
    profiles[uid]["branch"] = branch
    save_json(PROFILES_FILE, profiles)
    await interaction.response.send_message(f"Branch set to **{branch}**.", ephemeral=True)

@tree.command(name="profile_setmos", description="Set your MOS, rate, or military job")
@app_commands.describe(mos="Your MOS, rate, AFSC, or military job title")
async def profile_setmos(interaction: discord.Interaction, mos: str):
    profiles = load_json(PROFILES_FILE)
    uid = str(interaction.user.id)
    if uid not in profiles:
        profiles[uid] = {}
    profiles[uid]["mos"] = mos
    save_json(PROFILES_FILE, profiles)
    await interaction.response.send_message(f"Military job set to **{mos}**.", ephemeral=True)

@tree.command(name="profile_setjob", description="Set your civilian occupation")
@app_commands.describe(job="Your occupation or primary skills")
async def profile_setjob(interaction: discord.Interaction, job: str):
    profiles = load_json(PROFILES_FILE)
    uid = str(interaction.user.id)
    if uid not in profiles:
        profiles[uid] = {}
    profiles[uid]["job"] = job
    save_json(PROFILES_FILE, profiles)
    await interaction.response.send_message(f"Occupation set to **{job}**.", ephemeral=True)

@tree.command(name="profile", description="View a member's profile")
@app_commands.describe(member="Member to view (leave blank for yourself)")
async def profile(
    interaction: discord.Interaction,
    member: Optional[discord.Member] = None
):
    target = member or interaction.user
    profiles = load_json(PROFILES_FILE)
    warnings = load_json(WARNINGS_FILE)
    uid = str(target.id)
    data = profiles.get(uid, {})
    warn_count = len(warnings.get(uid, []))

    roles = [r.name for r in target.roles if r.name not in ["@everyone", BASE_ROLE, VERIFIED_ROLE]]

    embed = discord.Embed(
        title=target.display_name,
        color=discord.Color.from_rgb(180, 160, 120)
    )
    embed.set_thumbnail(url=target.display_avatar.url)

    if roles:
        embed.add_field(name="Roles", value=", ".join(roles), inline=False)
    if data.get("branch"):
        embed.add_field(name="Branch", value=data["branch"], inline=True)
    if data.get("mos"):
        embed.add_field(name="MOS / Job", value=data["mos"], inline=True)
    if data.get("job"):
        embed.add_field(name="Occupation", value=data["job"], inline=True)
    if not roles and not data:
        embed.description = (
            "No profile set. Use `/profile_setbranch`, "
            "`/profile_setmos`, or `/profile_setjob`."
        )
    embed.set_footer(text="Fían Bán — Corp, Fís, Fír")
    await interaction.response.send_message(embed=embed)

# ═══════════════════════════════════════════════════════════════════════════════
# REMINDERS AND SCHEDULED EVENTS
# ═══════════════════════════════════════════════════════════════════════════════

@tree.command(name="reminder_set", description="[Admin] Set a reminder in a channel at a specific time")
@app_commands.describe(
    channel="Channel to post the reminder in",
    message="Reminder message",
    minutes="Minutes from now to post the reminder"
)
@is_admin()
async def reminder_set(
    interaction: discord.Interaction,
    channel: discord.TextChannel,
    message: str,
    minutes: int
):
    reminders = load_json(REMINDERS_FILE)
    reminder_id = str(len(reminders) + 1)
    fire_time = (datetime.utcnow() + timedelta(minutes=minutes)).isoformat()
    reminders[reminder_id] = {
        "channel_id": channel.id,
        "message": message,
        "fire_time": fire_time,
        "guild_id": interaction.guild.id
    }
    save_json(REMINDERS_FILE, reminders)
    await interaction.response.send_message(
        f"Reminder set for **{minutes} minutes** from now in {channel.mention}.",
        ephemeral=True
    )

@tree.command(name="event_create", description="[Admin] Create a scheduled Discord event")
@app_commands.describe(
    name="Event name",
    description="Event description",
    channel="Voice channel for the event (optional)",
    hours_from_now="Hours until event starts"
)
@is_admin()
async def event_create(
    interaction: discord.Interaction,
    name: str,
    description: str,
    hours_from_now: int,
    channel: Optional[discord.VoiceChannel] = None
):
    start_time = datetime.utcnow() + timedelta(hours=hours_from_now)
    end_time = start_time + timedelta(hours=2)
    try:
        if channel:
            event = await interaction.guild.create_scheduled_event(
                name=name,
                description=description,
                start_time=start_time,
                end_time=end_time,
                channel=channel,
                entity_type=discord.EntityType.voice
            )
        else:
            event = await interaction.guild.create_scheduled_event(
                name=name,
                description=description,
                start_time=start_time,
                end_time=end_time,
                entity_type=discord.EntityType.external,
                location="Fían Bán Discord"
            )
        await interaction.response.send_message(
            f"Event **{event.name}** created, starting in {hours_from_now} hours.",
            ephemeral=True
        )
    except Exception as e:
        await interaction.response.send_message(f"Error creating event: {e}", ephemeral=True)

@tree.command(name="event_list", description="List upcoming scheduled events")
async def event_list(interaction: discord.Interaction):
    events = interaction.guild.scheduled_events
    if not events:
        await interaction.response.send_message(
            "No upcoming events.", ephemeral=True
        )
        return
    embed = discord.Embed(
        title="Upcoming Events",
        color=discord.Color.from_rgb(180, 160, 120)
    )
    for event in events:
        start = event.start_time.strftime("%B %d at %H:%M UTC")
        embed.add_field(
            name=event.name,
            value=f"{event.description or 'No description'}\n**Starts:** {start}",
            inline=False
        )
    embed.set_footer(text="Fían Bán — Corp, Fís, Fír")
    await interaction.response.send_message(embed=embed)

# ═══════════════════════════════════════════════════════════════════════════════
# CALENDAR TASK
# ═══════════════════════════════════════════════════════════════════════════════

@tasks.loop(hours=24)
async def check_calendar():
    today = date.today()
    for guild in bot.guilds:
        announce_channel = (
            discord.utils.get(guild.text_channels, name="announcements") or
            discord.utils.get(guild.text_channels, name="general") or
            discord.utils.get(guild.text_channels, name="warrior-calendar")
        )
        if not announce_channel:
            continue
        for festival in IRISH_CALENDAR:
            m, d = festival["date"]
            festival_date = date(today.year, m, d)
            days_until = (festival_date - today).days
            if days_until in [3, 0]:
                if days_until == 3:
                    title = f"⚔️ {festival['name']} — 3 days"
                else:
                    title = f"🔥 {festival['name']} — Today"
                embed = discord.Embed(
                    title=title,
                    description=festival["description"],
                    color=discord.Color.from_rgb(180, 160, 120)
                )
                embed.set_footer(text="Fían Bán — Corp, Fís, Fír")
                await announce_channel.send(embed=embed)

@tasks.loop(minutes=1)
async def check_reminders():
    reminders = load_json(REMINDERS_FILE)
    now = datetime.utcnow()
    fired = []
    for rid, reminder in reminders.items():
        fire_time = datetime.fromisoformat(reminder["fire_time"])
        if now >= fire_time:
            channel = bot.get_channel(reminder["channel_id"])
            if channel:
                embed = discord.Embed(
                    title="⏰ Reminder",
                    description=reminder["message"],
                    color=discord.Color.from_rgb(180, 160, 120)
                )
                embed.set_footer(text="Fían Bán — Corp, Fís, Fír")
                await channel.send(embed=embed)
            fired.append(rid)
    for rid in fired:
        del reminders[rid]
    if fired:
        save_json(REMINDERS_FILE, reminders)

# ═══════════════════════════════════════════════════════════════════════════════
# HELP
# ═══════════════════════════════════════════════════════════════════════════════

@tree.command(name="fianhelp", description="List all bot commands")
async def fianhelp(interaction: discord.Interaction):
    is_admin_user = (
        interaction.user.guild_permissions.administrator or
        interaction.user.guild_permissions.manage_guild
    )
    text = """
**Fían Bán Bot — Commands**

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Profile (everyone):**
`/profile` — View your or another member's profile
`/profile_setbranch` — Set military branch
`/profile_setmos` — Set MOS or military job
`/profile_setjob` — Set civilian occupation

**Events (everyone):**
`/event_list` — View upcoming scheduled events
`/bot_status` — Check bot status
"""
    if is_admin_user:
        text += """
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Roles [Admin]:**
`/role_create` `/role_edit` `/role_delete`
`/role_assign` `/role_remove`
`/role_list` `/role_members`
`/role_grant_sworn` `/role_revoke_sworn`

**Channels [Admin]:**
`/channel_create` `/channel_delete` `/channel_rename`
`/channel_topic` `/channel_clone` `/channel_move`
`/channel_lock` `/channel_unlock` `/channel_readonly`
`/channel_slowmode` `/channel_list`

**Categories [Admin]:**
`/category_create` `/category_delete` `/category_rename`

**Moderation [Mod]:**
`/mod_warn` `/mod_warnings` `/mod_clearwarnings`
`/mod_kick` `/mod_ban` `/mod_unban`
`/mod_timeout` `/mod_untimeout`
`/mod_purge` `/user_info`

**Messaging [Admin]:**
`/announce` `/bot_say` `/poll`
`/pin_message` `/unpin_message`

**Onboarding [Admin]:**
`/post_rules` `/post_roles` `/welcome_resend`

**Server [Admin]:**
`/server_info` `/reminder_set` `/event_create`
"""
    embed = discord.Embed(
        description=text,
        color=discord.Color.from_rgb(180, 160, 120)
    )
    embed.set_footer(text="Fían Bán — Corp, Fís, Fír")
    await interaction.response.send_message(embed=embed, ephemeral=True)

# ─── RUN ──────────────────────────────────────────────────────────────────────
if __name__ == "__main__":
    bot.run(TOKEN)
