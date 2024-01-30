<template>
    <div class="container py-4">
        <div class="d-flex flex-column">
            <h1 class="fw-bold mb-3">TOTP</h1>
            <div class="d-flex mb-3">
                <div class="d-inline-flex flex-wrap align-items-center border-bottom">
                    <span class="me-3">Secret:</span>
                    <span class="text-muted user-select-all">{{ secret }}</span>
                </div>
            </div>
            <div class="d-flex align-items-center mb-3">
                <h1 class="clickable mb-0 me-3" @click="copy_totp"><span class="badge bg-primary">{{ totp }}</span></h1>
                <small v-show="copied" class="fw-bold">Copied!</small>
            </div>
            <div class="d-flex">
                <span>Expires in <span class="fw-bold">{{ countdown }}</span> seconds</span>
            </div>
        </div>
        <hr>
        <div class="d-flex flex-column">
            <h1 class="fw-bold mb-3">Algorithm</h1>
            <p>This is a custom implementation that generates time based one-time passwords that change every 30 seconds.
                The secret is combined with the current time and hashed with SHA-256. Then, the hash is converted to a
                32-bit integer and the last 6 digits are taken as the code. The code displayed above changes every 30
                seconds and can be copied by clicking on it.</p>
            <p>Here's the algorithm in JavaScript:</p>
            <pre class="bg-dark text-light p-3 rounded">
async function generate_totp(secret){    
    // Get unixtime with minute precision
    const dt = new Date();
    const seconds = dt.getUTCSeconds() &lt 30 ? '0' : '1';
    const timestring = `${parseInt(Date.now() / 1000 / 60)}${seconds}`

    // Hash the secret + timestring with SHA-256
    const encoder = new TextEncoder();
    const data = encoder.encode(secret + timestring);
    const hash = await crypto.subtle.digest('SHA-256', data);

    // Convert the hash to a 32-bit integer
    const view = new DataView(hash);
    const int = view.getUint32(0);

    // Get the last 6 digits of the integer
    const code = String(int).slice(-6);

    // Return the code
    return code;
}</pre>
            <p>Please note that this might not be secure enough for cryptographical applications.</p>
        </div>
        <hr>
        <div class="d-flex flex-column">
            <h1 class="fw-bold mb-3">Source Code</h1>
            <p>The source code for this page is available on <a href="https://github.com/kaangiray26/totp">GitHub</a>.</p>
            <span>kaangiray26</span>
        </div>
    </div>
</template>

<script setup>
import { ref, onBeforeMount } from 'vue';

// Generate a random hexadecimal of length 16
// const secret = crypto.getRandomValues(new Uint8Array(16)).reduce((p, i) => p + (i % 16).toString(16), '');

const totp = ref('123456');
const secret = "8909d2239ea43a0e";
const countdown = ref(0);

const copied = ref(false);

async function generate_totp() {
    // Get unixtime with minutes precision
    const dt = new Date();
    const seconds = dt.getUTCSeconds() < 30 ? '0' : '1';
    const timestring = `${parseInt(Date.now() / 1000 / 60)}${seconds}`

    // Hash the secret + timestring with SHA-256
    const encoder = new TextEncoder();
    const data = encoder.encode(secret + timestring);
    const hash = await crypto.subtle.digest('SHA-256', data);

    // Convert the hash to a 32-bit integer
    const view = new DataView(hash);
    const int = view.getUint32(0);

    // Get the last 6 digits of the integer
    const code = String(int).slice(-6);

    // Display the code
    totp.value = code;
}

async function get_countdown() {
    countdown.value = 30 - (new Date().getUTCSeconds() % 30);
    if (countdown.value === 30) {
        generate_totp();
    }
}

async function copy_totp() {
    navigator.clipboard.writeText(totp.value)
        .then(() => {
            copied.value = true;
            setTimeout(() => {
                copied.value = false;
            }, 3000);
        })
}

onBeforeMount(() => {
    generate_totp();
    get_countdown();

    setInterval(() => {
        get_countdown();
    }, 1000);
})
</script>