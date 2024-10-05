<script>
  import { onMount } from "svelte";

  let hertz = 0;
  let isRecording = false;
  let audioContext;
  let analyser;
  let microphone;

  // Function to start capturing audio
  async function startRecording() {
    if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
      alert("Your browser does not support audio capture.");
      return;
    }

    // Create audio context
    audioContext = new (window.AudioContext || window.webkitAudioContext)();
    
    // Access the microphone
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    microphone = audioContext.createMediaStreamSource(stream);
    
    // Create analyser node
    analyser = audioContext.createAnalyser();
    analyser.fftSize = 2048;

    // Connect the microphone to the analyser
    microphone.connect(analyser);

    // Start analyzing the audio
    isRecording = true;
    analyzePitch();
  }

  // Function to analyze pitch
  function analyzePitch() {
    const dataArray = new Uint8Array(analyser.frequencyBinCount);

    function getPitch() {
      // Get the frequency data
      analyser.getByteFrequencyData(dataArray);
      
      // Find the index of the highest frequency
      let maxIndex = 0;
      for (let i = 1; i < dataArray.length; i++) {
        if (dataArray[i] > dataArray[maxIndex]) {
          maxIndex = i;
        }
      }

      // Calculate the frequency in Hertz
      const frequency = (maxIndex * audioContext.sampleRate) / analyser.fftSize;
      hertz = frequency.toFixed(2);

      if (isRecording) {
        requestAnimationFrame(getPitch); // Call recursively to keep updating
      }
    }

    getPitch(); // Start the pitch detection
  }

  // Function to stop capturing audio
  function stopRecording() {
    isRecording = false;
    if (microphone) {
      microphone.disconnect();
    }
    if (audioContext) {
      audioContext.close();
    }
  }

  onMount(() => {
    startRecording(); // Automatically start recording when the component mounts
  });

  // Clean up when the component is destroyed
  import { onDestroy } from "svelte";
  onDestroy(() => {
    stopRecording();
  });
</script>

<style>
  .pitch-detector {
    text-align: center;
    margin: 20px;
  }
</style>

<div class="pitch-detector">
  <h1>Voice Pitch Detector</h1>
  <p>Detected Pitch: {hertz} Hz</p>
  <button on:click={stopRecording}>Stop</button>
</div>