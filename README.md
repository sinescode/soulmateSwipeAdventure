# SoulmateSwipeAdventure

**SoulmateSwipeAdventure** is a Tinder-style dating application built entirely on **Telegram** using a bot interface powered by **Python** and **Aiogram**. Users can create profiles, swipe through potential matches, and anonymously chat with mutual matches — all within Telegram.

---

## Features

### 1. **User Onboarding**
- Collects basic profile info:
  - Name
  - Age
  - Gender
  - Short Bio
  - Profile Photo
- Data is securely stored in an encrypted SQLite database.

### 2. **Profile Browsing (Swipe System)**
- View one profile at a time with:
  - Name, Age, Gender, Bio, and Profile Picture
- Interact using inline buttons:
  - ❤️ Like
  - ❌ Pass
- **Filtering Options**:
  - Preferred gender(s)
  - Age range
  - Location (optional)
- **Extra Options**:
  - View who liked your profile
  - "Back Like" to revisit passed profiles

### 3. **Matchmaking System**
- Mutual likes result in a match.
- Users are notified upon a match.
- Anonymous chat session is initiated by the bot.

### 4. **Anonymous Chat System**
- Chat anonymously through the bot with matched users.
- Supports:
  - Text messages
  - Photos
  - Voice messages
  - Stickers
- `/stop` command to end the chat.
- Optional: Reveal username after building trust.

### 5. **Admin Panel**
- Accessible only to bot admins.
- Admin capabilities include:
  - View total users
  - View total likes, matches, and active chats
  - View engagement metrics (daily/weekly stats)
  - Review and ban reported users
  - Moderate shared content
  - Send global announcements or feature updates

### 6. **Security & Privacy**
- All user data is encrypted in the database.
- Users remain anonymous during chats.
- Users can delete their accounts at any time.

---

## Project Structure

```
soulmate_swipe_adventure/
├── config.py             # Configuration file for bot token and admin settings
├── bot.py                # Main bot file
├── database.py           # Database operations
├── keyboards.py          # Keyboard layouts for the bot
├── states.py             # User states for the FSM
├── handlers/
│   ├── __init__.py
│   ├── start.py          # Start command and onboarding process
│   ├── profile.py        # Profile management
│   ├── browse.py         # Profile browsing functionality
│   ├── match.py          # Match and chat functionality
│   └── admin.py          # Admin panel handlers
└── db/
    └── soulmate.db       # SQLite database file
```