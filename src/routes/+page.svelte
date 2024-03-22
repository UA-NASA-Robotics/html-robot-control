<script lang="ts">
	import { onMount } from 'svelte';

	let ws: WebSocket;
	let address: string = '';
	let statusText: string;
	let messages: string[] = [];
	let gp: Gamepad | null;

	async function connect() {
		statusText = `Connecting to ${address}`;
		try {
			ws = await new Promise((resolve, reject) => {
				const ws = new WebSocket(address);
				const timer = setInterval(() => {
					if (ws.readyState === 1) {
						clearInterval(timer);
						resolve(ws);
					}
				}, 10);
			});

			statusText = `Successfully connected to ${address}`;

			ws.onmessage = (e) => {
				messages.push(e.data.toString());
			};

			ws.onclose = function (e) {
				statusText = `Closed connection with ${this.url} because ${e.reason}`;
			};

			ws.onerror = function (e) {
				statusText = `Error with ${this.url}`;
			};
		} catch (e) {
			statusText = String(e);
		}
	}

	onMount(() => {
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

<p>{statusText}</p>
<input bind:value={address} placeholder="Enter address" />
<button on:click={connect}>
	Connect to {address}
</button>
<!-- {#each messages as message}
    <p>{message}</p>
{/each} -->
<p>{gp?.connected}</p>
{#each gp?.axes || [] as axes}
	<p>{axes}</p>
{/each}
