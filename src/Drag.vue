<template>
	<component :is="tag"
		:draggable="draggable"
		@drag="emitEvent('drag', $event)"
		@dragstart="emitEvent('dragstart', $event)"
		@dragenter="emitEvent('dragenter', $event)"
		@dragleave="emitEvent('dragleave', $event)"
		@dragend="emitEvent('dragend', $event)"
	>
		<slot :transfer-data="scopedData"></slot>
		<div v-if="hideImageHtml" :style="hideImageStyle">
			<slot name="image" :transfer-data="scopedData"></slot>
		</div>
		<slot v-else name="image" :transfer-data="scopedData"></slot>
	</component>
</template>

<script>
	import transferDataStore from './transferDataStore';

	export default {
		props: {
			draggable: {
				type: Boolean,
				default: true,
			},
			transferData: {},
			dropEffect: {
				validator: v => ['copy', 'move', 'link', 'none'].indexOf(v) >= 0,
			},
			effectAllowed: {
				validator: v => ['none', 'copy', 'copyLink', 'copyMove', 'link', 'linkMove', 'move', 'all', 'uninitialized'].indexOf(v) >= 0,
			},
			image: String,
			imageXOffset: {
				type: Number,
				default: 0,
			},
			imageYOffset: {
				type: Number,
				default: 0,
			},
			hideImageHtml: {
				type: Boolean,
				default: true,
			},
			tag: {
				type: String,
				default: 'div',
			},
		},
		data() {
			return { dragging: false };
		},
		computed: {
			scopedData() {
				return this.dragging && this.transferData;
			},
			hideImageStyle: () => ({ position: 'fixed', top: '-1000px' }),
		},
		methods: {
			emitEvent(name, nativeEvent) {
				const transfer = nativeEvent.dataTransfer;

				// Set drop effect on dragenter and dragover
				if (['dragenter', 'dragover'].includes(name)) {
					if (this.dropEffect) {
						transfer.dropEffect = this.dropEffect;
					}
				}

				// A number of things need to happen on drag start
				if (name === 'dragstart') {
					// Set the allowed effects
					if (this.effectAllowed) {
						transfer.effectAllowed = this.effectAllowed;
					}

					// Set the drag image
					if (this.image || this.$slots.image) {
						let image;
						if (this.image) {
							image = new Image();
							image.src = this.image;
						} else if (this.$slots.image) {
							image = this.$slots.image[0].elm;
						}
						if (transfer.setDragImage) {
							transfer.setDragImage(image, this.imageXOffset, this.imageYOffset);
						}
					}

					// Set the transfer data
					if (this.transferData !== undefined) {
						transferDataStore.data = this.transferData;
						// Set a dummy string for the real transfer data. Not actually used
						// for anything, but necesssary for browser compatibility.
						//
						// TODO: Maybe this should be the actual data serialized. But since
						// it's not actually used for anything it seems like a waste of CPU.
						nativeEvent.dataTransfer.setData('text', '');
					}

					// Indicate that we're dragging.
					this.dragging = true;
				}

				// At last, emit the event.
				this.$emit(name, this.transferData, nativeEvent);

				// Clean up stored data on drag end after emitting.
				if (name === 'dragend') {
					transferDataStore.data = undefined;
					this.dragging = false;
				}
			},
		},
	};
</script>
