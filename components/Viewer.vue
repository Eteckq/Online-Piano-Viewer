<template>
  <div>
    <button>start</button>
    <canvas ref="piano"></canvas>
  </div>
</template>

<script>
import { Midi } from "@tonejs/midi";
import * as Tone from "tone";
const keyWidth = 3.8;
const keyHeight = 12;

const notes = ["C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B"];

import Vue from "vue";

export default Vue.extend({
  data() {
    return {
      canvas: null,
      startTime: Date.now(),
      elapsedTime: 0,
      midi: null,
      synth: null,
    };
  },
  async mounted() {
    document.querySelector("button")?.addEventListener("click", async () => {
      await Tone.start();
      console.log("audio is ready");
      this.synth = new Tone.Sampler({
        urls: instruments["piano"],
        release: 1,
        baseUrl: `/samples/${"piano"}/`,
      }).toDestination();
      this.readFile();
    });

    this.canvas = this.$refs["piano"];
    this.refresh(keyWidth * 52, 100, (ctx, ratio) => {
      this.elapsedTime = -(this.startTime - Date.now()) / 1000;
      const referenceLineWidth = 1;
      ctx.lineWidth = referenceLineWidth / ratio;

      // ------- Background -------

      ctx.fillStyle = "#272a3d";
      ctx.fillRect(0, 0, keyWidth * 52, 100);

      // ------- VIEWER -------

      // Lines
      ctx.strokeStyle = "white";
      for (let index = 0; index <= 88; index++) {
        let key = this.keyAt(index);
        if (key === "C" || key === "F") {
          let infoKey = this.getCoordsFromIndex(index);
          ctx.beginPath();
          ctx.moveTo(infoKey.x, 0);
          ctx.lineTo(infoKey.x, 90);
          ctx.stroke();
        }
      }

      if (this.midi) {
        // Notes
        for (const note of this.midi.tracks[0].notes) {
          if (this.elapsedTime >= note.time - 5) {
            let coords = this.getCoordsFromIndex(note.midi - 12 * 3);
            let y = (this.elapsedTime - note.time) * 45;
            let height = note.duration * 20;
            ctx.fillStyle = "purple";
            ctx.fillRect(coords.x, y, coords.lenght, height);
            ctx.stroke();
          }
        }
      }

      // ------- KEYS -------

      const drawWhiteKey = (infos) => {
        ctx.strokeStyle = "black";
        ctx.fillStyle = "white";
        ctx.fillRect(infos.x, 100 - keyHeight, infos.lenght, keyHeight);
        ctx.stroke();
      };

      function drawBlackKey(infos) {
        ctx.strokeStyle = "white";
        ctx.fillStyle = "black";
        ctx.fillRect(infos.x, 100 - keyHeight, infos.lenght, keyHeight / 1.6);
        ctx.stroke();
      }

      for (let index = 0; index <= 88; index++) {
        if (!this.isSharp(index)) {
          drawWhiteKey(this.getCoordsFromIndex(index));
        }
      }

      for (let index = 0; index <= 88; index++) {
        if (this.isSharp(index)) {
          drawBlackKey(this.getCoordsFromIndex(index));
        }
      }
    });
  },
  methods: {
    async readFile(url = "/midi/Zanarkand.mid") {
      this.midi = await Midi.fromUrl(url);
      this.startTime = Date.now();

      console.log(this.midi.tracks[0].notes);

      this.midi.tracks.forEach((track) => {
        //tracks have notes and controlChanges

        //notes are an array
        const notes = track.notes;
        const now = Tone.now() + 1;
        notes.forEach((note) => {
          //note.midi, note.time, note.duration, note.name
          setTimeout(() => {
            this.synth.triggerAttackRelease(
              getNameMinusOctave(note.name, 1),
              note.duration,
              undefined,
              note.velocity
            );
          }, (note.time + now) * 1000);
        });

        //the control changes are an object
        //the keys are the CC number
        track.controlChanges[64];
        //they are also aliased to the CC number's common name (if it has one)
        track.controlChanges.sustain.forEach((cc) => {
          // cc.ticks, cc.value, cc.time
        });

        //the track also has a channel and instrument
        //track.instrument.name
      });
    },
    refresh(referenceWidth, referenceHeight, drawFunction) {
      this.canvas.width = this.canvas.clientWidth;
      this.canvas.height = this.canvas.clientHeight;

      const ratio = Math.min(
        this.canvas.width / referenceWidth,
        this.canvas.height / referenceHeight
      );
      const ctx = this.canvas.getContext("2d");
      ctx.scale(ratio, ratio);

      drawFunction(ctx, ratio);
      window.requestAnimationFrame(() => {
        this.refresh(referenceWidth, referenceHeight, drawFunction);
      });
    },
    isSharp(number) {
      let name = notes[Math.abs(number) % 12];
      return name[1] && name[1] === "#";
    },
    keyAt(number) {
      return notes[Math.abs(number) % 12];
    },
    getCoordsFromIndex(index) {
      let gapX = 0;
      for (let i = 0; i <= 88; i++) {
        if (this.isSharp(i)) {
          gapX += 1;
        }
        if (index === i) {
          if (this.isSharp(i)) {
            return {
              x: (i - gapX) * keyWidth + keyWidth / 1.4,
              lenght: keyWidth / 1.9,
            };
          } else {
            return { x: (i - gapX) * keyWidth, lenght: keyWidth };
          }
        }
      }
      return { x: 0, lenght: 0.1 };
    },
  },
});
const instruments = {
  piano: {
    A0: "A0.mp3",
    A1: "A1.mp3",
    A2: "A2.mp3",
    A3: "A3.mp3",
    A4: "A4.mp3",
    A5: "A5.mp3",
    A6: "A6.mp3",
    "A#0": "As0.mp3",
    "A#1": "As1.mp3",
    "A#2": "As2.mp3",
    "A#3": "As3.mp3",
    "A#4": "As4.mp3",
    "A#5": "As5.mp3",
    "A#6": "As6.mp3",
    B0: "B0.mp3",
    B1: "B1.mp3",
    B2: "B2.mp3",
    B3: "B3.mp3",
    B4: "B4.mp3",
    B5: "B5.mp3",
    B6: "B6.mp3",
    C0: "C0.mp3",
    C1: "C1.mp3",
    C2: "C2.mp3",
    C3: "C3.mp3",
    C4: "C4.mp3",
    C5: "C5.mp3",
    C6: "C6.mp3",
    C7: "C7.mp3",
    "C#0": "Cs0.mp3",
    "C#1": "Cs1.mp3",
    "C#2": "Cs2.mp3",
    "C#3": "Cs3.mp3",
    "C#4": "Cs4.mp3",
    "C#5": "Cs5.mp3",
    "C#6": "Cs6.mp3",
    D0: "D0.mp3",
    D1: "D1.mp3",
    D2: "D2.mp3",
    D3: "D3.mp3",
    D4: "D4.mp3",
    D5: "D5.mp3",
    D6: "D6.mp3",
    "D#0": "Ds0.mp3",
    "D#1": "Ds1.mp3",
    "D#2": "Ds2.mp3",
    "D#3": "Ds3.mp3",
    "D#4": "Ds4.mp3",
    "D#5": "Ds5.mp3",
    "D#6": "Ds6.mp3",
    E0: "E0.mp3",
    E1: "E1.mp3",
    E2: "E2.mp3",
    E3: "E3.mp3",
    E4: "E4.mp3",
    E5: "E5.mp3",
    E6: "E6.mp3",
    F0: "F0.mp3",
    F1: "F1.mp3",
    F2: "F2.mp3",
    F3: "F3.mp3",
    F4: "F4.mp3",
    F5: "F5.mp3",
    F6: "F6.mp3",
    "F#0": "Fs0.mp3",
    "F#1": "Fs1.mp3",
    "F#2": "Fs2.mp3",
    "F#3": "Fs3.mp3",
    "F#4": "Fs4.mp3",
    "F#5": "Fs5.mp3",
    "F#6": "Fs6.mp3",
    G0: "G0.mp3",
    G1: "G1.mp3",
    G2: "G2.mp3",
    G3: "G3.mp3",
    G4: "G4.mp3",
    G5: "G5.mp3",
    G6: "G6.mp3",
    "G#0": "Gs0.mp3",
    "G#1": "Gs1.mp3",
    "G#2": "Gs2.mp3",
    "G#3": "Gs3.mp3",
    "G#4": "Gs4.mp3",
    "G#5": "Gs5.mp3",
    "G#6": "Gs6.mp3",
  },
};

function getNameMinusOctave(name, octave) {
  let note = name[0];
  let o = name[1] - octave;
  if (name.length === 3) {
    note += name[1];
    o = name[2] - octave;
  }
  return note + o;
}
</script>

<style scoped>
div {
  margin: auto;
  width: 100vw;
  height: 100vh;
}

canvas {
  width: 100%;
  height: 100%;
  object-fit: contain;
}
</style>