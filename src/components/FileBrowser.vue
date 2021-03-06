<template>
	<VTable
		ref="table"
		class="element-table"
		:data="elements">
		<thead slot="head">
			<th style="width: 10%;" />
			<MyVTh sort-key="basename" style="width: 50%;">
				<template #descIcon>
					<svg width="16"
						height="16"
						xmlns="http://www.w3.org/2000/svg"
						viewBox="0 0 320 512">
						<path fill="currentColor" d="M41 288h238c21.4 0 32.1 25.9 17 41L177 448c-9.4 9.4-24.6 9.4-33.9 0L24 329c-15.1-15.1-4.4-41 17-41z" />
					</svg>
				</template>
				<template #sortIcon>
					<svg width="16"
						height="16"
						xmlns="http://www.w3.org/2000/svg"
						viewBox="0 0 320 512">
						<path fill="currentColor" d="M41 288h238c21.4 0 32.1 25.9 17 41L177 448c-9.4 9.4-24.6 9.4-33.9 0L24 329c-15.1-15.1-4.4-41 17-41zm255-105L177 64c-9.4-9.4-24.6-9.4-33.9 0L24 183c-15.1 15.1-4.4 41 17 41h238c21.4 0 32.1-25.9 17-41z" />
					</svg>
				</template>
				<template #ascIcon>
					<svg width="16"
						height="16"
						xmlns="http://www.w3.org/2000/svg"
						viewBox="0 0 320 512">
						<path fill="currentColor" d="M279 224H41c-21.4 0-32.1-25.9-17-41L143 64c9.4-9.4 24.6-9.4 33.9 0l119 119c15.2 15.1 4.5 41-16.9 41z" />
					</svg>
				</template>
				{{ t('filepicker', 'Name') }}
			</MyVTh>
			<MyVTh sort-key="size" style="width: 15%;">
				{{ t('filepicker', 'Size') }}
			</MyVTh>
			<MyVTh sort-key="lastmod_ts" style="width: 25%;">
				{{ t('filepicker', 'Modified') }}
			</MyVTh>
		</thead>
		<tbody slot="body" slot-scope="{displayData}">
			<tr v-for="value in displayData"
				:key="value.filename"
				:class="{ selectable: isSelectable(value), selected: selection.includes(value.filename) }"
				@click="onElemClick(value)">
				<td>
					<span :class="{ icon: true, ...getElemTypeClass(value) }" />
				</td>
				<td :style="''">
					<div>
						{{ value.basename }}
					</div>
				</td>
				<td :style="''">
					<div>
						{{ myHumanFileSize(value.size, true) }}
					</div>
				</td>
				<td :style="''">
					<div>
						{{ lastModFormat(value.lastmod_ts) }}
					</div>
				</td>
			</tr>
		</tbody>
	</VTable>
</template>

<script>
import { t, n } from '../translation'
import moment from '@nextcloud/moment'
import { humanFileSize } from '../utils'
import MyVTh from './MyVTh'

import { VTable } from 'vuejs-smart-table'

export default {
	name: 'FileBrowser',

	components: {
		VTable,
		MyVTh,
	},

	props: {
		elements: {
			type: Array,
			required: true,
		},
		forcedSelection: {
			type: Array,
			default: () => { return [] },
		},
		canSelectFiles: {
			type: Boolean,
			default: true,
		},
		multipleSelect: {
			type: Boolean,
			default: true,
		},
		disabled: {
			type: Boolean,
			default: false,
		},
	},

	data() {
		return {
			t,
			n,
			selection: [],
		}
	},

	computed: {
	},

	watch: {
		forcedSelection() {
			this.selection = this.forcedSelection
		},
		elements() {
			this.$refs.table.$el.scrollTop = 0
		},
	},

	mounted() {
	},

	methods: {
		lastModFormat(ts) {
			return moment.unix(ts).format('L HH:mm:ss')
		},
		myHumanFileSize(bytes, approx = false, si = false, dp = 1) {
			return humanFileSize(bytes, approx, si, dp)
		},
		isSelectable(elem) {
			return elem.type === 'directory' || this.canSelectFiles
		},
		getElemTypeClass(elem) {
			if (elem.type === 'directory') {
				return { 'icon-folder': true }
			} else {
				const mime = elem.mime
				if (mime.match(/^video\//)) {
					return { 'icon-video': true }
				} else if (mime === 'text/calendar') {
					return { 'icon-calendar': true }
				} else if (mime === 'text/csv' || mime.match(/^application\/.*opendocument\.spreadsheet$/) || mime.match(/^application\/.*office.*sheet$/)) {
					return { 'icon-spreadsheet': true }
				} else if (mime.match(/^text\//)) {
					return { 'icon-text': true }
				} else if (mime.match(/^application\/pdf$/)) {
					return { 'icon-pdf': true }
				} else if (mime.match(/^application\/gpx/)) {
					return { 'icon-location': true }
				} else if (mime.match(/^image\//)) {
					return { 'icon-picture': true }
				} else if (mime.match(/^audio\//)) {
					return { 'icon-audio': true }
				} else if (mime.match(/^application\/.*opendocument\.text$/) || mime.match(/^application\/.*word.*document$/)) {
					return { 'icon-office-document': true }
				} else if (mime.match(/^application\/.*opendocument\.presentation$/) || mime.match(/^application\/.*office.*presentation$/)) {
					return { 'icon-office-presentation': true }
				}
				return { 'icon-file': true }
			}
		},
		onElemClick(e) {
			if (this.disabled) {
				return
			}
			if (e.type === 'directory') {
				this.$emit('folder-clicked', e.filename)
				this.selection = []
			} else if (this.canSelectFiles) {
				if (this.multipleSelect) {
					if (this.selection.includes(e.filename)) {
						this.selection.splice(this.selection.indexOf(e.filename), 1)
					} else {
						this.selection.push(e.filename)
					}
				} else {
					if (this.selection.includes(e.filename)) {
						this.selection = []
					} else {
						this.selection = [e.filename]
					}
				}
				this.$emit('selection-changed', this.selection)
			}
		},
	},
}
</script>

<style scoped lang="scss">
.element-table {
	width: 100%;
	height: 100%;
	overflow-y: scroll;
	overflow-x: hidden;
	scrollbar-color: var(--color-border-dark) transparent;
	scrollbar-width: thin;
	display: block;
	border-spacing: 0;
	// padding: 10px 0 10px 0;

	.icon {
		display: inline-block;
		width: 100px;
		height: 50px;
		background-repeat: no-repeat;
		background-size: 30px;
		background-position: center;
	}

	th {
		text-align: left;
		height: 50px;
		font-weight: bold;
		&:not(:first-child) {
			cursor: pointer;
		}
		position: sticky;
		top: 0;
		z-index: 2;
		border: 0;
		&::before {
			width: 100%;
			height: 66px;
			content: ' ';
			display: block;
			position: absolute;
			top: 0;
			left: 0;
			background-image: linear-gradient(180deg, var(--color-main-background) 65%, transparent 100%);
			z-index: -1;
		}
	}

	tr {
		z-index: 1;

		&:not(:first-child) td {
			border-top: 1px solid var(--color-border);
		}

		&:not(.selectable) td > * {
			opacity: 30%;
		}

		&.selectable {
			cursor: pointer;
			td {
				cursor: pointer;
				* {
					cursor: pointer;
				}
			}

			&.selected {
				background-color: var(--color-background-darker);
			}

			&:hover:not(.selected) {
				background-color: var(--color-background-hover);
			}
		}
	}

	td {
		border: 0;
		height: 50px;

		.icon {
			mask-size: 30px auto;
			mask-position: center;
			-webkit-mask-size: 30px auto;
			-webkit-mask-position: center;
			background-color: var(--color-text-lighter);
		}
	}

	.icon-folder {
		background-color: var(--color-primary-element, grey) !important;
	}
	.icon-file {
		mask: url('./../../img/file.svg') no-repeat;
		-webkit-mask: url('./../../img/file.svg') no-repeat;
	}
	.icon-video {
		mask: url('./../../img/video.svg') no-repeat;
		-webkit-mask: url('./../../img/video.svg') no-repeat;
	}
	.icon-audio {
		mask: url('./../../img/audio.svg') no-repeat;
		-webkit-mask: url('./../../img/audio.svg') no-repeat;
	}
	.icon-calendar {
		mask: url('./../../img/calendar.svg') no-repeat;
		-webkit-mask: url('./../../img/calendar.svg') no-repeat;
	}
	.icon-text {
		mask: url('./../../img/text.svg') no-repeat;
		-webkit-mask: url('./../../img/text.svg') no-repeat;
	}
	.icon-location {
		mask: url('./../../img/location.svg') no-repeat;
		-webkit-mask: url('./../../img/location.svg') no-repeat;
	}
	.icon-picture {
		mask: url('./../../img/picture.svg') no-repeat;
		-webkit-mask: url('./../../img/picture.svg') no-repeat;
	}
	.icon-pdf {
		background-image: url('./../../img/pdf.svg');
		background-color: unset !important;
	}
	.icon-office-document {
		background-image: url('./../../img/office-document.svg');
		background-color: unset !important;
	}
	.icon-office-presentation {
		background-image: url('./../../img/office-presentation.svg');
		background-color: unset !important;
	}
	.icon-spreadsheet {
		background-image: url('./../../img/spreadsheet.svg');
		background-color: unset !important;
	}
	.icon-text,
	.icon-audio,
	.icon-calendar,
	.icon-picture {
		background-image: unset;
	}
}
</style>
