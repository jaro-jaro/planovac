<script lang="ts">
	// @ts-ignore
	import Calendar from '@event-calendar/core';
	// @ts-ignore
	import DayGrid from '@event-calendar/day-grid';
	// @ts-ignore
	import Interaction from '@event-calendar/interaction';

	import type { DataType } from 'csstype';
	import { onMount } from 'svelte';

	import { initializeApp, getApps } from 'firebase/app';
	import { getDatabase, ref, set, onValue, get } from 'firebase/database';

	const firebaseConfig = {
		apiKey: 'AIzaSyD0dPss3iRcYR3vjhIChMdnh8K3XR98IR0',
		authDomain: 'planovac-jaro.firebaseapp.com',
		projectId: 'planovac-jaro',
		storageBucket: 'planovac-jaro.appspot.com',
		messagingSenderId: '154181716572',
		appId: '1:154181716572:web:9f2e5404457eaed5433499',
		databaseURL: 'https://planovac-jaro-default-rtdb.europe-west1.firebasedatabase.app/'
	};

	const app = getApps().length === 0 ? initializeApp(firebaseConfig) : getApps()[0];
	const db = getDatabase(app);

	type EventType = {
		id: string;
		start: string;
		end: string;
		allDay: true;
		title: Clovek;
		backgroundColor: DataType.NamedColor;
		textColor: DataType.NamedColor;
	};

	type StredneZlejEventType = {
		id: string;
		start: string;
		end: string;
		title: Clovek;
	};

	type ZlejEventType = {
		id: string;
		start: Date;
		end: Date;
		title: Clovek;
		allDay: true;
		backgroundColor: DataType.NamedColor;
		textColor: DataType.NamedColor;
	};

	const kamici = [
		'Arťa',
		'Barča',
		'Bětka',
		'Janda',
		'Jošt',
		'0ndra',
		'Ráďa',
		'Terka',
		'Verča'
	] as const;

	type Clovek = (typeof kamici)[number];

	const barvy: Record<Clovek, DataType.NamedColor> = {
		Arťa: 'deeppink',
		Barča: 'aqua',
		Bětka: 'blue',
		Janda: 'orangered',
		Jošt: 'darkgreen',
		'0ndra': 'lime',
		Ráďa: 'gold',
		Terka: 'royalblue',
		Verča: 'rosybrown',
  Expedice: 'white',
	};

	const barvyTextu: Record<Clovek, DataType.NamedColor> = {
		Arťa: 'white',
		Barča: 'black',
		Bětka: 'white',
		Janda: 'white',
		Jošt: 'white',
		'0ndra': 'black',
		Ráďa: 'black',
		Terka: 'white',
		Verča: 'black',
  Expedice: 'black',
	};

	let clovek: Clovek | undefined;

	let events: EventType[] | undefined = undefined;

	// $: console.log(events)
	// $: console.log(clovek);
	// $: console.log(barva);

	const refff = ref(db, 'events/');

	$: {
		if (events != undefined) set(refff, events);
	}

	onValue(refff, (snapshot) => {
		const data = snapshot.val() as StredneZlejEventType[] | undefined;
		events =
			data?.map((ev): EventType => {
				return {
					...ev,
					allDay: true,
					backgroundColor: barvy[ev.title],
					textColor: barvyTextu[ev.title]
				};
			}) ??
			events ??
			[];
	});

	let ec: Calendar;

	let hid: number[] = [];

	const update = () => {
	};

	let localWindow: Window & typeof globalThis;

	onMount(async () => {
		localWindow = window;
		clovek = (window?.localStorage?.getItem('ja') ?? clovek) as Clovek;

		document.querySelectorAll('.ec-button').forEach((el: any) => {
			el.onclick = () => {
				update();
			};
		});
	});

	$: {
		if (localWindow != undefined && clovek != undefined)
			localWindow?.localStorage?.setItem('ja', clovek);
	}

	const remove = (events: EventType[], id: string) => {
		let copy = [...events];
		const i = events.findIndex((ev) => ev.id == id);
		copy.splice(i, 1);
		return copy;
	};

	const zlepsit = (events: EventType[], newEvent: EventType): EventType[] => {
		const ostatniEventy = events.filter((ev) => ev.title == clovek);

		const prekryvaci = ostatniEventy.filter(
			(ev) =>
				(newEvent.start <= ev.start || newEvent.start <= ev.end) &&
				(ev.start <= newEvent.end || ev.end <= newEvent.end)
		);

		if (prekryvaci.length == 0) {
			return [newEvent, ...events];
		}

		const min = [newEvent, ...prekryvaci].sort(
			(a: EventType, b: EventType) => new Date(a.start).getTime() - new Date(b.start).getTime()
		)[0];
		const max = [newEvent, ...prekryvaci].sort(
			(a: EventType, b: EventType) => new Date(b.end).getTime() - new Date(a.end).getTime()
		)[0];

		const combined = {
			...newEvent,
			start: min.start,
			end: max.end
		};

		let newEvents = [...events];

		prekryvaci.forEach((ev) => {
			const i = newEvents.findIndex((e) => e.id == ev.id);
			newEvents.splice(i, 1);
		});

		newEvents.push(combined);

		return newEvents;
	};

	const zlepsitEvent = (ev: ZlejEventType): EventType => {
		return {
			id: ev.id,
			allDay: true,
			start: ev.start.toISOString(),
			end: ev.end.toISOString(),
			title: ev.title,
			backgroundColor: ev.backgroundColor,
			textColor: ev.textColor
		};
	};

	const plugins = [DayGrid, Interaction];
	$: options = {
		view: 'dayGridMonth',
		events:
			events
				?.sort((a, b) => kamici.indexOf(a.title) - kamici.indexOf(b.title))
				?.map((ev) => {
					return {
						...ev,
						start: new Date(ev.start),
						end: new Date(ev.end)
					} as ZlejEventType;
				}) ?? [],
		lazyFetching: false,
		buttonText: {
			close: 'Zavřít',
			dayGridMonth: 'měsíc',
			listDay: 'seznam',
			listMonth: 'seznam',
			listWeek: 'seznam',
			listYear: 'seznam',
			resourceTimeGridDay: 'den',
			resourceTimeGridWeek: 'týden',
			timeGridDay: 'den',
			timeGridWeek: 'týden',
			today: 'dnes'
		},
		dayHeaderFormat: { weekday: 'long' },
		editable: true,
		eventDrop: (info: { event: ZlejEventType; oldEvent: ZlejEventType }) => {
			if (events == undefined) return;
			events = zlepsit(remove(events, info.oldEvent.id), zlepsitEvent(info.event));
		},
		titleFormat: { year: 'numeric', month: 'numeric' },
		selectable: clovek != undefined,
		selectBackgroundColor: clovek != undefined ? barvy[clovek] : 'white',
		longPressDelay: 200,
		eventResize: (info: { event: ZlejEventType; oldEvent: ZlejEventType }) => {
			if (events == undefined) return;
			events = zlepsit(remove(events, info.oldEvent.id), zlepsitEvent(info.event));
		},
		firstDay: 1,
		hiddenDays: hid,
		locale: 'cs',
		select: (info: { start: Date; end: Date }) => {
			if (clovek == undefined) {
				alert('Prosím, vyber se!');
				ec.unselect();
				return;
			}

			const newEvent: EventType = {
				id: self.crypto.randomUUID(),
				start: info.start.toISOString(),
				allDay: true,
				end: info.end.toISOString(),
				title: clovek,
				backgroundColor: barvy[clovek],
				textColor: barvyTextu[clovek]
			};

			events = zlepsit(events ?? [], newEvent);
			ec.unselect();
		},
		eventClick: (info: { event: EventType }) => {
			if (events == undefined) return;
			if (confirm('Opravdu chceš odstranit svou (nebo snad nějakáho kámíka??) volnost?')) {
				events = remove(events, info.event.id);
			}
		}
	};
</script>

<label for="lidi">Vyber se:</label>

<select name="lidi" bind:value={clovek}>
	<option value={undefined}>Nevybráno</option>
	{#each kamici as kamik}
		<option value={kamik}>{kamik}</option>
	{/each}
</select>

<Calendar bind:this={ec} {plugins} {options} />

<style>
	select {
		background-color: black;
		color: white;
		border-radius: 10px;
		width: 150px;
		font-size: medium;
		padding: 4px;
	}
	select option {
		margin: 8px;
	}
</style>
