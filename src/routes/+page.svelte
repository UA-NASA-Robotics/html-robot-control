<script lang="ts">
	import { onMount } from 'svelte';
    import WebSocket from 'ws';

    let ws: WebSocket;
    let statusText: string;
    let messages: string[] = [];

    async function connect() {
        statusText = "connecting to ws://172.28.88.24:9002";
        ws = await connectToServer();
        ws.onmessage = (e) => {
            messages.push(e.data.toString());
        }
        statusText = "successfully connected"
    }
    
    async function connectToServer(): Promise<WebSocket> {
        const ws = new WebSocket('ws://172.28.88.24:9002');
        return new Promise((resolve, reject) => {
            const timer = setInterval(() => {
                if(ws.readyState === 1) {
                    clearInterval(timer)
                    resolve(ws);
                }
            }, 10);
        });
    }

    async function sendHello() {
        statusText = 'sending "hi"';
        ws.send("hi", err => {
            if (err) {
                statusText = err.message;
            } else {
                statusText = 'success'
            }
        })
    }

    let gp: Gamepad | null;

    onMount(() => {
        alert('hi')
        window.addEventListener("gamepadconnected", (e) => {
            const gamepads = navigator.getGamepads()
            if (gamepads.length > 0) {
                gp = gamepads[0];
            }
            console.log(
                "Gamepad connected at index %d: %s. %d buttons, %d axes.",
                e.gamepad.index,
                e.gamepad.id,
                e.gamepad.buttons.length,
                e.gamepad.axes.length,
            );
        });

        setInterval(() => {
            const gamepads = navigator.getGamepads();
            if (gamepads.length > 0) {
                gp = gamepads[0];
            }
        }, 50)
    })
</script>

<h1>{statusText}</h1>
<button on:click={connect}>
    Connect to "ws://172.28.88.24:9002"
</button>
<button on:click={sendHello}>
    Send "hello"
</button>
<!-- {#each messages as message}
    <p>{message}</p>
{/each} -->
<p>{gp?.connected}</p>
{#each (gp?.axes || []) as axes}
    <p>{axes}</p>
{/each}