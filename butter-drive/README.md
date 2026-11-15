# butter-drive

The server that runs on Butter (Jetson Orin Nano). It moves the wheels, streams the camera and speaks through the USB speaker. Everything arrives as small JSON messages over one WebSocket.

## Files

| File | What it does |
|---|---|
| `server.py` | Motor control, safety watchdog, speech. Port 8770 |
| `camera_server.py` | Camera stream (MJPEG). Port 5600 |
| `jetson_check.py` | Checks pins, camera and speaker; can spin one wheel at a time |
| `run.sh` | Start, stop or check both servers by hand |
| `user-units/` | systemd user services that start both servers at boot |
| `test_server.py` | Tests for the protocol and the watchdog (no robot needed) |

## Messages

```
-> {"cmd":"hello"}                            <- {"type":"hello","robot":"butter","mock":false}
-> {"cmd":"drive","dir":"forward","ttl":450}  <- {"type":"ack","dir":"forward"}
-> {"cmd":"speed","value":0.1}                <- {"type":"speed","value":0.1}
-> {"cmd":"say","text":"Hello"}               speaks through Butter's speaker
-> {"cmd":"status"}                           <- {"type":"status","dir":"stop","speed":0.1,"clients":1}
```

`dir` is one of `forward`, `backward`, `left`, `right`, `strafe_left`, `strafe_right`, `stop`. Left and right rotate in place.

## Safety

- The wheels stop if no drive command arrives within its `ttl` (450 ms by default, never more than 1 s).
- The wheels stop when the controlling client disconnects, and when the server exits.
- One controller at a time: the newest client to send `drive` owns the motors.
- The L298N enable pins are low whenever Butter is stopped.
- Speed is a duty cycle from 0.05 to 1.0 and starts at 0.05.

## Pin map

40-pin header, BOARD numbering. Forward is IN2 high and IN1 low.

| Wheel | IN1 | IN2 |
|---|---|---|
| Left front | 11 | 13 |
| Left rear | 16 | 15 |
| Right front | 33 | 31 |
| Right rear | 29 | 18 |

| Driver | ENA | ENB |
|---|---|---|
| Left | 12 | 35 |
| Right | 7 | 32 |

The left rear motor is wired reversed on the chassis, so its two pins are swapped in software. Speed is set by pulsing the enable pins (software PWM, 50 Hz). The full schematic is in [`../docs/butter-wiring.png`](../docs/butter-wiring.png).

## Run it

Try it on any computer, with no robot:

```bash
pip install -r requirements.txt
python3 server.py --mock
```

On the Jetson:

```bash
./run.sh            # start both servers
./run.sh status
./run.sh stop
python3 jetson_check.py --wheel LF   # spin one wheel forward and back
```

Start at boot, without sudo (needs lingering enabled for the user):

```bash
mkdir -p ~/.config/systemd/user
cp user-units/*.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now butter-drive butter-camera
```

The paths in `run.sh` and the service files point at the Python environment on our robot. Change them to match yours.

## Speech

`say` uses [Piper](https://github.com/OHF-Voice/piper1-gpl). The voice model is loaded once at startup and kept in memory, which brings the wait before the first sound from about 2 seconds down to a fraction of a second.

## Tests

```bash
python3 -m pytest -q test_server.py
```
