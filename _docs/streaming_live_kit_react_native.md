Streaming API Integration: using LiveKit

Boost real-time video streaming with HeyGen and LiveKit integration
Suggest Edits

This guide demonstrates how to use the HeyGen streaming API endpoints and the LiveKit client SDK for real-time video streaming, which provides a simpler development interface.

    This native API integration enables real-time video generation using HeyGen avatars over a streaming connection.
    It uses LiveKit SDK for establishing a WebRTC-based stream, allowing continuous, low-latency video output as text is streamed in.
    Ideal for real-time or dynamic avatar communication experiences where latency matters, such as virtual meetings or live assistants.

    For Node.js environments, we strongly recommend using the Streaming Avatar SDK package, as it offers a more robust solution. This guide focuses on the raw LiveKit implementation, intended for basic use cases as well as for developers who require more customization options or wish to integrate with existing LiveKit infrastructure.

Implementation Guide
Overview

In this guide:

    We will use the LiveKit CDN client for easy setup without requiring npm packages.
    Simplified WebSocket handling.
    Support for both Talk (LLM) and Repeat modes.
    Real-time event monitoring for both WebSocket and LiveKit events.

Prerequisites

    API Token from HeyGen
    Basic understanding of JavaScript and LiveKit

Step 1: Basic HTML Setup

Create an HTML file with the necessary elements and include the LiveKit JS Client SDK minified CDN version :

<!DOCTYPE html>
<html lang="en">
  <head>
    <script src="https://cdn.jsdelivr.net/npm/livekit-client/dist/livekit-client.umd.min.js"></script>
  </head>

  <body>
    <div>
      <div>
        <div>
          <button id="startBtn">Start</button>
          <button id="closeBtn">Close</button>
        </div>
      </div>
      <div>
        <input id="taskInput" type="text" placeholder="Enter text" />
        <button id="talkBtn">Talk</button>
      </div>
    </div>
    <video id="mediaElement" autoplay></video>

    <script>
      // JavaScript code goes here
    </script>
  </body>
</html>

Step 2. Configuration

const API_CONFIG = {
  serverUrl: "https://api.heygen.com",
  token: "YOUR_API_TOKEN"
};

// Global state
let sessionInfo = null;
let room = null;
let mediaStream = null;

// DOM elements
const mediaElement = document.getElementById("mediaElement");
const taskInput = document.getElementById("taskInput");

Step 3. Core Implementation
3.1 Create and Start Session

async function createSession() {
  // Create new session
  const response = await fetch(`${API_CONFIG.serverUrl}/v1/streaming.new`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${API_CONFIG.token}`
    },
    body: JSON.stringify({
      version: "v2",
      avatar_id: "YOUR_AVATAR_ID"
    })
  });
  
  sessionInfo = await response.json();
  
  // Start streaming
  await fetch(`${API_CONFIG.serverUrl}/v1/streaming.start`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${API_CONFIG.token}`
    },
    body: JSON.stringify({
      session_id: sessionInfo.session_id
    })
  });
  
  // Connect to LiveKit room
  room = new LiveKitClient.Room();
  await room.connect(sessionInfo.url, sessionInfo.access_token);
  
  // Handle media streams
  room.on(LiveKitClient.RoomEvent.TrackSubscribed, (track) => {
    if (track.kind === "video" || track.kind === "audio") {
      mediaStream.addTrack(track.mediaStreamTrack);
      mediaElement.srcObject = mediaStream;
    }
  });
  

}

    The LiveKit CDN version is accessed through the LivekitClient global namespace.
    All LiveKit classes and constants must be prefixed with LivekitClient (e.g., LivekitClient.Room, LivekitClient.RoomEvent

3.2 Send Text to Avatar

async function sendText(text) {
  await fetch(`${API_CONFIG.serverUrl}/v1/streaming.task`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${API_CONFIG.token}`
    },
    body: JSON.stringify({
      session_id: sessionInfo.session_id,
      text: text,
      task_type: "talk"  // or "repeat" to make avatar repeat exactly what you say
    })
  });
}

3.3 Close Session

async function closeSession() {
  await fetch(`${API_CONFIG.serverUrl}/v1/streaming.stop`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${API_CONFIG.token}`
    },
    body: JSON.stringify({
      session_id: sessionInfo.session_id
    })
  });
  
  if (room) {
    room.disconnect();
  }
  
  mediaElement.srcObject = null;
  sessionInfo = null;
  room = null;
  mediaStream = null;
}

Step 4. Event Listeners

// Start session
document.querySelector("#startBtn").addEventListener("click", async () => {
  await createSession();
});

// Close session
document.querySelector("#closeBtn").addEventListener("click", closeSession);

// Send text
document.querySelector("#talkBtn").addEventListener("click", () => {
  const text = taskInput.value.trim();
  if (text) {
    sendText(text);
    taskInput.value = "";
  }
});

Further Features
1. Task Types

The task endpoint supports different task types:

    talk: Avatar processes text through LLM before speaking
    repeat: Avatar repeats the exact input text

2. WebSocket Events

Monitor HeyGen Avatar state through WebSocket events:

const wsUrl = `wss://api.heygen.com/v1/ws/streaming.chat?session_id=${sessionId}&session_token=${token}&silence_response=false`;
const ws = new WebSocket(wsUrl);

ws.addEventListener("message", (event) => {
  const data = JSON.parse(event.data);
  console.log("Event:", data);
});

3. LiveKit Room Events

Monitor room state and media tracks:

room.on(LivekitClient.RoomEvent.DataReceived, (message) => {
  const data = new TextDecoder().decode(message);
  console.log("Room message:", JSON.parse(data));
});

System Flow

    Session setup (steps 1-3)
    Video streaming (step 4)
    Avatar interaction loop (step 5)
    Session closure (step 6)

Complete Demo Code

Here's a full working HTML & JS implementation combining all the components with a basic frontend:

<!DOCTYPE html>
<html lang="en">
  <head>
    <title>HeyGen Streaming API LiveKit (V2)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/livekit-client/dist/livekit-client.umd.min.js"></script>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  </head>

  <body class="bg-gray-100 p-5 font-sans">
    <div class="max-w-3xl mx-auto bg-white p-5 rounded-lg shadow-md">
      <div class="flex flex-wrap gap-2.5 mb-5">
        <input
          id="avatarID"
          type="text"
          placeholder="Avatar ID"
          value="Wayne_20240711"
          class="flex-1 min-w-[200px] p-2 border border-gray-300 rounded-md"
        />
        <input
          id="voiceID"
          type="text"
          placeholder="Voice ID"
          class="flex-1 min-w-[200px] p-2 border border-gray-300 rounded-md"
        />
        <button
          id="startBtn"
          class="px-4 py-2 bg-green-500 text-white rounded-md hover:bg-green-600 transition-colors disabled:opacity-50 disabled:cursor-not-allowed"
        >
          Start
        </button>
        <button
          id="closeBtn"
          class="px-4 py-2 bg-red-500 text-white rounded-md hover:bg-red-600 transition-colors"
        >
          Close
        </button>
      </div>

      <div class="flex flex-wrap gap-2.5 mb-5">
        <input
          id="taskInput"
          type="text"
          placeholder="Enter text for avatar to speak"
          class="flex-1 min-w-[200px] p-2 border border-gray-300 rounded-md"
        />
        <button
          id="talkBtn"
          class="px-4 py-2 bg-green-500 text-white rounded-md hover:bg-green-600 transition-colors"
        >
          Talk (LLM)
        </button>
        <button
          id="repeatBtn"
          class="px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 transition-colors"
        >
          Repeat
        </button>
      </div>

      <video
        id="mediaElement"
        class="w-full max-h-[400px] border rounded-lg my-5"
        autoplay
      ></video>
      <div
        id="status"
        class="p-2.5 bg-gray-50 border border-gray-300 rounded-md h-[100px] overflow-y-auto font-mono text-sm"
      ></div>
    </div>

    <script>
      // Configuration
      const API_CONFIG = {
        apiKey: "apikey",
        serverUrl: "https://api.heygen.com",
      };

      // Global variables
      let sessionInfo = null;
      let room = null;
      let mediaStream = null;
      let webSocket = null;
      let sessionToken = null;

      // DOM Elements
      const statusElement = document.getElementById("status");
      const mediaElement = document.getElementById("mediaElement");
      const avatarID = document.getElementById("avatarID");
      const voiceID = document.getElementById("voiceID");
      const taskInput = document.getElementById("taskInput");

      // Helper function to update status
      function updateStatus(message) {
        const timestamp = new Date().toLocaleTimeString();
        statusElement.innerHTML += `[${timestamp}] ${message}<br>`;
        statusElement.scrollTop = statusElement.scrollHeight;
      }

      // Get session token
      async function getSessionToken() {
        const response = await fetch(
          `${API_CONFIG.serverUrl}/v1/streaming.create_token`,
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
              "X-Api-Key": API_CONFIG.apiKey,
            },
          }
        );

        const data = await response.json();
        sessionToken = data.data.token;
        updateStatus("Session token obtained");
      }

      // Connect WebSocket
      async function connectWebSocket(sessionId) {
        const params = new URLSearchParams({
          session_id: sessionId,
          session_token: sessionToken,
          silence_response: false,
          opening_text: "Hello, how can I help you?",
          stt_language: "en",
        });

        const wsUrl = `wss://${
          new URL(API_CONFIG.serverUrl).hostname
        }/v1/ws/streaming.chat?${params}`;

        webSocket = new WebSocket(wsUrl);

        // Handle WebSocket events
        webSocket.addEventListener("message", (event) => {
          const eventData = JSON.parse(event.data);
          console.log("Raw WebSocket event:", eventData);
        });
      }

      // Create new session
      async function createNewSession() {
        if (!sessionToken) {
          await getSessionToken();
        }

        const response = await fetch(
          `${API_CONFIG.serverUrl}/v1/streaming.new`,
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
              Authorization: `Bearer ${sessionToken}`,
            },
            body: JSON.stringify({
              quality: "high",
              avatar_name: avatarID.value,
              voice: {
                voice_id: voiceID.value,
                rate: 1.0,
              },
              version: "v2",
              video_encoding: "H264",
            }),
          }
        );

        const data = await response.json();
        sessionInfo = data.data;

        // Create LiveKit Room
        room = new LivekitClient.Room({
          adaptiveStream: true,
          dynacast: true,
          videoCaptureDefaults: {
            resolution: LivekitClient.VideoPresets.h720.resolution,
          },
        });

        // Handle room events
        room.on(LivekitClient.RoomEvent.DataReceived, (message) => {
          const data = new TextDecoder().decode(message);
          console.log("Room message:", JSON.parse(data));
        });

        // Handle media streams
        mediaStream = new MediaStream();
        room.on(LivekitClient.RoomEvent.TrackSubscribed, (track) => {
          if (track.kind === "video" || track.kind === "audio") {
            mediaStream.addTrack(track.mediaStreamTrack);
            if (
              mediaStream.getVideoTracks().length > 0 &&
              mediaStream.getAudioTracks().length > 0
            ) {
              mediaElement.srcObject = mediaStream;
              updateStatus("Media stream ready");
            }
          }
        });

        // Handle media stream removal
        room.on(LivekitClient.RoomEvent.TrackUnsubscribed, (track) => {
          const mediaTrack = track.mediaStreamTrack;
          if (mediaTrack) {
            mediaStream.removeTrack(mediaTrack);
          }
        });

        // Handle room connection state changes
        room.on(LivekitClient.RoomEvent.Disconnected, (reason) => {
          updateStatus(`Room disconnected: ${reason}`);
        });

        await room.prepareConnection(sessionInfo.url, sessionInfo.access_token);
        updateStatus("Connection prepared");

        // Connect WebSocket after room preparation
        await connectWebSocket(sessionInfo.session_id);

        updateStatus("Session created successfully");
      }

      // Start streaming session
      async function startStreamingSession() {
        const startResponse = await fetch(
          `${API_CONFIG.serverUrl}/v1/streaming.start`,
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
              Authorization: `Bearer ${sessionToken}`,
            },
            body: JSON.stringify({
              session_id: sessionInfo.session_id,
            }),
          }
        );

        // Connect to LiveKit room
        await room.connect(sessionInfo.url, sessionInfo.access_token);
        updateStatus("Connected to room");

        document.querySelector("#startBtn").disabled = true;
        updateStatus("Streaming started successfully");
      }

      // Send text to avatar
      async function sendText(text, taskType = "talk") {
        if (!sessionInfo) {
          updateStatus("No active session");
          return;
        }

        const response = await fetch(
          `${API_CONFIG.serverUrl}/v1/streaming.task`,
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
              Authorization: `Bearer ${sessionToken}`,
            },
            body: JSON.stringify({
              session_id: sessionInfo.session_id,
              text: text,
              task_type: taskType,
            }),
          }
        );

        updateStatus(`Sent text (${taskType}): ${text}`);
      }

      // Close session
      async function closeSession() {
        if (!sessionInfo) {
          updateStatus("No active session");
          return;
        }

        const response = await fetch(
          `${API_CONFIG.serverUrl}/v1/streaming.stop`,
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
              Authorization: `Bearer ${sessionToken}`,
            },
            body: JSON.stringify({
              session_id: sessionInfo.session_id,
            }),
          }
        );

        // Close WebSocket
        if (webSocket) {
          webSocket.close();
        }
        // Disconnect from LiveKit room
        if (room) {
          room.disconnect();
        }

        mediaElement.srcObject = null;
        sessionInfo = null;
        room = null;
        mediaStream = null;
        sessionToken = null;
        document.querySelector("#startBtn").disabled = false;

        updateStatus("Session closed");
      }

      // Event Listeners
      document
        .querySelector("#startBtn")
        .addEventListener("click", async () => {
          await createNewSession();
          await startStreamingSession();
        });
      document
        .querySelector("#closeBtn")
        .addEventListener("click", closeSession);
      document.querySelector("#talkBtn").addEventListener("click", () => {
        const text = taskInput.value.trim();
        if (text) {
          sendText(text, "talk");
          taskInput.value = "";
        }
      });
      document.querySelector("#repeatBtn").addEventListener("click", () => {
        const text = taskInput.value.trim();
        if (text) {
          sendText(text, "repeat");
          taskInput.value = "";
        }
      });
    </script>
  </body>
</html>

LiveKit Client SDKs

Here’s the list of LiveKit client SDK repositories:

    client-sdk-flutter: Dart, Flutter Client SDK for LiveKit
    client-sdk-js: TypeScript, LiveKit browser client SDK (JavaScript)
    client-sdk-swift: Swift, LiveKit Swift Client SDK for iOS, macOS, tvOS, and visionOS
    client-sdk-android: Kotlin, LiveKit SDK for Android
    client-sdk-unity: C#, Official Unity SDK for LiveKit
    client-sdk-react-native: TypeScript, Official React Native SDK for LiveKit
    client-sdk-react-native-expo-plugin: TypeScript, Expo plugin for the React Native SDK
    client-sdk-unity-web: C#, Official LiveKit SDK for Unity WebGL
    client-sdk-cpp: C++, C++ SDK for LiveKit

You can explore these repos for more detailed information.
Conclusion

The LiveKit-based implementation of HeyGen's Streaming API provides a streamlined approach to integrating interactive avatars into web applications. While this guide covers the basics of browser-side implementation, remember that for production Node.js environments, the @heygen/streaming-avatar npm package offers a more comprehensive solution.
Available Resources

    LiveKit Documentation
    HeyGen Streaming API Reference
    HeyGen Streaming Avatar SDK Reference
