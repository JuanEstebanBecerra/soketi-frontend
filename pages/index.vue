<template>
	<div>
		<button @click="test">test</button>
	</div>
</template>

<script setup lang="ts">
import Echo from 'laravel-echo'
import Pusher from 'pusher-js'

const test = () => {
	console.log(123)

	let client = new Pusher('app-key', {
		wsHost: '127.0.0.1',
		wsPort: 6001,
		forceTLS: false,
		cluster: 'mt1',
		// encrypted: true,
		disableStats: true,
		enabledTransports: ['ws', 'wss'],
	})

	client.subscribe('channel1').bind('message', (message: string) => {
		alert(`${message.sender} says: ${message.content}`)
	})
}
</script>

<style scoped></style>
