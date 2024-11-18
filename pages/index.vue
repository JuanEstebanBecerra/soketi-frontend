<template>
	<div>
		<button @click="sendMessage">test</button>
	</div>
</template>

<script setup lang="ts">
import Pusher from 'pusher-js'

const pusher = new Pusher('app-key', {
	cluster: 'mt1',
	wsHost: 'localhost',
	wsPort: 6001,
	forceTLS: false,
	enabledTransports: ['ws', 'wss']
})

const channel = pusher.subscribe('chat')

channel.bind('message.sent', (data: object) => {
	console.log('Message received:', data)
})

function sendMessage() {
	channel.trigger('client-message', { test1: 'message sent' })
}
</script>

<style scoped></style>
