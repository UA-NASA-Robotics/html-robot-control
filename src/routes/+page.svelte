<script lang="ts">
	import { onMount } from 'svelte';

	let ws: WebSocket;
	let gp: Gamepad | null;

	// Text input variables
	let url: string = 'ws://192.168.1.141:9002';

	let statusText: string;
	let messages: string[] = [];

	let previousPacket: [number, number];
	let prevString: string;

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

	async function disconnect() {
		statusText = 'disconnecting with ' + url;
		ws.send('stop-listening');
		ws.close();
	}

	// Sends a formatted packet to the server with the data in the parameters
	async function sendMotionPacket(
		leftWheel: number,
		rightWheel: number,
		triggers: [boolean, boolean, boolean, boolean]
	) {
		// if (leftWheel < -1 || leftWheel > 1 || rightWheel < -1 || rightWheel > 1 || !ws) return;

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

		let leftMagnitude = Math.round(Math.abs(leftWheel) * 7);
		let rightMagnitude = Math.round(Math.abs(rightWheel) * 7);
		let isLeftNegative = leftWheel > 0; // Set it to be opposite of what is being read
		let isRightNegative = rightWheel > 0; // Sama as above

		// If left wheel is negative set the left sign bit to 1
		if (isLeftNegative) bytes[1] |= 0b1000_0000;
		// If right wheel is negative set the right sign bit to 1
		if (isRightNegative) bytes[1] |= 0b0000_1000;

		// Add left wheel magnitude data to bits 1...3 in byte 1
		bytes[1] |= (leftMagnitude << 4) & 0b0111_0000;

		// Add right wheel magnitude data to bits 5...7 in byte 1
		bytes[1] |= rightMagnitude & 0b0000_0111;

		// If packet is identical to previously sent packet return
		// if (bytes[0] == previousPacket[0] && bytes[1] == previousPacket[1]) return;
		previousPacket = bytes;

		ws?.send(new Int8Array(bytes));
		prevString = '';
		for (const char of String.fromCharCode(...bytes)) {
			prevString += char.charCodeAt(0).toString(2).padStart(8, '0') + ' ';
		}
	}

	let sentMacros: boolean[] = [];
	const macroMap = [
		5, // Dump cycle
		6 // Dig cycle
	];

	async function sendMacroPacket(buttons: readonly GamepadButton[]) {
		let macroCode = -1;
		for (let i = buttons.length - 1; i >= 0; i--) {
			if (!buttons[i].pressed) {
				sentMacros[i] = false;
				continue;
			}

			if ((sentMacros.length > i && sentMacros[i]) || macroMap.length <= i) continue;

			macroCode = macroMap[i];
			sentMacros[i] = true;
		}

		if (macroCode < 0) return;

		let byte = 0b1000_0010;
		byte |= (macroCode << 2) & 0b0111_1100;

		ws?.send(new Int8Array([byte]));
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
				if (gp) {
					const left = gp.axes[1];
					const right = gp.axes[3];
					const t = [...gp.buttons.slice(4, 8).map((b) => b.pressed)];
					sendMotionPacket(left, right, [t[0], t[1], t[2], t[3]]);
					sendMacroPacket(gp.buttons);
				}
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
<button on:click={disconnect}> Disconnect </button>

<p>{prevString}</p>

{#each gp?.axes || [] as axes}
	<p>{axes}</p>
{/each}

{#each gp?.buttons || [] as button, index}
	<p>{index} {button.pressed} {button.touched} {button.value}</p>
{/each}
