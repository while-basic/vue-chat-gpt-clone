# Vue ChatGPT Clone

A modern chat interface built with Vue.js and the OpenAI API, featuring a responsive design and real-time message streaming.

## Features

- 🎨 Modern, responsive UI with dark mode
- ⚡ Real-time message streaming
- 📱 Mobile-friendly design
- 🔒 Secure API key handling
- ⌨️ Markdown support
- 🔄 Auto-scroll and focus management

## Setup

1. Clone the repository:
```bash
git clone https://github.com/while-basic/vue-chat-gpt-clone.git
cd vue-chat-gpt-clone
```

2. Install dependencies:
```bash
npm install
```

3. Create a `.env` file:
```bash
cp .env.example .env
```

4. Add your OpenAI API key to the `.env` file:
```
VITE_OPENAI_API_KEY=your_openai_api_key_here
```

5. Start the development server:
```bash
npm run dev
```

## Build for Production

To create a production build:

```bash
npm run build
```

## Technologies Used

- Vue.js 3
- TypeScript
- Vite
- OpenAI API
- Font Awesome

## License

MIT
