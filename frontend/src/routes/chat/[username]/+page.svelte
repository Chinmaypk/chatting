<script lang="ts">
	import type { PageData } from './$types';
	import Contact from '$lib/components/Contacts/Contact.svelte';
	import { session } from '$lib/store/user.store';
	import You from '$lib/components/Message/You.svelte';
	import Other from '$lib/components/Message/Other.svelte';
	import { messages, sendMessage } from '$lib/store/message.store';
	import { browser } from '$app/environment';
	import { page } from '$app/stores';
	import { BASE_URI_DOMAIN } from '$lib/constants';
	import type { Message } from '$lib/types/message.interface';
	export let data: PageData;
	const fullName = `${JSON.parse(data.context.user_object)[0].fields.first_name} ${
		JSON.parse(data.context.user_object)[0].fields.last_name
	}`;

	let messageInput: string, socket: WebSocket;
	if (browser) {
		// var u1 = parseInt(${JSON.parse(data.context.user_object.toString())[0].pk});
		// if (JSON.parse(data.context.user_object)[0].pk < session.user.pk ) {
		// 	greeting = "Good day";
		// }
		// else {
		// 	greeting = "Good evening";
		// }
		const websocketUrl = `${
			$page.url.protocol.split(':')[0] === 'http' ? 'ws' : 'wss'
		}://${BASE_URI_DOMAIN}/ws/chat/${JSON.parse(data.context.user_object)[0].pk}/?${
			$session.user.pk
		}`;

		socket = new WebSocket(websocketUrl);

		socket.addEventListener('open', () => {
			console.log('Connection established!');
		});

		socket.addEventListener('message', (event) => {
			const data = JSON.parse(event.data);
			const messageList: Array<Message> = JSON.parse(data.messages).map((message: any) => {
				return {
					message: message.fields.message,
					thread_name: message.fields.thread_name,
					timestamp: message.fields.timestamp,
					sender__pk: message.fields.sender__pk,
					sender__username: message.fields.sender__username,
					sender__last_name: message.fields.sender__last_name,
					sender__first_name: message.fields.sender__first_name,
					sender__email: message.fields.sender__email,
					sender__is_staff: message.fields.sender__is_staff,
					sender__is_active: message.fields.sender__is_active,
					sender__is_superuser: message.fields.sender__is_superuser
				};
			});
			$messages = messageList;
			messageInput = '';
		});
	}
	console.log('Hi');
	function getConn() {
		const websocketUrl = `${
			$page.url.protocol.split(':')[0] === 'http' ? 'ws' : 'wss'
		}://${BASE_URI_DOMAIN}/ws/chat/${JSON.parse(data.context.user_object)[0].pk}/?${
			$session.user.pk
		}`;

		socket = new WebSocket(websocketUrl);

		socket.addEventListener('open', () => {
			console.log('Connection established!');
		});

		socket.addEventListener('message', (event) => {
			const data = JSON.parse(event.data);
			const messageList: Array<Message> = JSON.parse(data.messages).map((message: any) => {
				return {
					message: message.fields.message,
					thread_name: message.fields.thread_name,
					timestamp: message.fields.timestamp,
					sender__pk: message.fields.sender__pk,
					sender__username: message.fields.sender__username,
					sender__last_name: message.fields.sender__last_name,
					sender__first_name: message.fields.sender__first_name,
					sender__email: message.fields.sender__email,
					sender__is_staff: message.fields.sender__is_staff,
					sender__is_active: message.fields.sender__is_active,
					sender__is_superuser: message.fields.sender__is_superuser
				};
			});
			$messages = messageList;
			messageInput = '';
		});
	}
	async function myfunc() {
		await getConn();
	}

	function sleep(time) {
		return new Promise((resolve) => setTimeout(resolve, time));
	}

	// Usage!
	
	const handleSendMessage = (event: MouseEvent) => {
		event.preventDefault();
		myfunc();

		sleep(1000).then(() => {
		// Do something after the sleep!
		
			sendMessage(messageInput, $session.user.username as string, socket);
		});
		// if (browser) {
		// 	// var u1 = parseInt(${JSON.parse(data.context.user_object.toString())[0].pk});
		// 	// if (JSON.parse(data.context.user_object)[0].pk < session.user.pk ) {
		// 	// 	greeting = "Good day";
		// 	// }
		// 	// else {
		// 	// 	greeting = "Good evening";
		// 	// }
		// 	const websocketUrl = `ws://${BASE_URI_DOMAIN}/ws/chat/${
		// 		JSON.parse(data.context.user_object)[0].pk
		// 	}/?${$session.user.pk}`;

		// 	socket = new WebSocket(websocketUrl);

		// }
		// if (browser) {
		// var u1 = parseInt(${JSON.parse(data.context.user_object.toString())[0].pk});
		// if (JSON.parse(data.context.user_object)[0].pk < session.user.pk ) {
		// 	greeting = "Good day";
		// }
		// else {
		// 	greeting = "Good evening";
		// }

		// myfunc().then(
		// 	function(value){
		// 	sendMessage(messageInput, $session.user.username as string, socket);
		// 	}
		// );
	};

	const handleSendFile = (event: MouseEvent) => {
		// event.preventDefault();
		// sendMessage(messageInput, $session.user.username as string, socket);
		document.getElementById('input-file').click();
	};
</script>

<div class="container py-5">
	<div class="row">
		<div class="col-md-6 col-lg-5 col-xl-4 mb-4 mb-md-0 scrollable">
			<h5 class="font-weight-bold mb-3 text-center text-lg-start">
				{$session.user.username}'s contacts
			</h5>
			<Contact contacts={JSON.parse(data.context.users)} />
		</div>
		<div class="col-md-6 col-lg-7 col-xl-8 scrollable" id="message-wrapper">
			<h5 class="font-weight-bold mb-3 text-center text-lg-start">
				{fullName}
			</h5>
			<ul class="list-unstyled" id="chat-body">
				{#each $messages as message, id}
					{#if message.sender__pk === $session.user.pk}
						<You {message} />
					{:else}
						<Other {message} />
					{/if}
				{/each}
			</ul>
			<input id="input-file" type="file" name="somename" style="visibility:collapse" />
			<div
				class="text-muted d-flex justify-content-start align-items-center pe-3 pt-3 mt-2 message-control"
			>
				<img
					src="https://mdbcdn.b-cdn.net/img/Photos/Avatars/avatar-{$session.user.pk}.webp"
					alt="You"
					title="You"
					style="width: 40px; height: 100%"
				/>
				<textarea
					placeholder="Type message"
					class="form-control form-control-lg"
					id="message-body"
					rows="1"
					bind:value={messageInput}
				/>

				<a
					class="ms-3"
					id="send-message-btn"
					title="Send"
					href={null}
					on:click={(event) => handleSendMessage(event)}
				>
					<i class="fas fa-paper-plane" />
				</a>
				<a
					class="ms-3"
					id="send-file-btn"
					title="Send"
					href={null}
					on:click={(event) => handleSendFile(event)}
				>
					<i class="fas fa-paperclip" />
				</a>
				<!-- <input id="input-file" type="file" name="somename" style="visibility:collapse"> -->
			</div>
		</div>
	</div>
</div>
