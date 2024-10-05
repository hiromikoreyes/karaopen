<script lang="js">
  import { onMount } from 'svelte';
  import { Midi } from '@tonejs/midi';
  import PitchDetector from './lib/PitchDetector.svelte';

  let canvas; // Reference to the canvas element
  const SingingMidi = []; // Array to store MIDI notes
  let offsetX = 0; // Initialize horizontal offset for scrolling
  let lastTimestamp = 0; // Store the last timestamp for animation

  async function initVoiceMidiTrack() {
    const midi = await Midi.fromUrl("./songs/mg.mid"); // Make sure the path is correct

    midi.tracks.forEach(track => {
      const notes = track.notes;
      notes.forEach(note => {
        SingingMidi.push(note);
      });
    });

    console.log(SingingMidi); // Log all the notes
  }

  function maxMidi(MidiArray) {
    let max = 0;
    MidiArray.forEach(note => {
      if (note.midi > max) {
        max = note.midi;
      }
    });
    return max;
  }

  function minMidi(MidiArray) {
    let min = 1000;
    MidiArray.forEach(note => {
      if (note.midi < min) {
        min = note.midi;
      }
    });
    return min;
  }

  function drawPianoRoll(keyHeight, keyWidth, min, max) {
    const ctx = canvas.getContext('2d');
    ctx.clearRect(0, 0, canvas.width, canvas.height); // Clear previous drawings

    const numKeys = max - min + 1;

    // Draw piano keys
    for (let i = 0; i < numKeys; i++) {
      ctx.fillStyle = '#fff';
      ctx.fillRect(0, i * keyHeight, keyWidth, keyHeight);
      ctx.strokeRect(0, i * keyHeight, keyWidth, keyHeight); // Outline the keys
    }

    // Draw notes as rectangles on the piano keys
    SingingMidi.forEach(note => {
      // Calculate the position of each note rectangle
      const x = (note.time * 100) - offsetX; // Scale time to x-axis and subtract offset for scrolling
      const y = (max - note.midi) * keyHeight; // Scale MIDI number to y-axis
      const width = note.durationTicks/10; // Scale duration to width
      const height = 15; // Small margin for separation
      
      if (x + width > 0 && x < canvas.width) {
        ctx.fillRect(x, y, width, height);
      }
    });
  }

  function update(time) {
    // Update the horizontal offset based on time
    offsetX += 0.5; // Adjust the scroll speed (higher value = faster scroll)
    
    const keyHeight = 15; // Height of each piano key rectangle
    const min = minMidi(SingingMidi);
    const max = maxMidi(SingingMidi);
    drawPianoRoll(keyHeight, 15, min, max);

    lastTimestamp = time; // Update the last timestamp
    requestAnimationFrame(update); // Call the update function again
  }

  onMount(async () => {
    await initVoiceMidiTrack();
    requestAnimationFrame(update); // Start the animation loop
  });
  
</script>

<PitchDetector/>

<canvas bind:this={canvas} width="1000" height="3000"></canvas>
