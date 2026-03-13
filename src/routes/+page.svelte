<script lang="ts">
	type Group = {
		chats: Chat[];
		sender: 'mine' | 'yours' | 'narration';
	};

	type Chat = {
		id: string;
		message: message;
		edited: boolean;
	};

	let groups: Group[] = $state([]);

	function onkeypress(event: Event) {
		if (event.key === 'Enter') {
			const input = event.target as HTMLInputElement;
			const message = input.value.trim();

			if (!message) return;

			if (mode === MODE_EDIT && editTarget) {
				editTarget.textContent = message;
				mode = MODE_DEFAULT;
				editTarget = null;
				input.value = '';
				mode = MODE_DEFAULT;
				return;
			}

			const sender = mode === MODE_NARRATION ? 'narration' : 'mine';

			const lastGroup = groups[groups.length - 1];
			const newChat: Chat = {
				id: crypto.randomUUID(),
				message,
				edited: false
			};

			if (lastGroup && lastGroup.sender === sender) {
				lastGroup.chats.push(newChat);
			} else {
				groups.push({
					sender,
					chats: [newChat]
				});
			}

			input.value = '';

			// scroll down
			setTimeout(() => {
				const log = document.querySelector('.log');
				if (log) {
					log.scrollTop = log.scrollHeight;
				}
			});

			return;
		}
	}

	const MODE_DEFAULT = 'default';
	const MODE_NARRATION = 'narration';
	const MODE_EDIT = 'edit';
	let mode = $state(MODE_DEFAULT);

	function onkeydown(event: Event) {
		if (event.key === 'Tab') {
			event.preventDefault();
			for (const group of groups) {
				group.sender =
					group.sender === 'mine' ? 'yours' : group.sender === 'yours' ? 'mine' : group.sender;
			}
			groups = groups;
			return;
		}

		if (event.key === 'n' && event.ctrlKey) {
			event.preventDefault();
			mode = mode === MODE_DEFAULT ? MODE_NARRATION : MODE_DEFAULT;
			return;
		}

		if (event.key === 'ArrowUp') {
			event.preventDefault();
			const input = event.target as HTMLInputElement;
			if (input.value) return;

			const lastGroup = groups[groups.length - 1];
			if (!lastGroup) return;

			const lastChat = lastGroup.chats[lastGroup.chats.length - 1];
			if (!lastChat) return;

			startEdit(lastChat.id)();
			return;
		}
	}

	let editTarget: HTMLElement | null = null;

	function startEdit(chatId: string) {
		return () => {
			if (mode === MODE_EDIT) return;

			for (const group of groups) {
				const chat = group.chats.find((c) => c.id === chatId);
				if (chat) {
					mode = MODE_EDIT;
					setTimeout(() => {
						const input = document.querySelector('.input input') as HTMLInputElement;
						input.value = chat.message;
						editTarget = document.querySelector(
							`.log .${group.sender} [data-chat-id="${chatId}"]`
						) as HTMLElement;
						input.focus();
					});
					break;
				}
			}
		};
	}
</script>

<svelte:head>
	<style>
		@import url('https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard-dynamic-subset.min.css');

		body {
			margin: 0;
			font-family: 'Pretendard', sans-serif;
		}

		input {
			font-family: 'Pretendard', sans-serif;
		}
	</style>
	<title>Selftalk</title>
</svelte:head>

<div class="container">
	<div class="content">
		<div class="log">
			{#each groups as group}
				<div class="group">
					{#each group.chats as chat}
						<div class={group.sender} ondblclick={startEdit(chat.id)}>
							{chat.message}
						</div>
					{/each}
				</div>
			{/each}
		</div>
		<div class="input">
			<input
				type="text"
				{onkeydown}
				{onkeypress}
				class:narration-mode={mode === MODE_NARRATION}
				class:edit-mode={mode === MODE_EDIT}
			/>
		</div>
	</div>
</div>

<style>
	.container {
		width: 100%;
		height: 100vh;
		background-color: #f0f0f0;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.content {
		width: 300px;
		height: 500px;
		padding: 20px;
		background-color: #7ab2d6;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
		border-radius: 8px;
		display: flex;
		flex-direction: column;
	}

	.log {
		flex: 1;
		overflow-y: auto;
	}

	.group {
		margin-bottom: 12px;
	}

	.mine,
	.yours {
		padding: 6px 8px;
		margin: 4px 0;
		border-radius: 8px;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
		max-width: 250px;
		width: fit-content;
	}

	.mine {
		background-color: #ffce00;
		margin-left: auto;
	}

	.yours {
		background-color: white;
	}

	.input {
		margin-top: 12px;
	}

	.narration {
		background-color: rgba(0, 0, 0, 0.1);
		text-align: center;
		padding: 4px 20px;
		border-radius: 9999px;
		margin: 8px auto;
		font-size: 12px;
		color: #333;
		width: fit-content;
	}

	.input input {
		width: 100%;
		padding: 8px;
		border: none;
		border-radius: 4px;
		box-sizing: border-box;
		outline: none;
	}

	.narration-mode {
		background-color: #d0d0d0;
	}

	.edit-mode {
		background-color: #ffe0e0;
	}
</style>
