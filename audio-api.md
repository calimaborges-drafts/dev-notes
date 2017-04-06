# audio-api

## basics

```javascript
var audio_context = window.AudioContext || window.webkitAudioContext;

var con = new audio_context();
```

## oscilator

```javascript
var osc = con.createOscillator();
osc.frequency.value = 600;
osc.type = 'sawtooth';
osc.connect(con.destination);
```

## gain

```javascript
var osc = con.createOscillator();
var amp = con.createGain();

osc.connect(amp);

var now = con.currentTime;
amp.gain.value = 0;
amp.gain.linearRampToValueAtTime(0.1, now + 2);
amp.gain.linearRampToValueAtTime(0  , now + 4);

osc.stop(now + 4.1);
```

## filter

```javascript
var filter = con.createBiquadFilter();
filter.frequency.value = 300;
filter.Q.value = 5 / 10;
var osc = con.createOscillator();
osc.connect(filter);
filter.connect(con.destination);
```
