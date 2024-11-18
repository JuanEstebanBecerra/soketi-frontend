<template>
	<UNotifications />
	<div
		class="w-screen h-screen flex justify-center items-center bg-gray-800 dark:bg-gray-900">
		<ThemeToggle />
		<form
			v-if="!token"
			@submit.prevent="login"
			class="w-[500px] h-[500px] bg-white dark:bg-gray-800 p-4 rounded-lg shadow-md flex flex-col gap-8">
			<div class="flex-1 flex flex-col gap-8">
				<UFormGroup label="Correo">
					<UInput v-model="formData.email" />
				</UFormGroup>

				<UFormGroup label="Contraseña">
					<UInput v-model="formData.password" />
				</UFormGroup>
			</div>

			<div class="flex justify-end">
				<UButton
					type="submit"
					label="Iniciar sesión"
					size="lg" />
			</div>
		</form>

		<div
			v-else
			class="w-[500px] h-[500px] bg-white dark:bg-gray-800 p-4 rounded-lg shadow-md flex flex-col gap-8">
			<div class="flex justify-between items-center">
				<h1 class="text-xl">Conectar a canal de proyecto</h1>

				<UTooltip
					text="Cerrar sesión"
					:popper="{ arrow: true, placement: 'top' }">
					<UButton
						color="gray"
						icon="i-heroicons-arrow-left-end-on-rectangle-20-solid"
						@click="logout" />
				</UTooltip>
			</div>

			<form
				@submit.prevent="connectToChannel"
				class="flex flex-col gap-4 h-full">
				<UFormGroup label="ID de proyecto">
					<UInput v-model="projectId" />
				</UFormGroup>

				<div class="flex-1 relative w-full">
					<div
						class="absolute w-full h-full overflow-auto flex flex-col gap-4">
						<TransitionGroup
							enter-active-class="transition-all"
							enter-from-class="translate-y-5 opacity-0"
							enter-to-class="opacity-100">
							<div
								v-for="(item, index) in messages"
								:key="index"
								class="shadow bg-gray-100 dark:bg-gray-700 w-full relative rounded-e-xl rounded-es-xl p-2">
								{{ item }}
							</div>
						</TransitionGroup>
					</div>
				</div>

				<div class="flex justify-end">
					<UButton
						type="submit"
						label="Conectar" />
				</div>
			</form>
		</div>
	</div>
</template>

<script setup lang="ts">
import Pusher from 'pusher-js'

const formData = ref({
	email: '',
	password: '',
})

const token = ref('')
let pusher: Pusher | undefined = undefined

const login = () => {
	$fetch<Record<string, any>>('http://localhost:8080/api/auth/login', {
		method: 'POST',
		body: formData.value,
	}).then((response) => {
		token.value = response?.data.access_token

		pusher = new Pusher('app-key', {
			cluster: 'mt1',
			wsHost: 'localhost',
			wsPort: 6001,
			forceTLS: false,
			enabledTransports: ['ws', 'wss'],
			authEndpoint: 'http://localhost:8080/api/broadcasting/auth',
			auth: {
				headers: {
					Authorization: 'Bearer ' + token.value,
					Accept: 'application/json',
				},
			},
		})

		pusher.connection.bind('connected', () => {
			useToast().add({
				title: 'Conexión con soketi exitosa',
				color: 'green',
			})
		})

		pusher.connection.bind('error', (err) => {
			useToast().add({
				title: 'Error de conexión con soketi',
				color: 'red',
			})
			console.error('Connection error:', err)
		})
	})
}

const logout = () => {
	$fetch<Record<string, any>>('http://localhost:8080/api/auth/logout', {
		method: 'POST',
		headers: {
			Authorization: 'Bearer ' + token.value,
			'Access-Control-Allow-Origin': '*',
		},
	}).then((response) => {
		token.value = ''
		pusher?.disconnect()
		messages.value = []
	})
}

const projectId = ref('')
const messages = ref<string[]>([])

const connectToChannel = async () => {
	const channel = await pusher?.subscribe(
		'private-project-' + projectId.value
	)

	channel?.bind('pusher:subscription_succeeded', () => {
		channel?.bind('project.updated', (data: object) => {
			messages.value.push(data?.content)
		})

		useToast().add({
			title: 'Suscrito al canal private-project-' + projectId.value,
			color: 'green',
		})
	})

	channel?.bind('pusher:subscription_error', (status) => {
		useToast().add({
			title: 'Error de conexión al canal',
			color: 'red',
		})
	})
}
</script>

<style scoped></style>
