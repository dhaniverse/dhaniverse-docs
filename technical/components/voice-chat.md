# Voice Chat Implementation with LiveKit

This implementation adds voice communication features to the Dhaniverse game using LiveKit.

## Features

### Discord-style Voice Controls (UI)

Voice is controlled via buttons placed directly above the chat input in the Game HUD:

- Connect/Disconnect: Join or leave the voice room
- Mic: Toggle your microphone (start/stop speaking)
- Listen: Mute/Unmute others
- Deaf: Completely disable voice (disconnect and mute)

Each control has clear color/state feedback and tooltips. No slash commands are used.

### Voice Indicators

- Real-time speaking indicators for all participants
- Audio level visualization
- Connection status for each player
- Visual feedback for voice state changes

## Setup

### 1. LiveKit Server

You need a LiveKit server. You can:

- Use LiveKit Cloud (recommended for production)
- Run a local LiveKit server for development

### 2. Environment Variables

Add these to your `.env` file:

```bash
# LiveKit Configuration
VITE_LIVEKIT_SERVER_URL=wss://your-livekit-server.com
VITE_LIVEKIT_API_KEY=your_api_key
VITE_LIVEKIT_API_SECRET=your_api_secret
```

For development with a local LiveKit server:

```bash
VITE_LIVEKIT_SERVER_URL=ws://localhost:7880
VITE_LIVEKIT_API_KEY=dev-key
VITE_LIVEKIT_API_SECRET=dev-secret
```

### 3. Install LiveKit

The required packages are already installed:

- `livekit-client` - LiveKit client SDK
- `@livekit/components-react` - React components for LiveKit

## Usage

### In GameHUD Component

Controls are rendered just above the chat input via `ChatVoiceControls`:

```tsx
<GameHUD
    username="Player1"
    voiceEnabled={true}
    // ... other props
/>
```

`GameHUD` integrates `ChatVoiceControls` with the correct `roomName` and `participantName` for you.

### Voice Service Integration

The `VoiceChatService` handles all voice operations:

```typescript
import { voiceChatService } from '../services/VoiceChatService';

// Initialize
await voiceChatService.initialize(config);

// Connect to voice chat
await voiceChatService.connect();

// Start speaking
await voiceChatService.startSpeaking();

// Toggle listening
voiceChatService.toggleListening();

// Disconnect
await voiceChatService.disconnect();
```

### Voice Integration Handler

The `VoiceCommandHandler` integrates voice with game events (no chat commands):

```typescript
import { voiceCommandHandler } from '../services/VoiceCommandHandler';

// Initialize
await voiceCommandHandler.initialize("room-name", "username");

// The handler broadcasts state changes and listens to player join/leave events.
```

## Architecture

### Services

1. **VoiceChatService** (`src/services/VoiceChatService.ts`)
   - Core LiveKit integration
   - Room management
   - Audio track handling
   - Voice activity detection

2. **VoiceCommandHandler** (`src/services/VoiceCommandHandler.ts`)
   - Game event integration
   - State synchronization and broadcasting

### Components

1. **ChatVoiceControls** (`src/ui/components/voice/ChatVoiceControls.tsx`)
   - Discord-style voice control UI (Connect, Mic, Listen, Deaf)
   - Rendered directly above the chat input in `GameHUD`

2. **GameHUD** (`src/ui/components/hud/GameHUD.tsx`)
   - Hosts the chat window and places `ChatVoiceControls` above the input
   - Integration with game events
   - System message display

Note: The previous `VoiceChat` HUD component is now unused.

### Hooks

1. **useVoiceChat** (`src/ui/hooks/useVoiceChat.ts`)
   - React hook for voice state management
   - Voice control methods
   - Error handling

## Events

The voice system integrates with the game's event system:

### Dispatched Events

- `voice-speaking-start` - When player starts speaking
- `voice-speaking-stop` - When player stops speaking
- `voice-listening-change` - When listening state changes
- `voice-deaf-change` - When deaf mode changes
- `voice-state-change` - When overall voice state changes
- `voice-error` - When voice errors occur

### Listened Events

- `playerJoined` - New player joins game
- `playerDisconnect` - Player leaves game

## Browser Permissions

Voice chat requires microphone permission. The browser will prompt users the first time they try to speak.

## Security Notes

Important: The current token generation is for development only. In production:

1. Generate LiveKit tokens on your backend server
2. Never expose API secrets in frontend code
3. Implement proper authentication and authorization
4. Use HTTPS for all voice communications

## Troubleshooting

### Common Issues

1. Microphone not working
   - Check browser permissions
   - Ensure HTTPS (required for microphone access)
   - Try refreshing the page

2. Cannot hear others
   - Check if listening is enabled (Listen button)
   - Verify audio output settings
   - Check if voice chat is connected

3. Connection issues
   - Verify LiveKit server URL (use `wss://` for cloud)
   - Check network connectivity
   - Ensure API keys are correct and match your LiveKit project

### Debug Mode

Enable debug logging:

```typescript
// In browser console
localStorage.setItem('livekit-debug', 'true');
```

## Future Enhancements

- Spatial audio (3D positional voice)
- Voice channel system (different rooms)
- Push-to-talk keybind support
- Voice recording and playback
- Voice effects and filters
- Integration with game proximity chat
