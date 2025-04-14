# **SoulmateSwipeAdventure - Project Description**

## **Overview**

**SoulmateSwipeAdventure** is a Tinder-style dating application built entirely within **Telegram** using a bot interface. Users can browse profiles, like or pass on others, and get matched if both users like each other. The bot facilitates anonymous chat between matched users while maintaining user privacy.

---

## **Features**

### **1. User Onboarding**
- **Onboarding Process**: 
   - When a user starts the bot, they will be asked to provide the following information:
     - **Name**
     - **Age**
     - **Gender**
     - **Short Bio**
     - **Profile Photo**
   - This data is stored securely in an **SQLite** database.

### **2. Profile Browsing**
- **Browsing Interface**: 
   - Users can view profiles of other users one at a time. Each profile includes:
     - Name
     - Age
     - Gender
     - Bio
     - Profile Photo
   - **Interactive Buttons**: 
     - **❤️ Like**: To express interest in a user.
     - **❌ Pass**: To move to the next user.
   - **Filter Options**: 
     - Users can set their browsing preferences by selecting filters for:
       - Preferred gender(s)
       - Age range
       - Location (optional, if location sharing is enabled)

### **3. Matchmaking**
- **Like and Match Process**:
   - When a user taps **❤️ Like**, the liked user is notified of the interaction.
   - If **both users like each other**, the bot:
     - Notifies both users that they’ve been **matched**.
     - Opens a **private, bot-mediated chat channel** for the matched users to communicate.

### **4. Chat System**
- **Anonymous Messaging**: 
   - The bot handles all messaging between matched users to maintain anonymity.
   - Users can send:
     - **Text messages**
     - **Photos**
     - **Voice messages**
     - **Stickers**
   - Users can stop the chat at any time by typing `/stop`, which ends the conversation.
   - **Optional Feature**: Allow users to reveal their usernames after gaining trust through continued interaction.

### **5. Admin Panel**
- **Admin Interface**: 
   - The bot includes an **Admin Panel** to provide control and management over the bot’s functions and user interactions.
   - **Access Control**: The Admin Panel is accessible only to the **admin(s)**, ensuring proper oversight of the platform.
   - **Features for Admin**:
     - **View Total Number of Users**: Track the total number of users registered on the platform.
     - **Ban/Report System**: 
       - Admin can review reports or complaints about users.
       - Ban users who violate the terms of service or engage in inappropriate behavior.
     - **Dashboard with Key Stats**: 
       - **Total Likes**: Track the total number of "Likes" given.
       - **Total Matches**: Monitor the number of successful matches made.
       - **Active Chats**: View the number of ongoing private chat sessions.
       - **Engagement Metrics**: Track daily and weekly user activity, such as the number of active users, matches, and chats.
     - **Content Moderation**: Admin can moderate content sent in chats, including photos, texts, and stickers, to ensure that all communications follow community guidelines.
     - **User Insights**: Analyze user demographics, such as age range, location (if provided), and gender, to gain insights into the platform’s usage.

---

## **Security and Privacy**
- **Data Encryption**: All user data (such as name, age, bio, etc.) is encrypted and stored securely in the database.
- **Anonymity**: The bot mediates all communication, ensuring that users remain anonymous until they decide to reveal their identities. No personal details are shared between users without consent.
- **User Control**: Users have control over their profiles and can delete their accounts at any time.

---

## **Technologies Used**
- **Telegram Bot API** for communication.
- **SQLite** database for storing user data.
- **Python (with Aiogram)** for backend bot logic.
- **Hosting**: Can be hosted on a cloud service (like Heroku, AWS, etc.).
