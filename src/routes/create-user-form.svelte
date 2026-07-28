
<script lang="ts">
	let { createSuccessful } = $props();

	let username = $state('');
	let password = $state('');
	let apiError = $state('');
	let usernameInvalid = $derived(username.trim() === '');
	let passwordTooShort = $derived(password.length < 10);
	let passwordTooLong = $derived(password.length > 24);
	let passwordHasSpaces = $derived(/\s/.test(password));
	let passwordMissingNumber = $derived(!/[0-9]/.test(password));
	let passwordMissingUppercase = $derived(!/[A-Z]/.test(password));
	let passwordMissingLowercase = $derived(!/[a-z]/.test(password));
	const CHALLENGE_SIGNUP_ENDPOINT =
'https://api.challenge.hennge.com/password-validation-challenge-api/001/challenge-signup';
	function extractTokenFromChallengeDetailsPath(pathname: string): string {
		const match = pathname.match(/\/challenge-details\/([^/]+)/);
		return match?.[1] ?? '';
	}

	function getAuthorizationHeaderFromPathname(pathname: string): Record<string, string> {
		const token = extractTokenFromChallengeDetailsPath(pathname);
		if (!token) {
			return {};
		}

		return {
			Authorization: `Bearer ${token}`
		};
	}

	let passwordInvalid = $derived(
		passwordTooShort ||
			passwordTooLong ||
			passwordHasSpaces ||
			passwordMissingNumber ||
			passwordMissingUppercase ||
			passwordMissingLowercase
	);

	let formInvalid = $derived(usernameInvalid || passwordInvalid);

	async function handleSubmit(event: SubmitEvent) {
		event.preventDefault();

		if (formInvalid) {
			return;
		}

		apiError = '';

		try {
			const authorizationHeader = getAuthorizationHeaderFromPathname(window.location.pathname);
			const response = await fetch(CHALLENGE_SIGNUP_ENDPOINT, {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json',
					...authorizationHeader
				},
				body: JSON.stringify({
					username,
					password
				})
			});

			if (response.ok) {
				apiError = '';
				createSuccessful();
				return;
			}

			if (response.status === 401 || response.status === 403) {
				apiError = 'Not authenticated to access this resource.';
				return;
			}

			if (response.status === 500) {
				apiError = 'Something went wrong, please try again.';
				return;
			}

			apiError = 'Sorry, the entered password is not allowed, please try a different one.';
		} catch {
			apiError = 'Something went wrong, please try again.';
		}
	}
</script>


<div class="form-wrapper">
	<form class="form" onsubmit={handleSubmit}>
		<!-- make sure the username and password are submitted -->
		<!-- make sure the inputs have the accessible names of their labels -->
		{#if apiError}
			<p role="alert">
    {apiError}
			</p>
		{/if}

		<label for="username">Username</label>
		<input id="username" name="username" bind:value={username} aria-invalid={usernameInvalid} />

		<label for="password">Password</label>
		<input id="password"  name="password" bind:value={password} type="password" aria-invalid={passwordInvalid} />

		{#if password.length > 0}
			<ul class="password-errors">
				{#if passwordTooShort}
					<li>Password must be at least 10 characters long</li>
				{/if}

				{#if passwordTooLong}
					<li>Password must be at most 24 characters long</li>
				{/if}

				{#if passwordHasSpaces}
					<li>Password cannot contain spaces</li>
				{/if}

				{#if passwordMissingNumber}
					<li>Password must contain at least one number</li>
				{/if}

				{#if passwordMissingUppercase}
					<li>Password must contain at least one uppercase letter</li>
				{/if}

				{#if passwordMissingLowercase}
					<li>Password must contain at least one lowercase letter</li>
				{/if}
			</ul>
		{/if}

		<button class="submit-button" disabled={formInvalid}>
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

	.password-errors {
		margin: 0;
		padding-left: 20px;
	}

	.password-errors li {
		margin: 0;
		font-size: 14px;
	}
</style>
