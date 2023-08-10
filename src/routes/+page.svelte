<div class="vertical">
	<div class="body">
		<h1 class="h1">Task Timer</h1>
		<h2 class="h2">00:00</h2>
		<textarea class="textarea" rows="16" placeholder="Enter tasks..." />
		<div class="buttons">
			<button type="button" class="btn btn-xl variant-filled-secondary">Clear</button>
			<button type="button" class="btn btn-xl variant-filled-tertiary">Done</button>
			<button type="button" class="btn btn-xl variant-filled-primary">Start</button>
		</div>
	</div>
</div>

<style>
	.vertical {
		height: 100%;
		display: flex;
		margin: 0 !important;
		align-items: center;
	}

	.h1 {
		font-size: 70px;
	}

	.h2 {
		font-size: 50px;
	}

	.h1,
	.h2 {
		text-align: center;
		margin-top: 10px;
	}

	textarea {
		margin-top: 10px;
		padding: 10px;
		resize: none;
		font-size: 20px;
	}

	.body {
		max-width: 800px;
		width: 100vw;
		margin: auto;
		display: flex;
		flex-direction: column;
	}

	.buttons {
		display: flex;
		gap: 10px;
		justify-content: center;
		margin-top: 10px;
	}	

</style>
