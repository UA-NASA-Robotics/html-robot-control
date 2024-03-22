<script lang="ts">
	import { onMount } from 'svelte';

	let ws: WebSocket;
	let address: string = '';
	let msg: string = '';
	let statusText: string;
	let messages: string[] = [];
	let gp: Gamepad | null;

	async function connect() {
		statusText = 'connecting to ws://172.28.88.24:9002';
		try {
			ws = await connectToServer();
			statusText = 'successfully connected';
			ws.onmessage = (e) => {
				messages.push(e.data.toString());
			};
		} catch (e) {
			statusText = String(e);
		}
	}

	async function connectToServer(): Promise<WebSocket> {
		const ws = new WebSocket(address);
		return new Promise((resolve, reject) => {
			const timer = setInterval(() => {
				if (ws.readyState === 1) {
					clearInterval(timer);
					resolve(ws);
				}
			}, 10);
		});
	}

	async function sendMsg() {
		statusText = `Sending "${msg}"`;
		ws.send(msg);
		// ws.send(msg, (err) => {
		// 	if (err) {
		// 		statusText = err.message;
		// 	} else {
		// 		statusText = 'Success';
		// 	}
		// });
	}

	onMount(() => {
		alert('hi');
		window.addEventListener('gamepadconnected', (e) => {
			const gamepads = navigator.getGamepads();
			if (gamepads.length > 0) {
				gp = gamepads[0];
			}
			console.log(
				'Gamepad connected at index %d: %s. %d buttons, %d axes.',
				e.gamepad.index,
				e.gamepad.id,
				e.gamepad.buttons.length,
				e.gamepad.axes.length
			);
		});

		setInterval(() => {
			const gamepads = navigator.getGamepads();
			if (gamepads.length > 0) {
				gp = gamepads[0];
			}
		}, 50);
	});
</script>

<h1>{statusText}</h1>
<input bind:value={address} placeholder="Enter address" />
<button on:click={connect}>
	Connect to {address}
</button>

<input bind:value={msg} />
<button on:click={sendMsg}>
	Send "{msg}"
</button>
<!-- {#each messages as message}
    <p>{message}</p>
{/each} -->
<p>{gp?.connected}</p>
{#each gp?.axes || [] as axes}
	<p>{axes}</p>
{/each}
