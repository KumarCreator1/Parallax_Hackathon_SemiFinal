# Integrity - Ethical Exam Proctoring

> **🎉 Live Demo Available!**
> The application is securely hosted and fully accessible online. You can skip the local setup and test the live version directly at:
> **👉 [https://examproctorclient.onrender.com](https://examproctorclient.onrender.com)**

## Video Submission
> **📺 Watch our video demo:** [Insert Video Link Here]

---

## 1. Problem Statement and Objective

**Objective:** Create a privacy-first, anti-cheat proctoring system using a dual-device architecture.

Traditional online proctoring suffers from critical vulnerabilities:
- **Limited FOV**: Laptop webcams only see the face; blind to what's happening on the screen or under the desk.
- **Hardware Hacks**: HDMI splitters and VM usage bypass software-level detection.
- **Privacy Concerns**: Streaming bedroom video to servers raises GDPR/privacy issues.
- **Bandwidth Heavy**: High-quality video streaming crashes on poor connections.
- **False Positives**: Minor movements trigger flags, overwhelming proctors.

---

## 2. Gap Analysis (Current vs Proposed Solution)

**Why Current Solutions Fall Short:**
The market lacks comprehensive solutions that address both physical and digital attack vectors simultaneously. Each existing approach creates critical gaps that sophisticated cheaters exploit.

- **Front-Facing Webcams**
  - *Critical Gap*: No 360° visibility allows hidden notes and unauthorized devices.
  - *Issue*: Laptop cameras only capture the student's face, remaining entirely blind to what is happening on the screen, the keyboard, or under the desk.
- **Cloud Video Streaming**
  - *Critical Gap*: Heavy bandwidth dependency crashes on poor connections; GDPR privacy violations.
  - *Issue*: Requires heavy bandwidth which crashes on poor internet connections; streaming private bedroom footage to remote servers raises severe GDPR and privacy concerns.
- **Software Anti-Cheat**
  - *Critical Gap*: No hardware fingerprinting enables VM and HDMI splitter evasion.
  - *Issue*: Traditional software detection is easily bypassed by students utilizing Virtual Machines (VMs) or hardware like HDMI splitters.
- **Automated Motion Flagging**
  - *Critical Gap*: No behavioral aggregation floods administrators with false positives.
  - *Issue*: Systems trigger alerts for minor, innocent movements, resulting in a flood of false positives that overwhelm administrators.

**Proposed Solution - "Integrity":**
To solve these issues, Integrity uses a **Local-First, Dual-Device** approach. It leverages the student's smartphone as a secondary "Sentinel" to monitor the physical environment, processing all video data **on-device**. Our philosophy is **"Status Codes, Not Video Streams"**—ensuring zero video storage and minimal bandwidth usage.

---

## 3. System Architecture & Workflow

The system is split into three main environments coordinating asynchronously:

1. **Student Local Environment**
   - **Exam Terminal (Laptop)**: Displays the exam and locks the browser. Monitors the student's gaze using TensorFlow.js (BlazeFace). Features a Flash Controller to emit challenges (e.g. ambient light changes) and a QR Generator for pairing.
   - **Sentinel Phone**: Positioned to monitor the physical desk space. Runs local AI (TensorFlow.js COCO-SSD) to detect unauthorized objects. Uses a Lux Detector to verify flash challenges.
2. **Network Layer**
   - **Signaling Server**: A Node.js socket server that handles WebRTC signaling (Coordinate P2P) between the laptop and the phone. It acts as a JSON aggregator to collect heartbeat and alert payloads. 
3. **Proctor Environment**
   - **Proctor Dashboard**: Aggregated JSON objects flow to an administrative dashboard, showing a Status Grid and dynamic Trust Scores based on aggregated flags.

---

## 4. Technology Stack Details

- **Frontend (PWA)**: React + Vite, TailwindCSS, Zustand (State Management)
- **AI & Computer Vision**: TensorFlow.js 
  - *BlazeFace* (MediaPipe) for lightweight gaze tracking.
  - *COCO-SSD* for object detection on Sentinel.
- **Real-Time Communication**: WebRTC (Peer-to-Peer DataChannels), Socket.io (Signaling)
- **Backend / Infrastructure**: Node.js Signaling Server, PostgreSQL (Supabase schema for users/auth context)

---

## 5. Setup and Execution Steps

To test the application on local network devices securely (since Web APIs like camera and microphone access require a secure `HTTPS` context), you can use [ngrok](https://ngrok.com/) to expose both the client and the signaling server over HTTPS.

### 5.1 Prerequisites for Testing Locally
- Setup Supabase and ensure `.env` files are configured for both `client/` and `signaling-server/`.
- **Testing for Hackathon Organizers**: To test the platform remotely, you can log in using our pre-existing testing accounts:
  - **Proctor Account:** `p@gmail.com` | Password: `123456`
  - **Student Account:** `s@gmail.com` | Password: `123456`
- Install [ngrok](https://ngrok.com/download) on your machine and authenticate:
  ```bash
  ngrok config add-authtoken <YOUR_TOKEN>
  ```

### 5.2 Start Local Servers
Open two terminal windows and start your development servers:

**Terminal 1 (Signaling Server)**
```bash
cd signaling-server
npm install
npm run dev
# Normally runs on localhost:3001
```

**Terminal 2 (Client)**
```bash
cd client
npm install
npm run dev
# Normally runs on localhost:5173
```

### 5.3 Expose the Signaling Server
Open a third terminal and create an HTTPS tunnel for the background port (3001):
```bash
ngrok http 3001
```

**Update Client Configuration:**
In `client/.env`, update the signaling server URL to use the secure ngrok URL instead of localhost:
```env
VITE_SIGNALING_SERVER_URL=https://<random-id>.ngrok-free.app
```
*(Restart your client development server in Terminal 2 so the new environment variable is loaded.)*

### 5.4 Expose the Client App
Open a fourth terminal and expose the client dev server port (typically 5173):
```bash
ngrok http 5173
```
You will get another secure HTTPS URL for your frontend (e.g., `https://<another-id>.ngrok-free.app`). Open this frontend HTTPS URL on your phone and laptop browser.

### 5.5 Folder Structure
```text
examProctorSoftware/
├── client/                 # Frontend React application (Vite)
├── signaling-server/       # WebRTC signaling backend (WebHook) (Node.js)
└── supabase_schema.sql     # Database schema for Supabase
```

---

## 6. Future Scope and Limitations

**Future Scope:**
- **Offline Mode**: Store exam answers locally with cryptographic signatures if the connection drops completely, syncing when reconnected.
- **LMS Integration**: Provide LTI 1.3 support for seamless integration with platforms like Canvas, Blackboard, or Moodle.

**Limitations:**
- Requires students to possess a secondary smartphone capable of running mobile web browsers.
- Implies an inherent setup overhead for students (i.e., properly positioning the phone to view the workspace).
