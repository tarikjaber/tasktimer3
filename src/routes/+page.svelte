<script lang="ts">
	import { parseTasks } from '../utils';
	import type { Task } from '../utils/types';
	import { onMount } from 'svelte';
	import { toastStore, type ToastSettings, Toast } from '@skeletonlabs/skeleton';

	const successToast: ToastSettings = {
		message: 'Successfully completed task.',
		background: 'variant-filled-success'
	};

	const failureToast: ToastSettings = {
		message: 'Mission Failed. Did not complete task in time.',
		background: 'variant-filled-error'
	};

	let playing = false;
	let textarea: HTMLTextAreaElement;
	let intervalId: number;
	let timeLeft = 0;
	let tasks: Task[] = [];
	let failed = false;
	let currentTaskIndex = 0;
	let startTime: number;
	$: timeString =
		Math.floor(timeLeft / 60)
			.toString()
			.padStart(2, '0') +
		':' +
		(timeLeft % 60).toString().padStart(2, '0');

	onMount(() => {
		document.title = 'Task Timer';
		textarea.focus();
		if (!('Notification' in window)) {
			alert('This browser does not support desktop notification');
		} else if (Notification.permission !== 'denied') {
			Notification.requestPermission();
		}
	});

	function clearAll() {
		textarea.value = '';
		textarea.focus();
		document.title = 'Task Timer';
		tasks = [];

		playing = false;
		if (intervalId) {
			window.clearInterval(intervalId);
		}
	}

	function onClear() {
		console.log('on clear');
		clearAll();
	}

	function onDone() {
		console.log('on done');
		currentTaskIndex++;
		new Audio('success.m4a').play();
		toastStore.trigger(successToast);

		if (currentTaskIndex >= tasks.length) {
			clearAll();
			new Notification('Finished All Tasks!');
			return;
		}
		startTime = Date.now();
	}

	function onStart() {
		console.log('on start');
		const tasksInputValue = textarea.value;
		failed = false;
		currentTaskIndex = 0;
		tasks = parseTasks(tasksInputValue);

		if (tasks.length == 0) {
			alert('No tasks found');
			return;
		}
		let newTextAreaValue = '';
		for (const task of tasks) {
			newTextAreaValue += task.name + ' ' + task.time / 60 + '\n';
		}
		textarea.value = newTextAreaValue;
		console.log(tasks);
		playing = true;

		startTime = Date.now();

		intervalId = window.setInterval(() => {
			console.log('interval called');
			const elapsedTime = Math.floor((Date.now() - startTime) / 1000);
			timeLeft = tasks[currentTaskIndex]?.time - elapsedTime;
			console.log(timeLeft);
			document.title = timeString + ' - ' + taskToString(tasks[currentTaskIndex]);

			if (timeLeft <= 0) {
				failed = true;
				toastStore.trigger(failureToast);
				new Audio('failure.m4a').play();
				new Notification('Mission Failed');
				clearAll();
			}
		}, 1000);
	}

	function taskToString(task: Task) {
		if (task.repetitionCount > 1) {
			return task.name + ' (' + task.index + ')';
		} else {
			return task.name;
		}
	}

	function handleKeyDown(event: KeyboardEvent) {
		if (event.key === 'Enter' && event.shiftKey) {
			event.preventDefault();
			if (!playing) {
				onStart();
			}
		}
	}
</script>

<Toast />
<div class="vertical">
	<div class="body" id="body">
		{#if playing}
			<h1 class="h1">{taskToString(tasks[currentTaskIndex])}</h1>
			<h2 class="h2">{timeString}</h2>
		{:else}
			{#if failed}
				<h1 class="h1 text-red-600">Failed</h1>
			{:else}
				<h1 class="h1">Task Timer</h1>
			{/if}
			<h2 class="h2">00:00</h2>
		{/if}
		<textarea
			class="textarea"
			rows="16"
			placeholder="Enter tasks..."
			bind:this={textarea}
			on:keydown={handleKeyDown}
		/>
		<div class="buttons">
			<button type="button" on:click={onClear} class="btn btn-xl variant-filled-secondary"
				>Clear</button
			>
			{#if playing}
				<button type="button" on:click={onDone} class="btn btn-xl variant-filled-tertiary"
					>Done</button
				>
			{:else}
				<button type="button" on:click={onStart} class="btn btn-xl variant-filled-primary"
					>Start</button
				>
			{/if}
		</div>
	</div>
</div>

<style>
	.vertical {
		height: 100%;
		overflow-y: auto;
		display: flex;
		margin: 0 !important;
		align-items: center;
	}

	.h1 {
		white-space: nowrap;
		overflow-x: clip;
		text-overflow: ellipsis;
		max-width: 100%;
		display: inline-block; /* This prevents the element from growing horizontally */
	}

	.h1,
	.h2 {
		text-align: center;
		margin-top: 10px;
	}

	textarea {
		outline: none;
		margin: 10px 20px;
		width: calc(100% - 40px);
		padding: 10px;
		resize: none;
		font-size: 20px;
	}

	textarea:focus {
		outline: none;
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
