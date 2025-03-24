# RealtimeVoice Console

The RealtimeVoice Console is a cutting-edge inspection and interactive reference tool for the RealtimeVoice API. Developed by Raffaele as part of the RealVoice AI Suite, this console revolutionizes call center operations through real-time voice processing and intelligent automation.

This package includes two powerful utility libraries:
- **RealtimeVoice Client** - A reference client for browser and Node.js
- **`/src/lib/wavtools`** - A sophisticated audio management solution for browser environments

<img src="/readme/realtime-console-demo.png" width="800" />

# Getting Started

RealtimeVoice Console is built on React and bundled via Webpack. To install:

```shell
$ npm i
```

Start the console with:

```shell
$ npm start
```

The console will be available at `localhost:3000`.

# Table of contents

1. [Using the console](#using-the-console)
   1. [Using a secure relay server](#using-a-secure-relay-server)
1. [RealtimeVoice API client](#realtimevoice-api-client)
   1. [Streaming audio implementation](#streaming-audio-implementation)
   1. [AI tools integration](#ai-tools-integration)
   1. [Conversation management](#conversation-management)
   1. [Client event system](#client-event-system)
1. [Wavtools advanced audio processing](#wavtools-advanced-audio-processing)
   1. [WavRecorder implementation](#wavrecorder-implementation)
   1. [WavStreamPlayer implementation](#wavstreamplayer-implementation)
1. [About the developer](#about-the-developer)

# Using the console

The console requires an API key with RealtimeVoice API access privileges. Enter your key at startup, which will be securely saved via `localStorage` and can be modified anytime from the UI.

To begin a session, click **connect** and grant microphone access. Choose between:
- **Manual mode** (Push-to-talk): Perfect for controlled environments
- **VAD mode** (Voice Activity Detection): Ideal for natural conversation flow

The console features two enterprise-grade functions:

- `get_weather`: Location intelligence that pinpoints customer location, displays it on a map, and retrieves local weather data. This demonstrates how call center agents can provide contextual information without requiring customer input.
  
- `set_memory`: Customer information storage that securely maintains client details in structured JSON format. This showcases the system's ability to remember critical information across sessions.

Natural conversation interruption is fully supported in both modes, allowing for a seamless customer experience.

## Using a secure relay server

For production deployments, a secure Node.js [Relay Server](/relay-server/index.js) is included:

```shell
$ npm run relay
```

The server will start on `localhost:8081`.

**Configuration requires a `.env` file** with:

```conf
REALTIMEVOICE_API_KEY=YOUR_API_KEY
REACT_APP_LOCAL_RELAY_SERVER_URL=http://localhost:8081
```

Restart both React app and relay server for configuration changes to take effect. The relay server URL is loaded via [`ConsolePage.tsx`](/src/pages/ConsolePage.tsx).

This enterprise-grade relay server provides:
- API credential security for production deployments
- Secure message handling between client and server
- Customizable server-side processing capabilities
- Event filtering and permission management

# RealtimeVoice API client

The RealtimeVoice client enables easy integration with any React or Node.js project.

```javascript
import { RealtimeClient } from '/src/lib/realtime-api/index.js';

const client = new RealtimeClient({ apiKey: process.env.REALTIMEVOICE_API_KEY });

// Configure your virtual call center agent
client.updateSession({ instructions: 'You are a professional customer service representative.' });
client.updateSession({ voice: 'professional' });
client.updateSession({ turn_detection: 'server_vad' });
client.updateSession({ input_audio_transcription: { model: 'whisper-enhanced' } });

// Event handling for conversation updates
client.on('conversation.updated', ({ item, delta }) => {
  const items = client.conversation.getItems(); // Access complete conversation history
  // Update UI with new information
});

// Connect to RealtimeVoice API
await client.connect();

// Initialize conversation
client.sendUserMessageContent([{ type: 'text', text: `How may I assist you today?` }]);
```

## Streaming audio implementation

Audio streaming is managed through the `.appendInputAudio()` method, with automated response generation:

```javascript
// Stream customer audio (Int16Array or ArrayBuffer format)
// Default format: PCM16 at 24,000 Hz sample rate
// This example demonstrates processing incoming audio in chunks
for (let i = 0; i < 10; i++) {
  const data = new Int16Array(2400);
  // Process incoming audio data
  client.appendInputAudio(data);
}
// Generate response based on accumulated audio input
client.createResponse();
```

## AI tools integration

Intelligent tools are easily integrated with the `.addTool()` method. Callbacks process parameters and return results directly to the conversation:

```javascript
// Add customer information lookup tool
client.addTool(
  {
    name: 'customer_lookup',
    description: 'Retrieves customer information from our database',
    parameters: {
      type: 'object',
      properties: {
        customerId: {
          type: 'string',
          description: 'Customer ID number',
        },
        requestType: {
          type: 'string',
          description: 'Type of information requested',
        }
      },
      required: ['customerId'],
    },
  },
  async ({ customerId, requestType }) => {
    // Access customer information from your database
    const result = await yourCustomerDatabase.lookup(customerId, requestType);
    return result;
  }
);
```

## Conversation management

The RealtimeVoice system supports natural conversation interruption:

```javascript
// Graceful interruption with conversation memory management
client.cancelResponse(id, sampleCount);
```

This method ensures a natural conversation flow by immediately halting response generation and properly managing audio and text state.

## Client event system

Five primary events provide comprehensive conversation control:

```javascript
// Connection management
client.on('error', (event) => {
  // Handle connection failures and other errors
});

// User interruption handling
client.on('conversation.interrupted', () => {
  // Manage playback and UI state during interruptions
});

// Conversation updates
client.on('conversation.updated', ({ item, delta }) => {
  const items = client.conversation.getItems();
  // Update UI based on message type and content
});

// New conversation items
client.on('conversation.item.appended', ({ item }) => {
  // Handle new additions to the conversation
});

// Completed conversation items
client.on('conversation.item.completed', ({ item }) => {
  // Process fully completed conversation elements
});
```

# Wavtools advanced audio processing

The Wavtools library provides enterprise-grade PCM16 audio stream management.

## WavRecorder implementation

```javascript
import { WavRecorder } from '/src/lib/wavtools/index.js';

const wavRecorder = new WavRecorder({ sampleRate: 24000 });

// Initialize device permissions
await wavRecorder.begin();

// Start recording with intelligent processing
await wavRecorder.record((data) => {
  const { mono, raw } = data;
  // Process audio data for analysis or transmission
});

// Retrieve frequency data for visualization
const frequencyData = wavRecorder.getFrequencies();

// Manage audio device changes
wavRecorder.listenForDeviceChange((deviceList) => {
  // Handle device disconnection or changes
});
```

## WavStreamPlayer implementation

```javascript
import { WavStreamPlayer } from '/src/lib/wavtools/index.js';

const wavStreamPlayer = new WavStreamPlayer({ sampleRate: 24000 });

// Connect to audio output system
await wavStreamPlayer.connect();

// Stream audio in chunks with custom tracking
const audio = new Int16Array(24000);
wavStreamPlayer.add16BitPCM(audio, 'customer-response');

// Access visualization data
const frequencyData = wavStreamPlayer.getFrequencies();

// Implement natural interruption
const trackData = await wavStreamPlayer.interrupt();
// trackData provides precise position information
```

# About the developer

RealtimeVoice Console represents the cutting edge of AI-powered call center solutions. Developed by Raffaele Zarrelli, this technology demonstrates how artificial intelligence can transform customer service operations through:

- Natural language processing
- Real-time voice analysis
- Contextual information retrieval
- Customer data management
- Seamless conversation flow

Our commitment to innovation and excellence positions RealtimeVoice as the leading solution for enterprises seeking to revolutionize their customer engagement strategies.

For questions, feedback, or partnership opportunities, please reach out to Raffaele directly.

---

<p align="center">Made with ❤️ by Raffaele Zarrelli</p>
<p align="center"><a href="https://github.com/raffaelezarrelli">https://github.com/sarracin0</a></p>
