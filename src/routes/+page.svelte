<script lang="ts">
	import { parseTasks } from "../utils";
	import type { Task } from "../utils/types";
	import { onMount } from "svelte";
	import { toastStore, type ToastSettings } from '@skeletonlabs/skeleton';

	let playing = false;
	let textarea: HTMLTextAreaElement;
	let intervalId: number;
	let timeLeft = 0;
	let tasks: Task[] = [];
	let failed = false;
	let currentTaskIndex = 0;
	let startTime: number;
	$: timeString = (Math.floor(timeLeft / 60)).toString().padStart(2, '0') + ':' + (timeLeft % 60).toString().padStart(2, '0');

	onMount(() => {
		document.title = "Task Timer";
		if (!("Notification" in window)) {
			alert("This browser does not support desktop notification");
		} else if (Notification.permission === "granted") {
			new Notification("Hi there!");
		} else if (Notification.permission !== "denied") {
			Notification.requestPermission().then((permission) => {
				if (permission === "granted") {
					new Notification("Hi there!");
				}
			})
		};
	})

	function clearAll() {
		textarea.value = '';
		textarea.focus();
		document.title = "Task Timer";
		tasks = [];
		new Notification("clear all")
		
		playing = false;
		if (intervalId) {
			window.clearInterval(intervalId);
		}
	}

	function onClear() {
		console.log('on clear')
		clearAll();
	}

	function onDone() {
		console.log('on done')
		currentTaskIndex++;
		if (currentTaskIndex >= tasks.length) {
			clearAll();
			return;
		}
		const notificationMessage = taskToString(tasks[currentTaskIndex]) + " started.";
		new Notification(notificationMessage);
		startTime = Date.now();
	}

	function onStart() {	
		console.log('on start')
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
			console.log("interval called")
			const elapsedTime = Math.floor((Date.now() - startTime) / 1000);
			timeLeft = tasks[currentTaskIndex]?.time - elapsedTime;
			console.log(timeLeft);
			document.title = timeString + " - " + tasks[currentTaskIndex]?.name;

			if (timeLeft <= 0) {
				failed = true;
				new Notification("Mission Failed.")
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
</script>

<div class="vertical">
	<div class="body">
		{#if playing}
			<h1 class="h1">{taskToString(tasks[currentTaskIndex])}</h1>
			<h1 class="h2">{timeString}</h1>
		{:else}
			{#if failed}
				<h1 class="h1 text-red-600">Failed</h1>
			{:else}
				<h1 class="h1">Task Timer</h1>
			{/if}
			<h2 class="h2">00:00</h2>
		{/if}
		<textarea class="textarea" rows="16" placeholder="Enter tasks..." bind:this={textarea} />
		<div class="buttons">
			<button type="button" on:click={onClear} class="btn btn-xl variant-filled-secondary">Clear</button>
			{#if playing}
				<button type="button" on:click={onDone} class="btn btn-xl variant-filled-tertiary">Done</button>
			{:else}
				<button type="button" on:click={onStart} class="btn btn-xl variant-filled-primary">Start</button>
			{/if}
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
		line-height: 70px;
		overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
	}

	.h2 {
		font-size: 50px;
		line-height: 40px;
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
