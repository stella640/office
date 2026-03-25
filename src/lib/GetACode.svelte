<script>
  import { emailStore } from '../stores.js';
  import MicrosoftSvg from '../assets/microsoft.svg';
  import SignUpFooter from '../lib/SignUpFooter.svelte';
  import { push, link } from "svelte-spa-router"; // Use link for <a> tags

  function goToOtp() {
    push("/otp");
  }
</script>

<div class="wrapper">
  <div class="container">
    <div class="card">
      <div class="logo-wrapper">
        <img src={MicrosoftSvg} alt="microsoft" class="logo"/>
        <div class="email-display">{$emailStore || 'user@example.com'}</div>
      </div>

      <h2>Get a Code to Sign in</h2>
      <p class="subtitle">We'll send a code to <span class="font-bold">{$emailStore || 'your email'}</span> to sign you in.</p>

      <div class="actions">
        <button class="next-btn" on:click={goToOtp}> 
          Send Code 
        </button>

        <a href="/enter-your-password" use:link class="secondary-link">
          Use your password
        </a>
      </div>
    </div>
  </div>
  <SignUpFooter/>
</div>

<style>
  /* Standard Wrapper/Card Styles */
  .wrapper {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    background: white;
  }

  .card {
    background: white;
    padding: 20px;
    max-width: 440px;
    margin: 0 auto;
  }

  .logo-wrapper {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .logo {
    width: 108px;
    margin-bottom: 24px;
  }

  .email-display {
    font-size: 14px;
    padding: 6px 12px;
    border-radius: 24px;
    border: 1px solid #f0f0f0;
    margin-bottom: 16px;
  }

  h2 {
    font-size: 24px;
    font-weight: 600;
    text-align: center;
    margin-bottom: 12px;
  }

  .subtitle {
    font-size: 15px;
    margin-bottom: 32px;
    text-align: center;
    color: #1b1b1b;
  }

  /* ACTION LAYOUT */
  .actions {
    display: flex;
    flex-direction: column;
    gap: 20px; /* Space between button and link */
  }

  .next-btn {
    background: #0067b8;
    color: white;
    border: none;
    padding: 10px 24px;
    font-size: 15px;
    cursor: pointer;
    width: 100%; /* Takes full width of container */
    transition: background 0.2s;
    outline: none;
  }

  .next-btn:hover {
    background: #005da6;
  }

  .secondary-link {
    display: flex;
    justify-content: center;
    align-items: center;
    /* display: inline-block; */
    color: #0067b8;
    text-decoration: none;
    font-size: 14px;
    cursor: pointer;
  }

  .secondary-link:hover {
    text-decoration: underline;
  }

    .font-bold {
    font-weight: 600;
  }

  @media (min-width: 768px) {
    .wrapper { background: #f2f2f2; justify-content: center; align-items: center; }
    .card { padding: 44px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
  }

  @media (min-width: 400px) and (max-width: 1023px) {
    .container {
        padding: 0 47px;
    }
}
/* DESKTOP VERSION */

@media (min-width:600px){

.wrapper{
    justify-content:center;
    align-items:center;
    background:url('../assets/background.svg');
    background-size:cover;
}

/* container centers card */

.container{
    max-width:440px;
    margin-top: 80px;
}

/* restore card style */

.card{
    background:white;
    padding:44px;
    border-radius:4px;
    box-shadow:0 4px 20px rgba(0,0,0,0.15);
    margin-bottom:20px;
}

/* options card becomes card */

.options-card{
    background:white;
    padding:15px 25px;
    border:none;
    box-shadow:0 2px 10px rgba(0,0,0,0.1);
}

/* footer bottom right */

:global(footer){
    left:auto;
    right:20px;
}

}
</style>