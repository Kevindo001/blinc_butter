# Butter Says

One page. Type a line, press Enter, and Butter says it through its own speaker.

Open `index.html` in a browser. It connects to Butter's server on port 8770, trying the USB-C cable address first and then the network address in the file. To point it somewhere else, add `?host=` to the address bar:

```
index.html?host=10.0.0.5
```

It only sends speech. It cannot move the wheels, so it is safe to use while someone else is driving.
