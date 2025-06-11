
### `.bashrc` Limitations:

* Runs **only when a terminal session opens**
* Does **not** run in headless mode (no keyboard/monitor)
* Not suitable for background services or GPIO/camera-based applications

---

## Conclusion: Which One Should You Use?

| Method    | Pros                                      | Cons                                                                                                       |
| --------- | ----------------------------------------- | -----------------------------------------------------------------------------------------------------------|
| `systemd` | Reliable, runs at boot, logs, restartable | Slightly more setup                                                                                        |
| `.bashrc` | Easy, works for quick scripts             | Doesn't work in background/headless or for hardware access and it does not work until you click on terminal|

**Use `systemd` for real projects**, background services, or hardware access (GPIO, camera, etc.).

---
