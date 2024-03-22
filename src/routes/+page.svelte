<script lang="ts">
	import { onMount } from 'svelte';

	let ws: WebSocket;
	let gp: Gamepad | null;

	// Text input variables
	let url: string = '';

	let statusText: string;
	let messages: string[] = [];

	let previousPacket: [number, number];

	// Creates web socket connection with the device with the specified url
	async function connect() {
		statusText = `Connecting to ${url}`;
		try {
			// Set ws = to a new WebSocket instance if succesfully created
			ws = await new Promise((resolve, reject) => {
				const ws = new WebSocket(url);
				const timer = setInterval(() => {
					if (ws.readyState === 1) {
						clearInterval(timer);
						resolve(ws);
					}
				}, 10);
			});

			statusText = `Successfully connected to ${url}`;

			// Set the incoming message handler of the new WebSocket instance
			ws.onmessage = (e) => {
				messages.push(e.data.toString());
			};

			// Set the connection close handler of the new WebSocket instance
			ws.onclose = function (e) {
				statusText = `Closed connection with ${this.url} because ${e.reason}`;
			};

			// Set the error handler of the new WebSocket instance
			ws.onerror = function (e) {
				statusText = `Error with ${this.url}`;
			};
		} catch (e) {
			// WebSocket was not successfully made
			statusText = String(e);
		}
	}

	// Sends a formatted packet to the server with the data in the parameters
	async function sendMotionPacket(
		leftWheel: number,
		rightWheel: number,
		triggers: [boolean, boolean, boolean, boolean]
	) {
		// Declare bytes array with length 2
		let bytes: [number, number] = [0, 0];

		// Add trigger data to bits 1...4 of byte 0
		let temp = 0b0100_0000;
		for (let i = 0; i < 4; i++) {
			if (triggers[i]) {
				bytes[0] |= temp;
			}
			temp >>= 1;
		}

		// Add left wheel data to leftmost 4 bits in byte 1
		bytes[1] |= (leftWheel << 4) & 0b1111_0000;

		// Add right wheel data to rightmost 4 bits in byte 1
		bytes[1] |= rightWheel & 0b0000_1111;

		// If packet is identical to previously sent packet return
		if (!previousPacket || (bytes[0] == previousPacket[0] && bytes[1] == previousPacket[1])) return;
		previousPacket = bytes;

		// Creating new input array buffer
		let byteArray = Buffer.from(bytes);

		// Converting buffer to string
		let str = byteArray.toString();
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

		const timeout = setInterval(() => {
			const gamepads = navigator.getGamepads();
			if (gamepads.length > 0) {
				gp = gamepads[0];
			}
		}, 50);

		window.addEventListener('gamepaddisconnected', (e) => {
			const gamepads = navigator.getGamepads();
			if (gamepads.length <= 0) {
				gp = null;
				clearInterval(timeout);
				sendMotionPacket(0, 0, [false, false, false, false]);
			}
		});
	});
</script>

<p>{statusText}</p>
<input bind:value={url} placeholder="Enter address" />
<button on:click={connect}> Connect </button>

<p>{gp?.connected}</p>
{#each gp?.axes || [] as axes}
	<p>{axes}</p>
{/each}
