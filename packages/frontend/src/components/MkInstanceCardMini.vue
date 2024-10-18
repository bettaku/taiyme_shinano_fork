<!--
SPDX-FileCopyrightText: syuilo and misskey-project
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<div
	:class="[$style.root, {
		[$style.isNotResponding]: instance.isNotResponding,
		[$style.isSilenced]: instance.isSilenced,
		[$style.isSuspended]: instance.isSuspended,
		[$style.isBlocked]: instance.isBlocked,
	}]"
>
	<img :class="$style.icon" :src="getInstanceIcon(instance)" alt="" loading="lazy"/>
	<div :class="$style.body">
		<span :class="$style.host">{{ instance.name || instance.host }}</span>
		<span :class="$style.sub">
			<span class="_monospace">
				<span style="font-weight: 700;">{{ instance.host }}</span> / {{ instance.softwareName || '?' }} {{ instance.softwareVersion }}
			</span>
		</span>
	</div>
	<MkMiniChart v-if="chartValues" :class="$style.chart" :src="chartValues"/>
</div>
</template>

<script lang="ts" setup>
import { shallowRef, watch } from 'vue';
import * as Misskey from 'misskey-js';
import { misskeyApiGet } from '@/scripts/misskey-api.js';
import { getProxiedImageUrlNullable } from '@/scripts/media-proxy.js';
import MkMiniChart from '@/components/MkMiniChart.vue';

const props = defineProps<{
	instance: Misskey.entities.FederationInstance;
}>();

const chartValues = shallowRef<number[] | null>(null);

watch(() => props.instance.host, (host) => {
	misskeyApiGet('charts/instance', {
		host,
		limit: 16 + 1,
		span: 'day',
	}).then(res => {
		// 今日のぶんの値はまだ途中の値であり、それも含めると大抵の場合前日よりも下降しているようなグラフになってしまうため今日は弾く
		res.requests.received.shift();
		chartValues.value = res.requests.received;
	}).catch(() => {
		chartValues.value = null;
	});
}, {
	immediate: true,
});

const getInstanceIcon = (instance: Misskey.entities.FederationInstance) => (
	getProxiedImageUrlNullable(instance.iconUrl, 'preview')
		?? getProxiedImageUrlNullable(instance.faviconUrl, 'preview')
		?? '/client-assets/dummy.png'
);
</script>

<style lang="scss" module>
$bodyTitleHieght: 18px;
$bodyInfoHieght: 16px;

.root {
	display: flex;
	align-items: center;
	padding: 16px;
	background: var(--MI_THEME-panel);
	border-radius: 8px;
	background-color: var(--panel);

	&.isNotResponding,
	&.isSilenced,
	&.isSuspended,
	&.isBlocked {
		background-image: repeating-linear-gradient(
			135deg,
			transparent 0px 10px,
			var(--c) 6px 16px
		);

		// NOTE: iOS/iPadOS環境でクラッシュする https://github.com/taiyme/misskey/issues/293
		html[data-browser-engine=webkit] & {
			background-image: unset !important;
		}
	}

	&,
	html[data-color-scheme=light] & {
		&.isNotResponding {
			--c: color(from color-mix(in srgb, var(--panel), orange 50%) srgb r g b / 0.25);
		}

		&.isSilenced {
			--c: color(from color-mix(in srgb, var(--panel), blue 50%) srgb r g b / 0.25);
		}

		&.isSuspended {
			--c: color(from color-mix(in srgb, var(--panel), black 15%) srgb r g b / 0.25);
		}

		&.isBlocked {
			--c: color(from color-mix(in srgb, var(--panel), red 50%) srgb r g b / 0.25);
		}
	}

	html[data-color-scheme=dark] & {
		&.isNotResponding {
			--c: color(from color-mix(in srgb, var(--panel), orange 50%) srgb r g b / 0.5);
		}

		&.isSilenced {
			--c: color(from color-mix(in srgb, var(--panel), blue 50%) srgb r g b / 0.5);
		}

		&.isSuspended {
			--c: color(from color-mix(in srgb, var(--panel), white 15%) srgb r g b / 0.5);
		}

		&.isBlocked {
			--c: color(from color-mix(in srgb, var(--panel), red 50%) srgb r g b / 0.5);
		}
	}
}

.icon {
	display: block;
	width: ($bodyTitleHieght + $bodyInfoHieght);
	height: ($bodyTitleHieght + $bodyInfoHieght);
	object-fit: cover;
	border-radius: 4px;
	margin-right: 12px;
}

	> :global(.body) {
		flex: 1;
		overflow: hidden;
		font-size: 0.9em;
		color: var(--MI_THEME-fg);
		padding-right: 8px;

.host {
	display: block;
	width: 100%;
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
	line-height: $bodyTitleHieght;
}

.sub {
	display: block;
	width: 100%;
	font-size: 80%;
	opacity: 0.7;
	line-height: $bodyInfoHieght;
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

	> :global(.chart) {
		height: 30px;
	}

  &:global(.blue) {
    --c: rgba(0, 42, 255, 0.15);
    background-image: linear-gradient(45deg, var(--c) 16.67%, transparent 16.67%, transparent 50%, var(--c) 50%, var(--c) 66.67%, transparent 66.67%, transparent 100%);
    background-size: 16px 16px;
  }

	&:global(.yellow) {
		--c: rgb(255 196 0 / 15%);
		background-image: linear-gradient(45deg, var(--c) 16.67%, transparent 16.67%, transparent 50%, var(--c) 50%, var(--c) 66.67%, transparent 66.67%, transparent 100%);
		background-size: 16px 16px;
	}

	&:global(.red) {
		--c: rgb(255 0 0 / 15%);
		background-image: linear-gradient(45deg, var(--c) 16.67%, transparent 16.67%, transparent 50%, var(--c) 50%, var(--c) 66.67%, transparent 66.67%, transparent 100%);
		background-size: 16px 16px;
	}

	&:global(.gray) {
		--c: var(--MI_THEME-bg);
		background-image: linear-gradient(45deg, var(--c) 16.67%, transparent 16.67%, transparent 50%, var(--c) 50%, var(--c) 66.67%, transparent 66.67%, transparent 100%);
		background-size: 16px 16px;
	}
}
</style>
