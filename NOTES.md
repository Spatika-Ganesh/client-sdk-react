# VAPI Widget Implementation

## Overview

This document contains internal technical implementation details for the VAPI Widget. For user documentation and API reference, see README.md.

## Key Technical Features

> **Note**: For user-facing features list, see README.md.

This document focuses on the technical implementation details of the widget's features.

## Component Architecture

### Core Component: VapiWidget

Located at `src/components/VapiWidget.tsx`, this component handles:

1. **Multi-mode Support**: Voice-only, chat-only, or hybrid interactions
2. **State Management**: Call state, conversation history, UI state
3. **Event Handling**: VAPI events and user interactions
4. **UI Rendering**: Floating button and expanded interface

### Custom Hooks

- **useVapiWidget**: Main hook that combines voice and chat functionality
- **useVapiCall**: Handles voice call integration with VAPI
- **useVapiChat**: Manages chat functionality with VAPI or custom API

## Implementation Details

> **Note**: Hybrid mode is not fully supported yet. When switching between modes, conversation history is not maintained.

### Props Interface

```typescript
interface VapiWidgetProps {
  // Required
  publicKey: string; // VAPI public key

  // VAPI Configuration (provide at least one)
  assistantId?: string; // VAPI assistant ID (supported by both voice and chat)
  assistant?: any; // Full assistant object (voice only)
  assistantOverrides?: any; // Assistant overrides (supported by both voice and chat)

  // API Configuration
  apiUrl?: string; // Optional custom API URL for chat mode

  // Layout & Position
  position?: 'bottom-right' | 'bottom-left' | 'top-right' | 'top-left';
  size?: 'tiny' | 'compact' | 'full';
  radius?: 'none' | 'small' | 'medium' | 'large';

  // Mode & Theme
  mode?: 'voice' | 'chat' | 'hybrid'; // Default: 'voice'
  theme?: 'light' | 'dark'; // Default: 'light'

  // Colors
  baseColor?: string; // Background color
  accentColor?: string; // Primary accent color
  buttonBaseColor?: string; // Floating button background
  buttonAccentColor?: string; // Floating button text/icon color

  // Text & Labels
  mainLabel?: string; // Widget header text (default: 'Talk with AI')
  startButtonText?: string; // Voice start button (default: 'Start')
  endButtonText?: string; // Voice end button (default: 'End Call')

  // Empty State Messages
  emptyVoiceMessage?: string;
  emptyVoiceActiveMessage?: string;
  emptyChatMessage?: string;
  emptyHybridMessage?: string;

  // Legal & Consent
  requireConsent?: boolean; // Show consent form on first use
  termsContent?: string; // Custom consent text
  localStorageKey?: string; // Key for storing consent

  // Display Options
  showTranscript?: boolean; // Show/hide transcript in voice mode

  // Event Handlers
  onCallStart?: () => void;
  onCallEnd?: () => void;
  onMessage?: (message: any) => void;
  onError?: (error: Error) => void;
}
```

### State Management

The widget manages several state variables:

- `isExpanded`: Controls widget expansion
- `hasConsent`: Tracks user consent status
- `chatInput`: Current chat input value
- Voice state via `useVapiCall` hook
- Chat state via `useVapiChat` hook
- Combined conversation history

### VAPI Integration

The widget supports three types of VAPI configuration:

```typescript
// Simple assistant ID (supports both voice and chat)
assistantId: "assistant-id"

// Assistant with overrides (supports both voice and chat)
assistantId: "assistant-id"
assistantOverrides: {
  variableValues: { name: 'John' },
}

// Full assistant object (voice only)
assistant: {
  model: {
    provider: 'openai',
    model: 'gpt-4o-mini',
    messages: [
      { role: 'system', content: 'You are a helpful assistant.' }
    ]
  },
  voice: {
    provider: '11labs',
    voiceId: 'burt'
  }
  // ... full assistant configuration
}
```

## Usage Examples

### 1. React Component Usage

#### Voice Mode (Default)

```tsx
import { VapiWidget } from 'vapi-client-widget-web';

function App() {
  return (
    <VapiWidget
      publicKey="your-vapi-public-key"
      assistantId="your-assistant-id"
      position="bottom-right"
      theme="light"
      accentColor="#3B82F6"
      size="compact"
      onCallStart={() => console.log('Call started')}
      onCallEnd={() => console.log('Call ended')}
    />
  );
}
```

#### Chat Mode

```tsx
<VapiWidget
  publicKey="your-vapi-public-key"
  assistantId="your-assistant-id"
  mode="chat"
  theme="dark"
  size="full"
  onMessage={(msg) => console.log('Message:', msg)}
/>
```

#### Hybrid Mode with Assistant Overrides

```tsx
<VapiWidget
  publicKey="your-vapi-public-key"
  assistantId="your-assistant-id"
  assistantOverrides={{
    variableValues: { name: 'John' },
  }}
  mode="hybrid"
  showTranscript={true}
  requireConsent={true}
  termsContent="Custom terms and conditions..."
/>
```

#### Voice Mode with Full Assistant Object

```tsx
<VapiWidget
  publicKey="your-vapi-public-key"
  assistant={{
    model: {
      provider: 'openai',
      model: 'gpt-4o-mini',
      messages: [{ role: 'system', content: 'You are a helpful assistant.' }],
    },
    voice: {
      provider: '11labs',
      voiceId: 'burt',
    },
  }}
  mode="voice"
  size="full"
/>
```

### 2. HTML Embedding (Data Attributes)

```html
<!DOCTYPE html>
<html>
  <head>
    <script src="./dist/widget.umd.js"></script>
  </head>
  <body>
    <div
      data-client-widget="VapiWidget"
      data-props='{
      "publicKey": "your-vapi-public-key",
      "assistantId": "your-assistant-id",
      "mode": "hybrid",
      "position": "bottom-right",
      "theme": "dark",
      "accentColor": "#10b981",
      "size": "compact"
    }'
    ></div>
  </body>
</html>
```

### 3. Programmatic Creation

```javascript
const widget = new WidgetLoader({
  container: '#widget-container',
  component: 'VapiWidget',
  props: {
    publicKey: 'your-vapi-public-key',
    assistantId: 'your-assistant-id',
    assistantOverrides: {
      model: 'gpt-4',
    },
    mode: 'chat',
    theme: 'light',
    baseColor: '#f3f4f6',
    accentColor: '#8b5cf6',
    size: 'full',
  },
});
```

## UI States and Behavior

### Floating Button States

- **Tiny Mode (Voice Only)**:
  - Small circular button (48×48px)
  - Expands to 80×80px with red glow when active
  - Direct voice toggle without expanded view
- **Compact Mode**:
  - Pill-shaped button with icon and label
  - Shows connection status via animated icon
- **Full Mode**:
  - Larger pill button with prominent label
  - Enhanced hover effects

### Expanded Widget Interface

- **Header**:
  - Assistant name/label
  - Dynamic status message
  - Reset button (clears conversation)
  - Close button
- **Conversation Area**:
  - Message bubbles with role distinction
  - Markdown rendering support
  - Auto-scrolling to latest message
  - Typing indicators
- **Controls Section**:
  - Voice mode: Start/End call button
  - Chat mode: Text input with send button
  - Hybrid mode: Both input field and voice button

### Mode-Specific Behaviors

#### Voice Mode

- Click microphone to start/end calls
- Optional transcript display
- Volume level visualization
- Voice activity indicators

#### Chat Mode

- Text input always visible
- Enter key to send messages
- Markdown formatting support
- Typing indicators

#### Hybrid Mode

- Seamless switching between voice and chat
- Conversation cleared when switching modes
- Input disabled during voice calls
- Smart mode detection

## Advanced Features

### Consent Management

Enable user consent flow:

```tsx
<VapiWidget
  requireConsent={true}
  termsContent="By using this service, you agree to..."
  localStorageKey="my_app_vapi_consent"
/>
```

### Custom Empty States

Provide helpful context-specific messages:

```tsx
<VapiWidget
  emptyVoiceMessage="Click the mic to start your conversation"
  emptyVoiceActiveMessage="I'm listening..."
  emptyChatMessage="Type your question below"
  emptyHybridMessage="Choose voice or text to begin"
/>
```

### Transcript Control

Hide transcript for voice-only experiences:

```tsx
<VapiWidget
  mode="voice"
  showTranscript={false} // Shows only volume indicator
/>
```

## Integration Requirements

### VAPI Setup

1. Create VAPI account at [vapi.ai](https://vapi.ai)
2. Create an assistant or use inline configuration
3. Get your public API key
4. Configure assistant settings

### Dependencies

- `@vapi-ai/web`: Core VAPI SDK
- `react`: React framework
- `react-dom`: React DOM renderer
- `react-markdown`: Markdown rendering
- `@phosphor-icons/react`: Icon library

### Browser Requirements

- Modern browser with WebRTC support (for voice)
- Microphone permissions (for voice)
- Secure context (HTTPS) for production

## Error Handling

The widget handles common error scenarios:

### Permission Errors

- Microphone access denied
- Clear error messaging
- Fallback to chat mode (in hybrid)

### Connection Errors

- Network connectivity issues
- VAPI service unavailability
- Automatic retry mechanisms

### Configuration Errors

- Invalid API keys
- Missing configuration
- Helpful error messages

## Performance Considerations

### Optimizations

- Lazy loading of VAPI SDK
- Efficient re-rendering with React hooks
- Debounced typing indicators
- Smooth animations with CSS

### Memory Management

- Proper cleanup of event listeners
- Connection termination on unmount
- Conversation history limits

## Testing and Development

### Development Setup

```bash
# Install dependencies
npm install

# Build all packages
npm run build:all

# Run example app
cd example && npm run dev
```

### Testing Modes

Test different configurations:

```tsx
// Test voice mode
<VapiWidget publicKey="..." assistantId="..." mode="voice" />

// Test chat mode
<VapiWidget publicKey="..." assistantId="..." mode="chat" />

// Test hybrid mode
<VapiWidget publicKey="..." assistantId="..." mode="hybrid" />
```

## Troubleshooting

### Common Issues

#### Widget Not Appearing

- Verify script inclusion
- Check console for errors
- Ensure container element exists

#### Voice Not Working

- Check microphone permissions
- Verify VAPI credentials
- Ensure HTTPS context
- Try chat mode as fallback

#### Chat Not Working

- Verify VAPI configuration
- Check network connectivity
- Inspect console for API errors

#### Styling Issues

- Ensure Tailwind CSS is loaded
- Check for CSS conflicts
- Verify theme settings

### Debug Mode

Enable detailed logging:

```tsx
// Check console for detailed logs
window.DEBUG_VAPI = true;
```
