---
layout: post
title: "supercollider session"
author: "@tyleretters"
date: 2021-01-02
---
![supercollider session](/assets/images/supercollider-session.jpg)

the study group convened for the first time in 2021 today. we ended up spending ~4 hours mob-coding a supercollider patch together. use at your own risk!

```supercollider
(55..58)

{ 1.rrand(5); } ! 5

/*
Impulse.ar(freq: 440.0, phase: 0.0, mul: 1.0, add: 0.0)
Impulse.kr(freq: 440.0, phase: 0.0, mul: 1.0, add: 0.0)
Arguments:
freq
Frequency in Hertz.

phase
Phase offset in cycles (0..1).
*/

(
{
  /// TODO: make own delay/verb
  //kick
  var envSpec = Env.perc(0, 0.2);
  var outerTrig = Impulse.kr(1) ! 2;
  var trig = Impulse.ar(TChoose.kr(outerTrig, [2, 3, 4]));
  var ampEnv = EnvGen.ar(envSpec, trig);
  var osc = SinOscFB.ar(ampEnv * 200, ampEnv ** 20 * 5);
  var filter = HPF.ar(osc, 50);

  // hat
  // TODO: mod decay, comb filter
  var trigOuterHat = Impulse.kr(2, 0.5);
  var trigHat = Impulse.kr(TChoose.kr(trigOuterHat, [2, 3, 4, 2.01, 2.5, 8, 12]));
  var envHat = EnvGen.kr(Env.perc(0.01, SinOsc.kr(0.2, 0, 0.04, 0.05)), trigHat);
  var oscHat = SinOscFB.ar(5500, 10);
  var hat = envHat * oscHat;
  var combFreq = TChoose.kr(PulseCount.ar(trig[0]) % 4, [1/55, 1/66, 1/77]);
  var combFilter = CombN.ar(
    hat,
    1,
    combFreq,
    SinOsc.ar(0.73, 0, 0.17, 0.20)
  );

  // snare
  // probability - trig[0] for lead, divide by 4, then sample
  // var attack = Demand.kr(trig[0], Dwhite(0.01, 0.1));
  // var attack = SinOsc.kr(0.32, 0, 0.48, 0.49); // screaming!
  var attack = SinOsc.kr(0.32, 0, 0.048, 0.049);
  // var attack = 0.01;
  var trigSnare = Impulse.kr(1/2, 0.5);
  var envSnare = EnvGen.kr(Env.perc(attack, 0.3), trigSnare);
  var oscSnare = SinOscFB.ar(envSnare * 1000 + 200, 25 * envSnare + 5, envSnare);
  var primes = (2..13).nthPrime;
  var reverb = Mix.ar(primes.collect({|i|
    var delayTime = SinOsc.ar(0.02.rrand(0.1), 0, 0.02.rand, i/100);
    (oscSnare + AllpassL.ar(oscSnare, 1/2, delayTime, 3)) / primes.size
  }));

  // bass
  var trigBass = Impulse.kr(1/2, 0.75);
  var envBass = EnvGen.kr(Env.perc(0.01, 1, 1, 0), trigBass);
  var bass = SinOscFB.ar(1/combFreq, envBass ** 3, envBass ** 0.25);

  // pad
  // TODO: add delay to pad (musical time interval)
  var trigPad = Impulse.kr(1/2, 0);
  var padEnv = EnvGen.ar(Env.perc(1, 3), trigPad);
  var padOsc = SinOscFB.ar(
    SinOsc.kr(
      2.47,
      0,
      4,
      TChoose.kr(trigPad, [ 440, 528, 616 ] ++ [ 660.0, 792.0, 924.0 ])
    ),
    padEnv * 1.2,
    padEnv
  );

  // mixer (audio -> bus)
  var bpfTrig = Impulse.kr(1/12, 0);
  var mix = Mix.ar([
    bass * 0.5,
    oscSnare * 0.4,
    reverb * 0.4,
    hat * 0.3,
    padOsc * 0.1,
    Pan2.ar(combFilter, 1) * 0.2,
    filter * ampEnv * 0.5
  ]);
  var filterFreqEnv = EnvGen.ar(Env.perc(4, 0, 100, 0), bpfTrig);
  var filterAmountEnv = EnvGen.ar(Env.perc(2, 2, 1, 0), bpfTrig);
  var filtered = BPF.ar(mix, filterFreqEnv.midicps, 0.1);

  (filtered * filterAmountEnv) + (mix * (1 - filterAmountEnv))
}.play
)

[1/55, 1/66, 1/77]

(55.cpsmidi - 0.1).midicps

[55, 66, 77] * 8
// [ 220, 264, 308 ]
[ 440, 528, 616 ]
[ 660.0, 792.0, 924.0 ]

(
SynthDef(\sadpad, { |freq=616, t_trig=1, lfoSpeed=2.47, vibratoDepth=4|
  // var padEnv = EnvGen.ar(Env.perc(1, 3), t_trig);
  var padEnv = EnvGen.ar(Env.perc(1, 3), t_trig, doneAction: 2);

  Out.ar(0,
    SinOscFB.ar(
      SinOsc.kr(lfoSpeed, 0, vibratoDepth, freq),
      padEnv * 1.2,
      padEnv
  ) * 0.3 ! 2)
}).add;

Pbind(
  \instrument, \sadpad,
  \trig, 1,
  \freq, Pseq([ 440, 528, 616 ] *.t [1, 1.5]),
  \lfoSpeed, Pwhite(0.0, 3.0),
  \vibratoDepth, 0
).play;
)
[ 440, 528, 616 ] *.t [1, 1.5]


{ Pan2.ar(SinOsc.ar(110), SinOsc.ar(0.5, )) } .play;

3 + 3 * 2
3 * 2 + 3


#(2..3).nthPrime;


(2..13).nthPrime
1.nthPrime

55.reciprocal
1/55

  Limiter
  SoftClip
  softClip

.distort
.softclip.distort

s.sampleRate / s.options.blockSize


(1..10).collect({|i| i * 2})
```