# VoiceForge
A real-time voice changer for gamers and streamers

import pyaudio
import numpy as np
from scipy.signal import butter, lfilter

class VoiceForge:
    def __init__(self, rate=44100, chunk=1024):
        self.rate = rate  # Sample rate (Hz)
        self.chunk = chunk  # Number of frames per buffer
        self.audio = pyaudio.PyAudio()
        self.stream = None

    def start_stream(self):
        """Start the audio stream."""
        self.stream = self.audio.open(
            format=pyaudio.paInt16,
            channels=1,
            rate=self.rate,
            input=True,
            output=True,
            frames_per_buffer=self.chunk,
            stream_callback=self.process_audio
        )
        self.stream.start_stream()

    def stop_stream(self):
        """Stop the audio stream."""
        if self.stream is not None:
            self.stream.stop_stream()
            self.stream.close()
        self.audio.terminate()

    def process_audio(self, in_data, frame_count, time_info, status):
        """Process the audio data and apply effects."""
        audio_data = np.frombuffer(in_data, dtype=np.int16)
        processed_data = self.apply_effect(audio_data)
        return (processed_data.tobytes(), pyaudio.paContinue)

    def apply_effect(self, audio_data):
        """Apply voice-changing effects."""
        # Example effect: pitch shift
        pitch_factor = 1.5  # Change this value to adjust the pitch
        resampled = self.pitch_shift(audio_data, pitch_factor)
        return resampled

    def pitch_shift(self, audio_data, factor):
        """Simple pitch shifting by resampling."""
        indices = np.round(np.arange(0, len(audio_data), factor)).astype(int)
        indices = indices[indices < len(audio_data)]
        return audio_data[indices]

    def add_reverb(self, audio_data):
        """Add a simple reverb effect."""
        decay = 0.5  # Adjust for reverb intensity
        delayed = np.zeros_like(audio_data)
        delay_samples = int(0.1 * self.rate)  # 100ms delay

        if len(audio_data) > delay_samples:
            delayed[delay_samples:] = audio_data[:-delay_samples] * decay

        return audio_data + delayed

if __name__ == "__main__":
    vf = VoiceForge()
    print("Starting VoiceForge. Speak into the microphone to hear your modified voice.")
    try:
        vf.start_stream()
        while True:
            pass
    except KeyboardInterrupt:
        print("Stopping VoiceForge.")
        vf.stop_stream()
