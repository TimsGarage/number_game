<script lang="ts">
    import { onMount } from "svelte";
    
    type Props = {
        callback: (value: string[]) => void;
        value?: string[];
        disabled?: boolean;
        length?: number;
    };

    let {
        length = 3, // Default length
        value,
        disabled = false,
        callback
    }: Props = $props();

    let inputs = [];
    let otp = $state(Array(length).fill(""));
  
    onMount(() => {
      if (value !== undefined && value.length === length) {
        otp = value;
      }
    });
  
    function handleInput(event, index) {
      const input = event.target;
      const val = input.value;
      if (/^[0-9a-zA-Z]$/.test(val)) {
        otp[index] = val;
        updateValue();
        if (index < length - 1) inputs[index + 1].focus();
      } else {
        input.value = "";
      }
    }
  
    function handleKeyDown(event, index) {
      const key = event.key;
      if (key === "Backspace") {
        if (otp[index]) {
          otp[index] = "";
          updateValue();
        } else if (index > 0) {
          inputs[index - 1].focus();
        }
      } else if (key === "ArrowLeft" && index > 0) {
        inputs[index - 1].focus();
      } else if (key === "ArrowRight" && index < length - 1) {
        inputs[index + 1].focus();
      } else if (key === "Enter" && index === length - 1 && otp.every(ch => ch)) {
        callback(otp)
        otp = Array(length).fill(""); // Reset after submission
        inputs.forEach(input => input.value = ""); // Clear inputs
        inputs[0].focus(); // Focus the first input
      }
    }
  
    function handlePaste(event) {
      const pasted = event.clipboardData.getData("text").slice(0, length);
      if (!/^[0-9a-zA-Z]+$/.test(pasted)) return;
      otp = pasted.split("").slice(0, length);
      updateValue();
      // Set input values and focus last
      otp.forEach((ch, i) => (inputs[i].value = ch));
      if (otp.length < length) inputs[otp.length].focus();
      else inputs[length - 1].focus();
    }
  
    function updateValue() {
      value = otp.join("");
    }
  </script>
  
  <div class="otp-container" on:paste={handlePaste}>
    {#each Array(length) as _, i}
      <input
        {disabled}
        class="otp-input"
        bind:this={inputs[i]}
        maxlength="1"
        on:input={e => handleInput(e, i)}
        on:keydown={e => handleKeyDown(e, i)}
        bind:value={otp[i]}
      />
    {/each}
  </div>
  
  <style>
    .otp-container {
      display: flex;
      gap: 0.5rem;
      width: fit-content;
    }
  
    .otp-input {
      width: 2rem;
      height: 2.5rem;
      text-align: center;
      font-size: 1.25rem;
      border: 1px solid #ccc;
      border-radius: 4px;
    }
  
    .otp-input:focus {
      outline: none;
      border-color: #007bff;
      box-shadow: 0 0 0 2px rgba(0,123,255,0.25);
    }

    .otp-input:disabled {
        color: #919191;
    }
  </style>