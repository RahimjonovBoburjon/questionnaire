# Interactive Questionnaire Modal

A Vue.js application with an interactive questionnaire modal that sends responses to a Telegram bot.

## Features

- 🎯 Multi-step questionnaire with 4 questions
- 📱 Responsive design with Tailwind CSS
- 🔄 Progress tracking with visual progress bar
- ✅ Confirmation dialog before submission
- 📨 Telegram Bot integration for receiving responses
- 🎨 Modern UI with smooth animations

## Setup

### 1. Install Dependencies
```bash
npm install
```

### 2. Environment Configuration

Create a `.env` file in the root directory with your Telegram bot credentials:

```env
VITE_TELEGRAM_BOT_TOKEN=your_bot_token_here
VITE_TELEGRAM_ADMIN_CHAT_ID=your_chat_id_here
```

#### Getting Telegram Bot Credentials:

1. **Create a Bot:**
   - Message @BotFather on Telegram
   - Use `/newbot` command
   - Follow instructions to create your bot
   - Copy the bot token

2. **Get Your Chat ID:**
   - Send any message to your bot
   - Visit: `https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates`
   - Find your `chat_id` in the response

### 3. Run Development Server
```bash
npm run dev
```

## Security

- ✅ Environment variables for sensitive data
- ✅ `.env` file is gitignored
- ✅ No hardcoded credentials in source code

## Tech Stack

- **Frontend:** Vue.js 3 + Vite
- **Styling:** Tailwind CSS
- **Backend Integration:** Telegram Bot API
