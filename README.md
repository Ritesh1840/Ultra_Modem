# ▚ UltraModem

**Acoustic FSK data link — browser to browser, over the air, no cables.**

![UltraModem Receiver UI in action]

UltraModem is a pure JavaScript and Web Audio API implementation of a data-over-audio communication system. It allows two devices running a web browser to transmit and receive short text messages through the air using high-frequency sound, effectively turning their speakers and microphones into a modem.

This project runs entirely in the browser, with no server-side components, frameworks, or build steps required.

## How to Use

1.  **Open the application** on two separate devices (e.g., a laptop and a phone). You can start at the main `index.html` page.
2.  **Navigate to Sender Mode:** On one device, choose **▲ SENDER**.
3.  **Navigate to Receiver Mode:** On the other device, choose **◉ RECEIVER** and click the **◉ LISTEN** button to activate the microphone.
4.  **Position the devices:** Place the devices a few feet apart in a quiet room, with the speaker of the sender facing the microphone of the receiver.
5.  **Transmit Data:** On the sender device, type a message (up to 64 characters) and click **▲ SEND**.
6.  **Receive Data:** Watch the receiver's UI as it locks onto the signal, decodes the bits, verifies the checksum, and displays the final message.

## How It Works

UltraModem uses **Frequency-Shift Keying (FSK)** to encode binary data. A `0` is represented by a tone at **18500 Hz** and a `1` is represented by a tone at **19500 Hz**. These near-ultrasonic frequencies are at the edge of human hearing, making the transmission relatively unobtrusive.

### Key Technical Features

*   **Sample-Accurate Timing:** Instead of relying on JavaScript's `setInterval()`, which is prone to event loop jitter, the receiver uses a `ScriptProcessorNode` (or `AudioWorklet`). This allows it to process audio in exact, sample-accurate chunks, ensuring the demodulator never loses sync with the incoming bitstream.

*   **Efficient Goertzel Algorithm:** To detect which frequency is present in an audio chunk, the demodulator uses the **Goertzel algorithm**. Unlike a full Fast Fourier Transform (FFT) which calculates the magnitude of a wide spectrum of frequencies, Goertzel is highly optimized for detecting the strength of a few specific, pre-determined frequencies. This makes it significantly more efficient and provides better signal-to-noise performance for this use case.

*   **Robust Preamble Synchronization:** A transmission begins with a `101010...` preamble. The receiver continuously scans the incoming audio for this pattern. To achieve a lock, it correlates the recent history of detected bits against the expected preamble pattern at multiple sub-bit phases. Once a strong correlation is found, the receiver is "locked" and knows the exact sample index where each subsequent data bit begins.

### Packet Structure

Each data packet is carefully structured for robust transmission:

| Preamble          | Start Marker | Length (8 bits) | Data (N*8 bits) | Checksum (8 bits) | End Marker        |
| ----------------- | ------------ | --------------- | --------------- | ----------------- | ----------------- |
| `1010101010101010`| `11111111`   | `0...N`         | `...`           | `...`             | `00000000`        |

*   **Preamble (16 bits):** An alternating sequence of 1s and 0s that allows the receiver to detect a signal and synchronize its clock.
*   **Start Marker (8 bits):** A unique pattern that signals the end of the preamble and the start of the payload.
*   **Length (8 bits):** An unsigned integer specifying the number of bytes in the data payload.
*   **Data (Variable):** The raw text message, with each character converted to its 8-bit ASCII representation.
*   **Checksum (8 bits):** A simple XOR checksum calculated from the data payload, used for error detection.
*   **End Marker (8 bits):** Signals the end of the packet.

## Tunable Parameters

The core transmission parameters can be adjusted in `main.js`:

*   `FREQ_0` / `FREQ_1`: The frequencies for bit `0` and bit `1`.
*   `BIT_MS`: The duration of each bit in milliseconds. A shorter duration increases the data rate but may reduce reliability. The default is 50ms (20 bits/sec).
*   `PREAMBLE`: The bit pattern used for synchronization.

## Running Locally

No build process is required. Simply serve the project directory with a local static file server.

While any server will work, we recommend using the Node.js-based `http-server` as it is a standard tool in modern web development and provides more robust features than Python's built-in server.

**Recommended (Node.js):**

```bash
npx http-server
```

**Alternative (Python):**

```bash
python -m http.server
```

Or with Node.js:

```bash
npx http-server
```

Then open `http://localhost:3000` (or the appropriate port) in your browser.
---

Copyright © 2026 Rupam Kayal. All rights reserved.
