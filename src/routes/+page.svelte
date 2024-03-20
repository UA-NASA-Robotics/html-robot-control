<script lang="ts">
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
</script>

<h1>{statusText}</h1>
<button on:click={connect}>
    Connect to "ws://172.28.88.24:9002"
</button>
<button on:click={sendHello}>
    Send "hello"
</button>
{#each messages as message}
    <p>message</p>
{/each}