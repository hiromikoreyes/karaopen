<script lang="js">
  import { onMount } from 'svelte';
  import { Midi } from '@tonejs/midi';

  let canvas; // Reference to the canvas element
  const SingingMidi = []; // Array to store MIDI notes


  async function initVoiceMidiTrack() {
    const midi = await Midi.fromUrl("./songs/melody.mid"); // Make sure the path is correct

    midi.tracks.forEach(track => {
      const notes = track.notes;
      notes.forEach(note => {
        SingingMidi.push(note);
      });
    });

    console.log(SingingMidi); // Log all the notes

    
  }

  function maxMidi(MidiArray){
    let max = 0;

    MidiArray.forEach((note)=>{
      if(note.midi > max){
        max = note.midi;
      }
    })
    return max;
  }

  function minMidi(MidiArray){
    let min = 1000;

    MidiArray.forEach((note)=>{
      if(note.midi < min){
        min = note.midi;
      }
    })
    return min;
  }




  function drawPianoRoll(keyHeight, keyWidth, offsetX, offsetY, min, max) {
    const ctx = canvas.getContext('2d');
    ctx.clearRect(0, 0, canvas.width, canvas.height); // Clear previous drawings

    const numKeys = 36;
    console.log(numKeys)
    // Draw piano keys
    for (let i = 0; i < numKeys; i++) {
      ctx.fillStyle = '#fff';
      ctx.fillRect(offsetX, offsetY + i * keyHeight, keyWidth, keyHeight);
      ctx.strokeRect(offsetX, offsetY + i * keyHeight, keyWidth, keyHeight); // Outline the keys
    }

    // Draw notes as rectangles on the piano keys
    SingingMidi.forEach(note => {
      // Calculate the position of each note rectangle
      const x = note.time * 100; // Scale time to x-axis
      const y = offsetY + (127 - note.midi) * keyHeight; // Scale MIDI number to y-axis
      const width = note.durationTicks; // Scale duration to width
      const height = keyHeight - 2; // Small margin for separation
      
      ctx.fillRect(offsetX + note.ticks, (max - note.midi) * keyHeight, width, keyHeight);

    });

  }

  onMount(async () => {
    await initVoiceMidiTrack(); 
    const keyHeight = 15; // Height of each piano key rectangle
    const keyWidth = 15; // Width of each piano key
    const offsetX = 0; // Left offset for the piano roll
    const offsetY = 0; // Top offset for the piano roll
    const min = minMidi(SingingMidi);
    const max = maxMidi(SingingMidi);
    drawPianoRoll(keyHeight, keyWidth, offsetX, offsetY, min, max); 
  });
</script>

<canvas bind:this={canvas} width="1920" height="1080" style="border: 1px solid black;"></canvas>
