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

    // The X and Y values of the left stick.
    let leftStickXValue: number = 0;
    let leftStickYValue: number = 0;

    // The X and Y values of the right stick.
    let rightStickXValue: number = 0;
    let rightStickYValue: number = 0;

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
                if (gp) {

                    // NOTE: don't know if this logic is correct.
                    // Reads the X and Y values of the left stick.
                    leftStickXValue = gp.axes[0];
                    leftStickYValue = gp.axes[1];

                    // Reads the X and Y values of the right stick.
                    rightStickXValue = gp.axes[2];
                    rightStickYValue = gp.axes[3];
                }
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


<!-- NOTE: UNSURE IF POSITIONING OR FUNCTIONALITY IS CORRECT. Need to test with controller-->
<svg width="400" height="300">
    <!-- Left stick button-->
    <circle cx={(leftStickXValue * 50) + 150} cy={(leftStickYValue * 50) + 150} r={20} fill="gray" />

    <!-- Right stick button-->
    <circle cx={(rightStickXValue * 50) + 300} cy={(rightStickYValue * 50) + 150} r={20} fill="gray" />
</svg>

<!-- {#each messages as message}
    <p>{message}</p>
{/each} -->
<p>{gp?.connected}</p>
{#each (gp?.axes || []) as axes}
    <p>{axes}</p>
{/each}



<!DOCTYPE html>
<html lang="en">

</html>