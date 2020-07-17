<template>
    <div class="chat">
        <h1>Chat</h1>
        <p
            v-for="message in messages"
            v-bind:key="message">
            {{ message }}
        </p>
        <div class="new-message">
            <textarea v-model="newMessage" v-on:keyup.enter.exact="sendMessage" placeholder="Write a message"></textarea>
        </div>
    </div>
</template>

<script>
import { HubConnectionBuilder, LogLevel } from '@aspnet/signalr'

export default {
    name: 'Chat',
    data: function() {
        return {
            messages: [],
            newMessage: '',
            connection: null
        }
    },
    methods: {
        sendMessage: function() {
            if (!/\S+/.test(this.newMessage)) return;

            this.connection.invoke("SendMessage", this.newMessage).catch(err => console.error(err));
            this.newMessage = '';
        }
    },
    mounted: function() {
        const connection = new HubConnectionBuilder()
            .withUrl("/chatHub")
            .configureLogging(LogLevel.Information)
            .build();

        async function start() {
            try {
                await connection.start();
                console.log("connected");
            } catch (err) {
                console.log(err);
                setTimeout(() => start(), 5000);
            }
        }

        connection.onclose(async () => {
            await start();
        });

        // Start the connection.
        start();

        // register a handler
        connection.on("ReceiveMessage", (message) => {
            this.messages.push(message);
        });

        this.connection = connection;
    }
}
</script>

<style scoped>
.chat {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
}
.chat > p {
    text-align: center;
    white-space: pre-line;
    margin: 0 1em 1em 1em;
    padding: 1em;
    background-color: aqua;
    border-radius: 1em;
}
.new-message {
    width: calc(100% - 3em);
    margin: 0 3em;
    padding: 0.5em;
    border-radius: 1em;
    background-color: #eeeeee;
}
textarea {
    width: 100%;
    height: 4em;
    resize: none;
    font-family: inherit;
    outline: none;
    border: none;
    background-color: inherit;
}
</style>