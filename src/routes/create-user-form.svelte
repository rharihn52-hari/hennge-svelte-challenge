
<script lang="ts">
	let { createSuccessful } = $props();

	let username = $state('');
	let password = $state('');
	let usernameInvalid = $derived(username.trim() === '');

	function handleSubmit(event: SubmitEvent) {
		event.preventDefault();
		if (usernameInvalid) {
			return;
		}
		createSuccessful();
	}
</script>


<div class="form-wrapper">
	<form class="form" on:submit={handleSubmit}>
		<!-- make sure the username and password are submitted -->
		<!-- make sure the inputs have the accessible names of their labels -->
		<label for="username">Username</label>
		<input id="username" bind:value={username} />

		<label for="password">Password</label>
		<input id="password" bind:value={password} type="password" />

		<button class="submit-button" disabled={usernameInvalid}>
			Create User
		</button>
	</form>
</div>

<style>
	.form-wrapper {
		max-width: 500px;
		width: 80%;
		background-color: #efeef5;
		padding: 24px;
		margin: auto;
		border-radius: 8px;
	}

	.form {
		display: flex;
		gap: 8px;
		flex-direction: column;
	}

	label {
		font-weight: 700;
	}

	input {
		outline: none;
		padding: 8px 16px;
		height: 40px;
		font-size: 14px;
		background-color: #f8f7fa;
		border: 1px solid rgba(0, 0, 0, 0.12);
		border-radius: 4px;
	}

	.submit-button {
		outline: none;
		border-radius: 4px;
		border: 1px solid rgba(0, 0, 0, 0.12);
		background-color: #7135d2;
		color: white;
		font-size: 16px;
		font-weight: 500;
		height: 40px;
		padding: 0 8px;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-top: 8px;
		align-self: flex-end;
		cursor: pointer;
	}
</style>
