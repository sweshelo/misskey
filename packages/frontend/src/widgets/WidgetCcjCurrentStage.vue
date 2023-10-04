<!--
SPDX-FileCopyrightText: syuilo and other misskey contributors
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
	<MkContainer :showHeader="widgetProps.showHeader" class="mkw-ccj-current-stage">
		<template #icon><i class="ti ti-cookie"></i></template>
		<template #header>CCJ Current Stage</template>
		<h1>CCJ 現在のステージ</h1>
		<p v-for="schedule in ccjInfo.schedules" :key="schedule.date.start.toDateString()">{{ `${schedule.date.start} ~ ${schedule.date.end} : ${schedule.stage.odd} / ${schedule.stage.even}` }}</p>
	</MkContainer>
</template>

<script lang="ts" setup>
import { JSDOM } from 'jsdom';
import { useWidgetPropsManager, Widget, WidgetComponentEmits, WidgetComponentExpose, WidgetComponentProps } from './widget.js';
import { GetFormResultType } from '@/scripts/form.js';
import MkContainer from '@/components/MkContainer.vue';

const name = 'ccjCurrentStage';

type ScheduleInfo = {
	date: {
		start: Date;
		end: Date;
	},
	stage: {
		odd: string;
		even: string;
	}
};
type CCJInfo = {
	schedules: ScheduleInfo[]
};

const ccjInfo: CCJInfo = {
	schedules: [],
};

const fetchCCJInfo = () => {
	window.fetch('https://p.eagate.573.jp/game/chase2jokers/ccj/news/index.html').then(r => r.text()).then((r) => {
	const dom = new JSDOM(r);
	const { document } = dom.window;
	ccjInfo.schedules = [...document.querySelectorAll('.list > li')].filter((elm: Element) => elm?.querySelector('dd')?.innerText.includes('ステージスケジュール')).map((elm: Element) => [...elm.querySelectorAll('table > tbody > tr')].slice(1)).filter((elm) => elm.length).flat().map((elm) => {
		// 各カラム
		const [datespanElement, evenElement, oddElement] = [...elm.querySelectorAll('td')];

		// 日付パース
		const [dateA, dateB] = datespanElement.innerText.split(' - ').map((s) => new Date('2023/' + s.replace(/\(.*?\)/, '')));

		return {
			date: {
				start: dateA,
				end: dateB,
			},
			stage: {
				odd: oddElement.innerText,
				even: evenElement.innerText,
			},
		};
	});
});
};

fetchCCJInfo();

const widgetPropsDef = {
	showHeader: {
		type: 'boolean' as const,
		default: false,
	},
};

type WidgetProps = GetFormResultType<typeof widgetPropsDef>;

const props = defineProps<WidgetComponentProps<WidgetProps>>();
const emit = defineEmits<WidgetComponentEmits<WidgetProps>>();

const { widgetProps, configure } = useWidgetPropsManager(name,
	widgetPropsDef,
	props,
	emit,
);

defineExpose<WidgetComponentExpose>({
	name,
	configure,
	id: props.widget ? props.widget.id : null,
});
</script>
