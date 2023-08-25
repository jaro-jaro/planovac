<script lang="ts">
	// @ts-ignore
	import Calendar from '@event-calendar/core';
	// @ts-ignore
	import DayGrid from '@event-calendar/day-grid';
	// @ts-ignore
	import Interaction from '@event-calendar/interaction';

	import type { DataType } from 'csstype'

	type EventType = {
		id?: string;
		allDay: true;
		start: string;
		end: string;
		title: Clovek;
		editable: true;
		startEditable: true;
		display?: string;
		backgroundColor: DataType.NamedColor;
		textColor: DataType.NamedColor;
	};

	const kamici = ["Arťa", "Barča", "Bětka", "Janda", "Jošt", "0ndra", "Ráďa", "Terka", "Verča"] as const

	type Clovek = typeof kamici[number]

	const barvy: Record<Clovek, DataType.NamedColor> = {
		"Arťa": "deeppink",
		"Barča": "aqua",
		"Bětka": "blue",
		"Janda": "orangered",
		"Jošt": "darkgreen",
		"0ndra": "royalblue",
		"Ráďa": "gold",
		"Terka": "black",
		"Verča": "rosybrown",
	}

	const barvyTextu: Record<Clovek, DataType.NamedColor> = {
		"Arťa": "white",
		"Barča": "black",
		"Bětka": "white",
		"Janda": "white",
		"Jošt": "white",
		"0ndra": "white",
		"Ráďa": "black",
		"Terka": "white",
		"Verča": "black",
	}

	let clovek: Clovek = "Ráďa"

	let barva = barvy[clovek]

	let barvaTextu = barvyTextu[clovek]

	let events = kamici.map(kamik => {
		return {
			allDay: true,
			start: '2023-08-23 00:00:00',
			end: '2023-08-28 00:00:00',
			title: kamik,
			editable: true,
			startEditable: true,
			backgroundColor: barvy[kamik],
			textColor: barvyTextu[kamik],
		} as EventType
	})

	$: console.log(events)

	$: {

	}

	const plugins = [DayGrid, Interaction];
	let options = {
		view: 'dayGridMonth',
		events: events,
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
		dayHeaderFormat: (date: Date) => {
			switch (date.getDay()) {
				case 0:
					return 'Neděle';
				case 1:
					return 'Pondělí';
				case 2:
					return 'Úterý';
				case 3:
					return 'Středa';
				case 4:
					return 'Čtvrtek';
				case 5:
					return 'Pátek';
				case 6:
					return 'Sobota';
			}
		},
		editable: true,
		eventDrop: (info: { event: EventType, oldEvent: EventType }) => {
			const index = events.map(ev => ev.title).indexOf(info.oldEvent.title)
			events[index] = info.event
		}
	};
</script>

<Calendar {plugins} {options} />