# Threading
- Audio callback: real-time, lock-free, allocation-free.
- AI inference: normal-priority worker threads.
- Hand-off: SPSC ring buffers (`flux-host/rt/spsc`). On underrun output silence or dry passthrough — never wait (avoids priority inversion).
